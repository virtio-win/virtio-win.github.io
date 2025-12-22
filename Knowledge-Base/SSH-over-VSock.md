# SSH over VSock with Windows Guests

This guide explains how to establish an SSH connection to a Windows guest VM over VSock. This method bypasses the need for a network interface (NIC) on the guest, allowing management even if networking is misconfigured or disabled.

This guide includes setup instructions for QEMU and libvirt.

## How It Works

* **VSock** is used as a fast, host-to-guest transport layer.
* The **vstbridge** service listens on the VSock channel and forwards traffic to TCP localhost:22.
* The SSH client on the host uses a proxy (like socat or libvirt-ssh-proxy) to communicate with this bridge.

## Prerequisites
### Host
* Virtualization platform supporting VSock (e.g., QEMU/KVM with virtio-vsock)
* Tools: socat (for manual connection) or libvirt-ssh-proxy (for automatic connection).
### Guest (Windows)
* Windows 10/11 or Windows Server 2019/2022/2025 with enabled SSH service

## Step 1: Configure VM Hardware (Host Side)

Before booting the VM, ensure the VSock device is attached.

### QEMU

Add a virtio-vsock device to your QEMU command line:

```bash
-device vhost-vsock-pci,id=vhost-vsock-pci0,guest-cid=3
```

Replace `3` with your desired guest CID (Context Identifier). The CID must be unique for each VM and should be greater than 2.

### libvirt

Add the following XML configuration to your VM definition:

```xml
<domain>
  ...
  <devices>
    <vsock model='virtio'>
      <cid auto='no' address='3'/>
    </vsock>
  </devices>
  ...
</domain>
```

You can also use `auto='yes'` to let libvirt automatically assign a CID.

## Step 2: Install Drivers & Services (Guest Side)

### 2.1. Install the viosock driver

Install the viosock driver using any method described in [Driver installation](https://virtio-win.github.io/Knowledge-Base/Driver-installation). For example, via Device Manager:
1. Open Device Manager.
2. Look for a device with a yellow triangle icon under "Other devices" (the Hardware ID is shown in the screenshot below, which you can verify by right-clicking the device → **Properties** → **Details** tab → **Hardware Ids**).
  ![Image](https://github.com/user-attachments/assets/608a9c5b-784e-487f-b1c7-977e89914e43)
1. Right-click → **Update Driver** → **Browse my computer** → fill in the appropriate directory where the downloaded driver files are located.

### 2.2. Enable Windows OpenSSH Server

Enable the built-in SSH server to listen for connections. The bridge will forward VSock traffic to this service.

#### Installation via PowerShell

1. Open PowerShell as Administrator.

2. Check if OpenSSH is already installed:

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

The command should return the following output if neither are already installed:

```powershell
Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent
Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```

3. Install the SSH server component (and optionally the client):

```powershell
# Install the OpenSSH Server (required)
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Install the OpenSSH Client (optional - only needed if you want to SSH from this Windows guest to other machines)
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

Each command should return the following output:

```powershell
Path          :
Online        : True
RestartNeeded : False
```

4. Start the SSH service and set it to start automatically:

```powershell
# Start the sshd service
Start-Service sshd

# Set to start automatically on boot (recommended)
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the Firewall rule is configured (should be created automatically by setup)
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}
```

**Note:** For GUI-based installation or additional configuration options, see the [Microsoft OpenSSH documentation](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse).

### 2.3. Install the VSock-TCP Bridge Service (vstbridge)
The `vstbridge.exe` utility links the VSock channel to the SSH server.

1. Locate `vstbridge.exe`. This file is included with the virtio-win package in the directory where the viosock driver files are located.
2. Open CMD or PowerShell as **Administrator** in that folder.
3. Install the service:

```bat
vstbridge.exe -i
```

The service will start automatically and listen for VSock connections on port 22.

## Step 3: Connect to the Guest (Host Side)

### Method 1: Manual Connection via socat

This method uses `socat` to proxy the connection directly to the guest's VSock CID.

```bash
ssh -o ProxyCommand="socat - VSOCK-CONNECT:<cid>:22" <win_user>@0.0.0.0
```

Replace:
* `<cid>` with the VSock CID of the guest VM.
* `<win_user>` with the actual username in the Windows guest.


### Method 2: Libvirt SSH Proxy

If your libvirt version supports the SSH proxy scheme and the `libvirt-ssh-proxy` package is installed, you can use this simplified method.

```bash
ssh <win_user>@qemu/<vm_name>
```

Replace:
* `<win_user>` with the Windows guest username.
* `<vm_name>` with your VM name as shown in `virsh list`.


For more information, see the [Libvirt SSH Proxy Documentation](https://libvirt.org/kbase/ssh-proxy.html).