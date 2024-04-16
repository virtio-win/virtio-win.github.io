Recommended environment for drivers build: EWDK 1903/2004

**Note:**
* **At the moment of transition to EWDK the drivers are built using EWDK 1903.**
* **Currently usage of EWDK 2004 is blocked only by the problem of building NetKVM with SDV (static driver verifier).**
* **So we prefer to have both EWDKs ready to use (this requires approx. 30G of local storage).**

Building of drivers for following operating systems is not supported by batch build procedure:
* Windows XP / Server 2003
* Windows Vista / Server 2008

Building drivers for Windows 7 / Server 2008R2 is optional.
Use VIRTIO_WIN_BUILD_LEGACY=Win7 to include them into the build.

Note: latest tag that support building of drivers for legacy operating systems:
https://github.com/virtio-win/kvm-guest-drivers-windows/releases/tag/08.03.2021-last-buldable-point-XP

Components to install
* WinFsp https://github.com/billziss-gh/winfsp/releases/download/v1.8/winfsp-1.8.20304.msi (Core + Developer + Kernel Developer)
* Cryprographic provider development kit (CPDK) Windows 10 https://www.microsoft.com/en-us/download/details.aspx?id=30688
_(Note: The kit was updated, latest one was published 3/22/2021 and installs headers and binaries under ...\Windows Kits\10.0\...)_
* .NET Framework version 4.8 https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer
* Visual Studio 2013 redistributable x86 (required by EWDK/SDV) [KB 3138367](http://download.microsoft.com/download/c/c/2/cc2df5f8-4454-44b4-802d-5ea68d086676/vcredist_x86.exe)

Preparing the environment:
1. Download EWDK 1903 ISO https://go.microsoft.com/fwlink/p/?linkid=2086136
* mount it (for example as E:)
* copy entire content to local directory (for example c:\ewdk1903): `mkdir c:\ewdk1903 && xcopy /e e:\* c:\ewdk1903`
* unmount and delete EWDK 1903 ISO file
* if the name of the local directory is different from the default (c:\ewdk1903) `set EWDK1903_DIR=<Copy_Of_EWDK 1903>`

2. Download EWDK 2004 ISO https://go.microsoft.com/fwlink/p/?linkid=2128902
* mount it (for example as E:)
* copy entire content to local directory (for example c:\ewdk2004): `mkdir c:\ewdk2004 && xcopy /e e:\* c:\ewdk2004`
* unmount and delete EWDK 2004 ISO file
* if the name of the local directory is different from the default (c:\ewdk2004) `set EWDK2004_DIR=<Copy_Of_EWDK 2004>`
* replace the DVL files of EWDK 2004 by ones of EWDK 1903 
* copy /y c:\ewdk1903\Program Files\Windows Kits\10\Tools\dvl\\* c:\ewdk2004\Program Files\Windows Kits\10\Tools\dvl

Build procedure:
* Run buildAll.bat (for complete build including SDV)
* Run build_AllNoSdv.bat (to build everything excluding SDV)
