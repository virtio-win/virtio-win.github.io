# Windows 11 VM installation on ARM64 with qemu

This guide explains how to install and run a Windows 11 for ARM64 virtual machine with `qemu-system-aarch64`.

## Prerequisites

### Windows ISO (ARM64)
- Download the Windows 11 (ARM64) ISO from Microsoft’s download page: [Windows 11 for Arm-based PCs](https://www.microsoft.com/en-us/software-download/windows11arm64).

### Virtio drivers
- Download the latest `virtio-win` driver ISO from the repository page: [virtio-win](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md). It will be used during setup to provide the storage and network drivers.

### EFI
- Install the AArch64 UEFI firmware from your distribution (packages: `qemu-efi-aarch64` or `edk2-aarch64`). This provides `QEMU_EFI.fd`.
- If it is not available in your distro, download the Debian package: [qemu-efi-aarch64](https://packages.debian.org/sid/qemu-efi-aarch64) and extract it using:

```bash
ar x qemu-efi-aarch64*.deb
tar xvf data.tar.xz
cp ./usr/share/qemu-efi-aarch64/QEMU_EFI.fd .
```

### Create a virtual disk
```bash
qemu-img create win11-arm64.img 100G
```

You should now have the following files in your working directory:
- `Win11_*.iso`
- `virtio-win-*.iso`
- `QEMU_EFI.fd`
- `win11-arm64.img`

## Run QEMU

### Option A: Software emulation (TCG)

```bash
qemu-system-aarch64 \
  -M virt -m 8G -cpu max,pauth-impdef=on -smp 8 \
  -bios ./QEMU_EFI.fd\
  -accel tcg,thread=multi\
  -device ramfb \
  -device qemu-xhci -device usb-kbd -device usb-tablet \
  -nic user,model=virtio-net-pci \
  -device usb-storage,drive=install \
  -drive if=none,id=install,format=raw,media=cdrom,file=./Win11_*.iso \
  -device usb-storage,drive=virtio-drivers \
  -drive if=none,id=virtio-drivers,format=raw,media=cdrom,file=./virtio-win-*.iso \
  -drive if=virtio,id=system,format=raw,file=./win11-arm64.img  
```

### Option B: Hardware acceleration (KVM, aarch64 host)

Requires a Linux aarch64 host with KVM enabled. On x86_64 hosts, use Option A.
From the Option A command, change:
- `-accel tcg,thread=multi` -> `-accel kvm`
- `-cpu max,pauth-impdef=on` -> `-cpu host`
Everything else remains the same.


## Windows setup walkthrough

1. Boot from the Windows ISO
   - When prompted, press any key to boot from the installer.
2. Start installation
   - Choose language, click Install. When asked for a key, click “I don’t have a product key”.
   ![Windows setup: language and Install](https://github.com/user-attachments/assets/612667af-ecc9-4c8a-9157-dabeb18e541f)
   
3. Bypass Windows 11 checks (if shown)
   - If you see “This PC can’t run Windows 11”, press Shift+F10 → run `regedit`.
   - Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\Setup`.
   - Right-click `Setup` → New → Key → name it `LabConfig`.
   - In `LabConfig`, create the following DWORD (32-bit) values:
     - Must:
       - `BypassSecureBootCheck` = `1`
     - Can be done for convenience (only if you still see the error):
       - `BypassCPUCheck` = `1`
       - `BypassRAMCheck` = `1`
       - `BypassStorageCheck` = `1`
       - `BypassTPMCheck` = `1`
   - Close Registry Editor and the command prompt, go back to the previous page, and click “I don’t have a product key” again. Installation should proceed.

   - Important: Some Windows 11 images do not show a Back button. If so, complete this bypass (create `LabConfig` and set the 5 DWORDs to `1`) before clicking “I don’t have a product key”.

  ![Compatibility error screen](https://github.com/user-attachments/assets/ddd9e8f7-7bd9-44d8-aaf4-03e4dc581cd2)
  ![Registry Editor: LabConfig with bypass keys](https://github.com/user-attachments/assets/f3787180-c16a-4ddc-ade5-10dad7cb5342)

4. Storage driver and disk selection
   - Choose “Custom Install”.
   - Click “Load driver” → browse the `virtio-win` ISO → `viostor/w11/ARM64/` → load the driver.
   - Create a partition on the virtio disk and click Next.

   ![Load driver → viostor/w11/ARM64](https://github.com/user-attachments/assets/f9e42d73-5e8d-4310-9a99-f397ffb61bdb)
   ![Disk selection with virtio disk visible](https://github.com/user-attachments/assets/28b15e62-cb57-48de-8119-cc0bfd055e14)

5. Installation progress
   - The installer copies files and reboots several times.

   ![Installing Windows: progress screen](https://github.com/user-attachments/assets/d176684b-c197-4cc3-a2a9-d4f64c2698e6)

6. Continue without an internet connection
   - If the OOBE insists on a network connection, press Shift+F10 and run:

```bat
OOBE\BYPASSNRO
```

   - The system will reboot and offer an option to proceed without a network.

7. First boot to the Windows desktop
   - After the reboot, choose “I don’t have internet” → “Continue with limited setup”.
   - Create a local user and finish the OOBE. You should arrive at the Windows desktop.

8. Get a network connection
   - Open Device Manager → expand “Other devices” → right‑click “Ethernet Controller” → Update driver.
   - Browse to the virtio ISO NetKVM folder (e.g., `E:\NetKVM`), and install the driver.
   - After the driver installs, the adapter should obtain network connectivity.
