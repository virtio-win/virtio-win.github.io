# Steps required for building

## For Windows 10, Windows Server 2022
1.  Download Windows 11 21H2 EWDK ISO (https://go.microsoft.com/fwlink/?linkid=2202360) and mount it (let's say to `E:\`). You can download the ISO to the host and connect it to the VM as CD-ROM.

## For Windows 11
2. Download Windows 11 22H2 EWDK ISO (https://go.microsoft.com/fwlink/?linkid=2249942) and mount it (let's say to `F:\`). You can download the ISO to the host and connect it to the VM as CD-ROM.

## General steps

3. Download and install WinFSP (https://github.com/billziss-gh/winfsp/releases/tag/v2.0) with "Core", "Developer" and "Kernel Developer" features enabled.
4. Download and install CPDK 8.0 (https://www.microsoft.com/en-us/download/details.aspx?id=30688).
5. Download and install .NET Framework version 4.8 (https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer). Windows Server 2022 and Windows 11 don't need it.
6. Change directory to `kvm-guest-drivers-windows`.
7. Run `build_AllNoSdv.bat [Win10|Win11]` depending on your Windows version.
   - Choose `Win10` for Windows 10, Windows Server 2022 (x86, x64, ARM64)
   - Choose `Win11` for Windows 11 (x86 (several UMD only), x64, ARM64)
8. Run `Tools\signAll.bat` to sign drivers. Loading of test-signed drivers must be enabled on your system (https://docs.microsoft.com/en-us/windows-hardware/drivers/install/the-testsigning-boot-configuration-option).
9. Find drivers/services in folders named `Install` inside driver folders.



# Additional steps for driver development

1. Copy Windows 11, version 22H2 EWDK to a local directory
   * Download Windows 11, version 22H2 EWDK (released October 24, 2023) with Visual Studio Buildtools 17.1.5 ISO (https://go.microsoft.com/fwlink/?linkid=2249942) and mount it (let's say to `E:\`).
   * copy entire content to local directory (c:\ewdk11_22h2): `mkdir c:\ewdk11_22h2 && xcopy /e e:\* c:\ewdk11_22h2`
   * unmount and delete Windows 11 22H2 EWDK ISO file
   * `set EWDK11_22H2_DIR=c:\ewdk11_22h2`

1. Copy Windows 11, version 21H2 EWDK to a local directory
   * Download Windows 11 21H2 EWDK with Visual Studio Build Tools 16.11.10 ISO (https://go.microsoft.com/fwlink/?linkid=2202360) and mount it (let's say to `E:\`).
   * copy entire content to local directory (c:\ewdk11): `mkdir c:\ewdk11 && xcopy /e e:\* c:\ewdk11`
   * unmount and delete Wirndows 11 21H2 EWDK ISO file
   * `set EWDK11_DIR=c:\ewdk11`

1. Copy DVL from EWDK for Windows 10, version 1903 to a local directory
   * Download EWDK for Windows 10, version 1903 with Visual Studio Build Tools 16.0 ISO (https://go.microsoft.com/fwlink/p/?linkid=2086136) and mount it (let's say to `E:\`).
   * copy all files (`dvl.exe`, `Microsoft.StaticToolsLogo.ObjectModel.dll`) from `E:\Program Files\Windows Kits\10\Tools\dvl` to `C:\dvl1903`
   * unmount and delete Wirndows 10 version 1903 EWDK ISO file

1. Download and install WinFSP 2023 (https://github.com/winfsp/winfsp/releases/tag/v2.0) with the "Core", "Developer" and "Kernel Developer" feature enabled.
1. Download and install CPDK 8.0 (https://www.microsoft.com/en-us/download/details.aspx?id=30688).
1. Download and install .NET Framework version 4.8 (https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer). Windows Server 2022 and Windows 11 don't need it.

1. [CodeQL Windows Setup](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/static-tools-and-codeql#codeql-windows-setup) for Windows 11 22H2
   * mkdir C:\codeql-home
   * Navigate to the GitHub [CodeQL Download Page](https://github.com/github/codeql-cli-binaries/releases/tag/v2.6.3)
   * Download version **2.6.3** of the zip file if you are certifying a driver for the Windows Hardware Compatibility Program in 2023. For example for 64-bit Windows "codeql-win64.zip". 
   * Unzip the CodeQL folder in the zip file to the `C:\codeql-home\codeql\` directory.
   * Confirm that the CodeQL command works by displaying the help. `C:\codeql-home\codeql\>codeql --help`

1. [Clone the repository to access the driver-specific queries](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/static-tools-and-codeql#clone-the-repository-to-access-the-driver-specific-queries)
   * Navigate to the [Microsoft CodeQL GitHub repository](https://github.com/microsoft/Windows-Driver-Developer-Supplemental-Tools).
   * [Clone](https://github.com/git-guides/git-clone) the repository to download all CodeQL queries and [query suites](https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/) with driver-specific queries.  
   ```batch
   C:\codeql-home\>git clone https://github.com/microsoft/Windows-Driver-Developer-Supplemental-Tools.git --recursive -b WHCP_22H2
   ```

Branch depending on the OS you are certifying for, per the following table:

| Release                         | Branch to use   |
|---------------------------------|-----------------|
| Windows Server 2022             | WHCP_21H2       |
| Windows 11                      | WHCP_21H2       |
| Windows 11,  version 22H2       | WHCP_22H2       |

# Notes on the build batch

The building of drivers for the following operating systems is not supported by the batch build procedure:
   * Windows XP / Server 2003
   * Windows Vista / Server 2008
   * Windows 7 / Server 2008R2 
   * Windows 8 / Server 2012
   * Windows 8.1 / Server 2012R2

Note: latest tag that supports the building of drivers for legacy operating systems:
   * https://github.com/virtio-win/kvm-guest-drivers-windows/releases/tag/08.03.2021-last-buldable-point-XP
   * https://github.com/virtio-win/kvm-guest-drivers-windows/releases/tag/02.12.2021-last-buildable-point-Win7
   * https://github.com/virtio-win/kvm-guest-drivers-windows/releases/tag/01.11.2023-last-buildable-point-Win8

Run `buildAll.bat` for the complete build including SDV. Note that an SDV build takes a long time.

Set VIRTIO_WIN_SDV_2022=1 to build DVL for system device drivers.
