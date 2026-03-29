# Zephyr Minimal & C++23 Development Environment

This DevContainer setup provides a highly optimized and minimalistic environment for Zephyr RTOS development, specifically tailored for modern C++ with C++23 support.

## Introduction

Developing with Zephyr often involves significant disk space due to the extensive toolchains and dependencies. This setup addresses that by drastically reducing the DevContainer image size while still providing a fully functional development environment. Key optimizations include using a slim Debian base image and installing only the essential Zephyr SDK components. Additionally, it configures VS Code IntelliSense for C++23, enhancing the development experience for modern C++ projects.

## Key Features and Optimizations

* **`debian:bookworm-slim` Base Image:** Utilizes a highly stripped-down Debian image to minimize the initial footprint.
* **Targeted Zephyr SDK Installation:** Instead of installing the entire multi-architecture Zephyr SDK (which can exceed 6GB), this configuration installs only the specific toolchain you need (e.g., ARM Cortex-M). This is a massive space saver.
* **`uv` for Efficient Python Management:** Leverages `uv`, a fast Python package installer and resolver, for managing Zephyr's Python dependencies.
* **C++23 IntelliSense:** VS Code is configured to provide syntax highlighting, error checking, and code completion based on the C++23 standard.

## Setup Guide

To set up this environment, you will need DevPod installed and configured on your machine.

### Step 1: Configure Your DevContainer Files

Ensure your project includes the following files within a `.devcontainer/` directory. These files define your development environment.

#### [`Dockerfile`](.devcontainer/Dockerfile)

This Dockerfile is central to the minimalistic setup. It:

* Starts with `debian:bookworm-slim` for a lean base.
* Installs minimal system dependencies using `apt-get install -y --no-install-recommends` to avoid unnecessary packages.
* Integrates `uv` for Python virtual environment and package management.
* Crucially, it downloads and installs only a specific Zephyr SDK toolchain. By default, it's configured for `arm-zephyr-eabi`.

**Customization:** If you are developing for a different architecture (e.g., RISC-V), you **must** change the `-t arm-zephyr-eabi` flag in the Zephyr SDK `setup.sh` command within the `Dockerfile` to your desired toolchain (e.g., `riscv64-zephyr-elf`).

#### [`devcontainer.json`](.devcontainer/devcontainer.json)

This file configures your VS Code environment within the DevContainer. Key settings include:

* `C_Cpp.default.cppStandard`: Set to `"c++23"` to enable C++23 IntelliSense.
* `C_Cpp.default.intelliSenseMode`: Set to `"linux-gcc-x64"` for optimal performance.
* A comprehensive list of recommended VS Code extensions for Zephyr development.

### Step 2: Launch and Rebuild Your DevPod Environment

Once your `.devcontainer/` files are set up, launch or rebuild your DevPod environment. This command will build the Docker image defined in your `Dockerfile` and start your VS Code DevContainer.

```bash
devpod up . --ide vscode --rebuild
```

### Step 3: Initialize Zephyr within VS Code

After the DevContainer has launched in VS Code, open a new terminal within VS Code and run the following commands to initialize the Zephyr west manifest and install Python dependencies:

```bash
west init -m https://github.com/zephyrproject-rtos/zephyr --mr main .
west update
west zephyr-export
uv pip install -r zephyr/scripts/requirements.txt
```

## Enabling C++23 for Zephyr Compilation

While the `devcontainer.json` configures VS Code IntelliSense for C++23, to actually **compile** your Zephyr project with C++23, you need to instruct the Zephyr CMake build system. Add the following lines to your project's `prj.conf` file:

```text
# Enable C++ support
CONFIG_CPP=y

# Select C++2B/C++23 standard
# (Note: In Zephyr Kconfig, C++23 is often defined as C++2B depending on your Zephyr release)
CONFIG_STD_CPP2B=y

# Optional: Require standard library support if you intend to use std:: vectors, etc.
CONFIG_REQUIRES_FULL_LIBCPP=y
```

*(Note: Zephyr's GCC 12.2 compiler fully supports experimental C++23 via the `-std=c++2b` flag, which is what the Kconfig above activates).)*
