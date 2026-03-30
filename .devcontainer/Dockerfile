FROM debian:bookworm-slim

# 1. Setup Environment Variables (Keep as Root)
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    CCACHE_DIR=/workspaces/.ccache \
    UV_LINK_MODE=copy \
    ZEPHYR_SDK_VERSION=0.17.4 \
    ZEPHYR_SDK_INSTALL_DIR=/opt/zephyr-sdk \
    VIRTUAL_ENV=/opt/zephyr-venv
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
    uv venv $VIRTUAL_ENV --python 3.13 -q && \
    uv pip install --no-cache -q west jsonschema
    uv pip install pyelftools>=0.29 PyYAML>=6.0 pykwalify \
    canopen packaging progress "patool>=2.0.0" "psutil>=5.6.6" \
    pylink-square pyserial "requests>=2.32.0" semver "tqdm>=4.67.1" \
    reuse anytree intelhex "west>=0.14.0" "windows-curses; sys_platform == 'win32'" \
    colorama "ply>=3.10" "gcovr>=6.0" coverage pytest mypy "mock>=4.0.1" \
    junitparser "pyocd>=0.35.0" tabulate natsort "cbor>=1.0.0" "python-can>=4.3.0" \
    spdx-tools "gitpython>=3.1.41" gitlint junit2html lpc_checksum "spsdk==2.6.0" \
    "Pillow>=10.3.0" PyGithub graphviz "clang-format>=15.0.0" "junitparser>=2" "lxml>=5.3.0" \
    "pylint>=3" "python-magic-bin; sys_platform == 'win32'" "python-magic; sys_platform != 'win32'" \
    ruff==0.11.11 sphinx-lint unidiff vermin yamllint


RUN echo "==> Downloading Zephyr SDK v${ZEPHYR_SDK_VERSION}..." && \
    wget --show-progress --progress=bar:force:noscroll -q "https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v${ZEPHYR_SDK_VERSION}/zephyr-sdk-${ZEPHYR_SDK_VERSION}_linux-x86_64.tar.xz" && \
    echo "==> Extracting SDK..." && \
    tar -xJf zephyr-sdk-${ZEPHYR_SDK_VERSION}_linux-x86_64.tar.xz -C /opt/ && \
    rm zephyr-sdk-${ZEPHYR_SDK_VERSION}_linux-x86_64.tar.xz && \
    mv /opt/zephyr-sdk-${ZEPHYR_SDK_VERSION} ${ZEPHYR_SDK_INSTALL_DIR} && \
    echo "==> Running Zephyr SDK Setup..." && \
    cd ${ZEPHYR_SDK_INSTALL_DIR} && \
    ./setup.sh -t arm-zephyr-eabi -h -c > /dev/null && \
    mkdir -p /etc/udev/rules.d/ && \
    cp -r ${ZEPHYR_SDK_INSTALL_DIR}/sysroots/x86_64-pokysdk-linux/usr/share/openocd/contrib/60-openocd.rules /etc/udev/rules.d/

# 5. Fix Permissions so 'vscode' can use the tools in /opt
RUN chown -R vscode:vscode /opt/zephyr-sdk /opt/zephyr-venv

# 6. Final Config
RUN usermod -aG dialout,plugdev vscode
USER vscode
WORKDIR /workspaces
ENV ZEPHYR_TOOLCHAIN_VARIANT=zephyr
