ARG ZEPHYR_SDK_VERSION="0.17.4"
FROM debian:bookworm-slim

# 1. Setup Environment Variables (Keep as Root)
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    CCACHE_DIR=/workspaces/.ccache \
    UV_LINK_MODE=copy \
    ZEPHYR_SDK_VERSION=${ZEPHYR_SDK_VERSION}\
    ZEPHYR_SDK_INSTALL_DIR=/opt/zephyr-sdk \
    VIRTUAL_ENV=/opt/zephyr-venv \
    UV_PYTHON_INSTALL_DIR=/opt/uv-python
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# 2. Install System Dependencies + Sudo (As Root)
RUN echo "==> Installing system dependencies..." && \
    apt-get update -q && \
    apt-get install -y -qq --no-install-recommends \
    sudo git cmake ninja-build gperf \
    ccache dfu-util device-tree-compiler wget \
    python3 python3-venv htop \
    xz-utils file make gcc gcc-multilib g++-multilib libsdl2-dev \
    build-essential gdb tree \
    usbutils udev ca-certificates \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Create User (As Root)
RUN groupadd --gid 1000 vscode \
    && useradd --uid 1000 --gid 1000 -m vscode \
    && echo vscode ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/vscode \
    && chmod 0440 /etc/sudoers.d/vscode

# 4. Tooling Installation (As Root)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

RUN echo "==> Setting up Python environment..." && \
    mkdir -p $UV_PYTHON_INSTALL_DIR && \
    uv venv $VIRTUAL_ENV --python 3.13 --seed -q

WORKDIR /workspaces/.devcontainer
COPY devcontainer-requirements.txt .
RUN echo "==> Installing Python dependencies..." && \
    uv pip install --no-cache -q -r devcontainer-requirements.txt
WORKDIR /workspaces

RUN echo "==> Downloading Zephyr SDK v${ZEPHYR_SDK_VERSION}..." && \
    # Handle dynamic URL naming (v1.x uses _gnu, v0.x does not)
    if [ "$(printf '%s\n' "1.0.0" "$ZEPHYR_SDK_VERSION" | sort -V | head -n1)" = "1.0.0" ]; then \
        SDK_SUFFIX="_linux-x86_64_gnu.tar.xz"; \
    else \
        SDK_SUFFIX="_linux-x86_64.tar.xz"; \
    fi && \
    wget -q "https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v${ZEPHYR_SDK_VERSION}/zephyr-sdk-${ZEPHYR_SDK_VERSION}${SDK_SUFFIX}" && \
    \
    echo "==> Extracting SDK..." && \
    tar -xJf zephyr-sdk-${ZEYR_SDK_VERSION}${SDK_SUFFIX} -C /opt/ && \
    rm zephyr-sdk-${ZEPHYR_SDK_VERSION}${SDK_SUFFIX} && \
    mv /opt/zephyr-sdk-${ZEPHYR_SDK_VERSION} ${ZEPHYR_SDK_INSTALL_DIR}

RUN echo "==> Running Zephyr SDK Setup..." && \
    cd ${ZEPHYR_SDK_INSTALL_DIR} && \
    ./setup.sh -t arm-zephyr-eabi -h -c > /dev/null

RUN echo "==> Handle dynamic udev rules path .." && \
    mkdir -p /etc/udev/rules.d/ && \
    RULES_PATH=$(find ${ZEPHYR_SDK_INSTALL_DIR} -name "60-openocd.rules" | head -n 1) && \
    if [ -n "$RULES_PATH" ]; then \
        cp "$RULES_PATH" /etc/udev/rules.d/; \
    fi

# 5. Fix Permissions so 'vscode' can use the tools in /opt
RUN chown -R vscode:vscode /opt/zephyr-sdk /opt/zephyr-venv /opt/uv-python

# 6. Final Config
RUN usermod -aG dialout,plugdev vscode
USER vscode
WORKDIR /workspaces
ENV ZEPHYR_TOOLCHAIN_VARIANT=zephyr
