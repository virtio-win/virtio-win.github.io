# QEMU Guest Agent (qemu-ga) on Windows

## Overview
The QEMU Guest Agent (qemu-ga) is a service that runs inside a guest VM and facilitates communication between the host and the guest OS over a virtio-serial channel. It enables the host to execute structured commands that require coordination with the guest OS, such as initiating a graceful shutdown, freezing the filesystem for snapshots, or retrieving system information.

This document covers how to compile qemu-ga for Windows targets from a Linux host (cross-compilation), how to install the resulting binaries, and how to interact with the agent from the host side.


## Prerequisites and Dependencies
### On Linux Host
To build `qemu-ga` for Windows using cross-compilation with MinGW, you need to install the appropriate compilers, headers, libraries, and build tools.

The following packages are required:

- libtool
- zlib-devel
- glib2-devel
- python3-devel
- gettext
- gettext-devel
- meson
- ninja-build
- msitools >= 0.93.93
- mingw32-gcc ≥ 7.4.0
- mingw32-gcc-c++ ≥ 7.4.0
- mingw64-gcc ≥ 7.4.0
- mingw64-gcc-c++ ≥ 7.4.0
- mingw32-glib2 ≥ 2.78.0
- mingw64-glib2 ≥ 2.78.0
- mingw32-headers ≥ 10.0.0
- mingw64-headers ≥ 10.0.0
- mingw-w64-tools ≥ 10.0.0

> **Note:** The version constraints are important — some packages include fixes required for QEMU Guest Agent to build and function properly. 


## Configuring QEMU / libvirt for QGA
To enable communication between the QEMU Guest Agent inside the Windows guest and the host system, the virtual machine must expose a **virtio-serial** device with a dedicated channel named `org.qemu.guest_agent.0`.
This channel allows the host to send JSON commands to the guest.

### 1. Direct QEMU CLI
Add the following options:

```bash
...
 -chardev socket,path=/tmp/qga.sock,server=on,wait=off,id=qga0 \
 -device virtio-serial \
 -device virtserialport,chardev=qga0,name=org.qemu.guest_agent.0
```
### 2. libvirt XML Configuration
If the VM is managed with **libvirt**, add a `<channel>` element to the domain XML:

```xml
<devices>
    ...
    <channel type='unix'>
        <source mode='bind'/>
        <target type='virtio' name='org.qemu.guest_agent.0'/>
    </channel>
</devices>
```

## How to Compile QGA from Source
To build the QEMU Guest Agent (`qemu-ga`) for Windows from source, you can use the traditional `configure` and `make` build system with MinGW cross-compilation tools on a host.

#### 1. Clone the QEMU source

```bash
git clone https://github.com/qemu/qemu.git
cd qemu
```

#### 2. Build
You can build both 32-bit and 64-bit versions of the guest agent using the appropriate MinGW cross-compilers.  

* Build for 32-bit Windows:
  ```bash
    ./configure \
        --disable-docs \
        --disable-system \
        --disable-user \
        --cross-prefix=i686-w64-mingw32- \
        --enable-guest-agent \
        --enable-guest-agent-msi \
        --enable-qga-vss || cat config.log

    make qemu-ga
  ```
   The following output files will be generated:
  * build/qga/qemu-ga.exe — 32-bit agent binary
  * build/qga/qemu-ga-i386.msi — optional 32-bit Windows installer (if MSI is enabled)

* Build for 64-bit Windows:
  ```bash
    ./configure \
        --disable-docs \
        --disable-system \
        --disable-user \
        --cross-prefix=x86_64-w64-mingw32- \
        --enable-guest-agent \
        --enable-guest-agent-msi \
        --enable-qga-vss || cat config.log

    make qemu-ga
  ```
  The following output files will be generated:
  * build/qga/qemu-ga.exe — 64-bit agent binary
  * build/qga/qemu-ga-x86_64.msi — optional 64-bit Windows installer (if MSI is enabled)

## How to Install QGA
The QEMU Guest Agent can be installed inside a Windows virtual machine in two ways.  
Before installation, **verify that the VirtIO-serial driver is installed and working**.


### Option A: Using the MSI Installer (Recommended)

If you built the agent with `--enable-guest-agent-msi`, you can install it like any standard Windows package:

1. Copy the generated MSI from the Linux host to the Windows guest (For example, using shared folders).
2. Run the installer in the guest:
    ```powershell
    msiexec /i qemu-ga-x86_64.msi /qn /norestart
    ```

### Option B: Manual Installation from `.exe` (Without MSI)

You can also install the agent manually by copying the built `qemu-ga.exe` **together with its required DLLs** from the host to the Windows guest,  
and then registering it as a Windows service.  
Using the MSI is easier because it automatically includes all required files and sets up the service.


## How to Run QGA Commands from the Host


After installation, ensure that the **QEMU Guest Agent** service is running.
Once the QEMU Guest Agent is installed and running in the Windows guest,  
you can communicate with it from the host to perform tasks such as graceful shutdown,  
querying guest information, or freezing filesystems for consistent snapshots.
Commands are sent as **JSON objects**.

### Option A: Using `virsh` (with libvirt)

If your VM is managed by libvirt,
you can interact with the agent using:

```bash
virsh qemu-agent-command <vm-name> '<JSON command>'
```

#### Example: Check connectivity
```bash
virsh qemu-agent-command win-vm '{"execute":"guest-ping"}'
```
Expected output: {}

### Option B: Using a UNIX socket (direct QEMU)

If you started QEMU manually and added a virtio-serial port for the agent you can send commands directly through the socket using socat:
```bash
echo '{"execute":"guest-ping"}' | socat - UNIX-CONNECT:/tmp/qga.sock
```

### Common Commands

| Command JSON                                   | Description                                      | Example Output (shortened)                     |
|------------------------------------------------|--------------------------------------------------|------------------------------------------------|
| `{"execute":"guest-ping"}`                     | Test connectivity                                | `{}`                                           |
| `{"execute":"guest-info"}`                     | General info about guest & agent                 | `{"version":"8.2.0","supported_commands":...}` |
| `{"execute":"guest-get-osinfo"}`               | Detailed Windows OS info                         | `{"name":"Microsoft Windows 10 Pro", ...}`     |
| `{"execute":"guest-shutdown"}`                 | Graceful shutdown                                |                                                |
| `{"execute":"guest-shutdown","arguments":{"mode":"reboot"}}` | Graceful reboot                    |                                                |
| `{"execute":"guest-fsfreeze-freeze"}`          | Freeze all filesystems for snapshot              | `{"return":1}` (number of frozen filesystems)  |
| `{"execute":"guest-fsfreeze-thaw"}`            | Thaw frozen filesystems                          | `{"return":1}` (number of thawed filesystems)  |
| `{"execute":"guest-fsfreeze-status"}`          | Query freeze status                              | `{"return":"thawed"}` or `"frozen"`            |
| `{"execute":"guest-get-fsinfo"}`               | List mounted filesystems                         | JSON array with drive letters and info         |
| `{"execute":"guest-get-time"}`                 | Get guest system time (ns)                       | `{"return":1693149886000000000}`               |


> **For the complete list of available QGA commands,**  
> see the [QEMU Guest Agent QAPI schema](https://gitlab.com/qemu-project/qemu/-/blob/master/qga/qapi-schema.json).

---
