# Zephyr Minimal & C++23 Development Environment

A high-performance, space-optimized DevContainer for Zephyr RTOS development, specifically engineered for modern C++23 and seamless toolchain management.

## 🚀 Key Features & Optimizations

* **Ultra-Lean Base:** Uses `debian:bookworm-slim` to minimize image size and security surface area.
* **Modular SDK Architecture:** Installs only the essential Zephyr SDK components (e.g., ARM Cortex-M) rather than the full 6GB+ multi-arch suite.
* **Blazing Fast Tooling:** Integrates `uv` (Astral's Rust-based Python resolver) for near-instant dependency installation.
* **Modern C++ Suite:** Pre-configured for **C++23** IntelliSense and development with the Zephyr SDK v0.17.4 toolchain.
* **Hardware Ready:** Configured with USB passthrough and `udev` rules for debugging real hardware via OpenOCD.

## 🛠️ Architecture Overview

The environment is defined by three core components:

### 1. [`Dockerfile`](Dockerfile)

Manages the system-level environment:

* **System Deps:** Installs `ninja`, `cmake`, `gperf`, and `ccache` for fast builds.
* **Python:** Sets up a dedicated virtual environment in `/opt/zephyr-venv` using `uv`.
* **Toolchain:** Automatically extracts the ARM Zephyr SDK and runs its setup script.
* **Permissions:** Configures a `vscode` user (UID/GID 1000) with `sudo` and hardware access (`dialout`, `plugdev`).

### 2. [`devcontainer.json`](devcontainer.json)

Orchestrates the VS Code experience:

* **Standards:** Enforces `c++23` and `linux-gcc-arm` IntelliSense modes.
* **Extensions:** Packs essential tools like Zephyr IDE, DeviceTree, Kconfig, and Cortex-Debug.
* **Runtime:** Enables `--privileged` mode and `/dev/bus/usb` mounting for hardware flashing.

### 3. [`devcontainer-requirements.txt`](devcontainer-requirements.txt)

Contains critical Python packages (like `west`) required for the initial bootstrap.

## 🏃 Getting Started

### 1. Launch with DevPod

```bash
devpod up . --ide vscode
```

### 2. Initialize Zephyr Workspace

Inside the container terminal:

```bash
west init -m https://github.com/zephyrproject-rtos/zephyr --mr main .
west update
west zephyr-export
```

## 🔧 Advanced Configuration

### Changing Architectures

To develop for RISC-V or x86 instead of ARM, modify the `Dockerfile`:

```dockerfile
# Change -t flag to your target (e.g., riscv64-zephyr-elf)
RUN cd ${ZEPHYR_SDK_INSTALL_DIR} && ./setup.sh -t riscv64-zephyr-elf -h -c
```

### Compiling with C++23

Update your `prj.conf` to enable the modern standard:

```kconfig
CONFIG_CPP=y
CONFIG_STD_CPP2B=y
CONFIG_REQUIRES_FULL_LIBCPP=y
```

## 🔌 Hardware Debugging

The container is ready for hardware interaction. If you are on **Windows**, ensure you attach your debugger via WSL:

```powershell
usbipd attach --busid <BUS_ID> --wsl
```

On **Linux**, it should be detected automatically if `udev` rules are reloaded on the host.
