# Overview

Virtiofs is a shared file system that lets virtual machines access a directory tree on the host. More information on the underlying approach is available at [virtio-fs.gitlab.io](https://virtio-fs.gitlab.io/). Virtiofs for Windows is a user mode file system, implemented using [WinFsp framework](https://github.com/billziss-gh/winfsp). Virtiofs consists of VirtIO-powered driver and user-space service based on WinFsp.

# Status

Virtiofs is at an early stage of development and should be considered as a "Tech Preview" feature. To see what may not work, please check [known limitations](#known-limitations).

# Setup

## Host

This section shows how to setup virtiofs device on the host side. Two options are described: libvirt and QEMU.

### libvirt

Following XML should be added to your libvirt VM descrition:

```xml
<domain>
  ...
  <memoryBacking>
    <source type='memfd'/>
    <access mode='shared'/>
  </memoryBacking>
  ...
  <devices>
    <filesystem type="mount" accessmode="passthrough">
      <driver type="virtiofs" queue="1024"/>
      <source dir="/home/user/viofs"/>
      <target dir="mount_tag"/>
      <address type="pci" domain="0x0000" bus="0x06" slot="0x00" function="0x0"/>
    </filesystem>
  </devices>
  ...
</domain>
```

The `<memoryBacking>` is necessary. Element `<source dir="/home/user/viofs"/>` describes host directory to share.

More information on libvirt virtiofs options is provided in [libvirt docs](https://libvirt.org/kbase/virtiofs.html).

### QEMU

Run virtiofsd daemon:

```
/usr/libexec/virtiofsd --socket-path=/tmp/virtiofs_socket -o source=/home/user/viofs
```

Adjust following QEMU command-line parameters:

* Instantiate the character device for socket communication between QEMU and virtiofsd:
`-chardev socket,id=char0,path=/tmp/virtiofs_socket`
* Instantiate the virtiofs PCI device:
`-device vhost-user-fs-pci,queue-size=1024,chardev=char0,tag=my_virtiofs`
* Force use of memory sharing with virtiofsd (replace `4G` with your desired VM RAM size):
`-m 4G -object memory-backend-file,id=mem,size=4G,mem-path=/dev/shm,share=on -numa node,memdev=mem`

### virtiofsd

Modern virtiofsd is written in Rust and supported here: https://gitlab.com/virtio-fs/virtiofsd

To build virtiofsd:
```
cargo build
```


## Guest

### Setup with installer
1. Download and install [WinFSP](https://github.com/billziss-gh/winfsp/releases) with at least "Core" feature enabled.
2. Install virtiofs driver and service from [VirtIO-Win package](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md).

### Manual setup (for development purposes)
1. Download and install [WinFSP](https://github.com/billziss-gh/winfsp/releases) with at least "Core" feature enabled. If you plan to make changes to virtiofs driver or service then enable "Core", "Developer" and "Kernel Developer" features in the installer.
2. Install virtiofs driver with Device Manager or `pnputil.exe`.
3. Setup virtiofs service by running `sc create VirtioFsSvc binPath="<path to the binary>\virtiofs.exe" start=auto depend=VirtioFsDrv`. Don't forget to appropriately set `binPath`.
4. You can immediately start the service by running `sc start VirtioFsSvc`. The service will start automatically on boot. Virtiofs uses the first available drive letter starting with `Z:` unless no mount-point is specified.

![](https://user-images.githubusercontent.com/8286747/151011858-c9d122d2-d95a-421c-9914-c7d4dd05e2e3.png)

![](https://user-images.githubusercontent.com/8286747/151012458-fb8fa2c9-5059-45ac-994b-4ca03619a27b.png)

![](https://user-images.githubusercontent.com/8286747/151011678-88e804d8-4231-4597-932c-91b18e492492.png)

### Options

Virtiofs service can parse following settings from command-line:
```
  -d DebugFlags       [-1: enable all debug logs]
  -D DebugLogFile     [file path; use - for stderr]
  -i                  [case insensitive file system]
  -F FileSystemName   [file system name for OS]
  -m MountPoint       [X:|* (required if no UNC prefix)]
  -t Tag              [mount tag; max 36 symbols]
  -o UID:GID          [host owner UID:GID]
```

Since command-line arguments can't be assigned to Windows service permanently, virtiofs can parse them from the registry. When command-line arguments are absent the service looks up for the following parameters under `HKLM\Software\virtiofs`:
* `DebugFlags` (DWORD)
* `DebugLogFile` (String)
* `MountPoint` (String)
* `CaseInsensitive` (DWORD)
* `Owner` (String)
* `FileSystemName` (String)

For example, registry values depicted below correspond to `virtiofs.exe -d -1 -D C:\viofs_debug.log -m X:`. Please note that `-1` corresponds to `0xffffffff` in DWORD value. 
![](https://user-images.githubusercontent.com/8286747/146226495-0d7614ca-8a7d-4465-9aa3-3dc9dc9cb6de.png)

Also, parameters named `OverflowUid` and `OverflowGid` are always parsed from the registry. Normally, these parameters only affect if the host daemon is running inside a Linux user namespace. They denote [UID and GID perceived as `nobody` on the host](https://www.kernel.org/doc/Documentation/sysctl/fs.txt), so they should be in sync with corresponding host values. If the service finds out shared folder root owner UID/GID becomes `nobody`, it will try previous UID/GID as owner for new files and folders. They assumed to be `65534` if no such parameters are found in the registry.

### Case insensitivity

*At the moment, the case insensitivity feature is under development and may not work properly in some cases.* Case insensitivity can be turned on by `-i` command-line parameter of `CaseInsensitive` registry key. In this situation, when processing a request, virtiofs first tries to access the file or directory by its exact name, as in case-sensitive mode. Second, if there is no such node, virtiofs finds a node with a matching name in the parent directory, ignoring case.

### File system name

When running an executable as administrator, the Windows OS seems to require that the name of the file system that is housing the executable is "NTFS". This can be done by `-F` command-line parameter or `FileSystemName` registry key.

### Multiple virtiofs instances

#### Setup

Support for multiple virtiofs instances is made by WinFSP.Launcher service, so virtiofs own service should not be running:
```
sc stop VirtioFsSvc
sc config VirtioFsSvc start=demand
```
The virtiofs service is now stopped and will not start even after reboot.

Virtiofs configuration for WinFsp.Launcher:
```
"C:\Program Files (x86)\WinFsp\bin\fsreg.bat" virtiofs "<path to the binary>\virtiofs.exe" "-t %1 -m %2"
```
Corresponding data is now available to view and edit under `HKLM\SOFTWARE\WOW6432Node\WinFsp\Services\virtiofs` registry key.

#### Mount

Mount virtiofs with tag `mount_tag0` to `Y:\`:
```
"C:\Program Files (x86)\WinFsp\bin\launchctl-x64.exe" start virtiofs viofsY mount_tag0 Y:
```
Mount virtiofs with tag `mount_tag1` to `Z:\`:
```
"C:\Program Files (x86)\WinFsp\bin\launchctl-x64.exe" start virtiofs viofsZ mount_tag1 Z:
```
Here `viofsY` and `viofsZ` are instance names for WinFsp.Launcher. They are selected arbitrary, but must differ between instances.
![image](https://user-images.githubusercontent.com/8286747/190429409-b4c42336-4292-4d91-8cb0-292736b17307.png)
#### Unmount

Unmount is done by the instance name:
```
"C:\Program Files (x86)\WinFsp\bin\launchctl-x64.exe" stop virtiofs viofsY
"C:\Program Files (x86)\WinFsp\bin\launchctl-x64.exe" stop virtiofs viofsZ
```
# Known limitations

* Case insensitivity may not work properly

# Testing

## WinFsp Tests

The following command line is used to execute WinFsp's test suite:

```
winfsp-tests-x64.exe --external --resilient -create_fileattr_test -create_readonlydir_test -create_allocation_test -create_notraverse_test -getfileattr_test -getfileinfo_test -getfileinfo_name_test -setfileinfo_test -setsecurity_test -reparse_guid_test -reparse_nfs_test -stream_*
```

Virtiofs for Windows does not currently pass all available tests so some of them are skipped for now:

| Test Name | Failure Description |
|---|---|
| create_fileattr_test | Can't set FILE_ATTRIBUTE_SYSTEM. |
| create_readonlydir_test | Can't create a file on a read-only directory. |
| create_allocation_test | Can't set a file allocation to zero. |
| create_notraverse_test |  |
| getfileattr_test | Not all Windows' file attributes exists. |
| getfileinfo_test | File name length is longer than expected. File system is mounted as network drive? |
| getfileinfo_name_test |  |
| setfileinfo_test | Can't remove FILE_ATTRIBUTE_ARCHIVE as it is hard-coded. |
| setsecurity_test | Not all Window's security descriptors are implemented.  |
| reparse_guid_test |  |
| reparse_nfs_test |  |
| stream_* | NTFS-like file streams are not supported. Need to decide where and how the information is stored. |