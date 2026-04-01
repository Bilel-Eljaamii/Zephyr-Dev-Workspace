# Zephyr Dev Environment

This repository provides a highly optimized, minimalistic DevContainer for Zephyr RTOS development, specifically tailored for modern C++23.

## Prerequisites

Before starting, ensure you have the following installed based on your operating system.

### 🪟 Windows

1. **Docker Desktop:** [Download and install](https://www.docker.com/products/docker-desktop/). Ensure "Use the WSL 2 based engine" is checked in Settings.
2. **WSL 2:** Run `wsl --install` in PowerShell.
3. **DevPod:** [Download and install](https://devpod.sh/docs/getting-started/installation).
4. **Git:** [Download and install](https://git-scm.com/download/win).

### 🍎 macOS

1. **Docker Desktop:** [Download and install](https://www.docker.com/products/docker-desktop/).
2. **DevPod:** Install via Homebrew: `brew install devpod`.
3. **Git:** Usually pre-installed, or `brew install git`.

### 🐧 Linux

1. **Docker:** [Install Docker Engine](https://docs.docker.com/engine/install/).
2. **Post-install:** Add your user to the `docker` group: `sudo usermod -aG docker $USER`.
3. **DevPod:** [Download the latest binary](https://devpod.sh/docs/getting-started/installation).
4. **Git:** Install via your package manager (e.g., `sudo apt install git`).

---

## Quick Start Guide

1. **Clone the repository:**

   ```bash
   git clone https://github.com/Bilel-Eljaamii/Zephyr-Dev-Workspace.git
   cd Zephyr-Dev-Workspace
   ```

2. **Launch the environment:**
   Use DevPod to build and open the environment in VS Code:

   ```bash
   devpod up . --ide vscode
   ```

3. **Initialize Zephyr (Inside the container terminal):**
   Once VS Code opens, run the following to set up the Zephyr project:

   ```bash
   west init -m https://github.com/zephyrproject-rtos/zephyr --mr main .
   west update
   west zephyr-export
   ```

## Hardware Debugging (USB Passthrough)

The DevContainer is configured with `--privileged` and `/dev/bus/usb` mapping.

- **Linux:** Works out of the box.
- **Windows:** Use [usbipd-win](https://github.com/dorssel/usbipd-win) to attach USB devices to WSL2.
- **macOS:** USB passthrough to Docker containers is limited; consider using [remote debugging](https://docs.zephyrproject.org/latest/connectivity/usb/device/usb_device.html) or a network-attached debugger.

---

For more technical details about the container architecture, see [.devcontainer/README.md](.devcontainer/README.md).
