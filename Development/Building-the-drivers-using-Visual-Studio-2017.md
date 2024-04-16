# Tools needed for driver build

## Building virtio-win drivers (including ARM64) using Visual Studio 2017

### [1] install Visual Studio 2017 Community
[Download link](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&rel=15)

**Select 'Desktop development with C++'**
**Add following 'Individual components':**

_From 'Compilers, build tools, and runtimes'_

* VC++ 2017 Libs for Spectre (ARM)
* VC++ 2017 Libs for Spectre (ARM64)
* VC++ 2017 Libs for Spectre (x86/x64
* Visual C++ compiler and libraries for ARM
* Visual C++ compiler and libraries for ARM64
* Windows Universal CRT SDK 
* Windows XP Support for C++

_From 'SDK, libraries and frameworks'_

* Visual C++ ATL (x86/x64) with Spectre Mitigation
* Visual C++ ATL for ARM with Spectre Mitigation
* Visual C++ ATL for ARM64 with Spectre Mitigation
* Visual C++ MFC for x86/x64 with Spectre Mitigations
* Visual C++ MFC support for ARM64 with Spectre Mitigation
* Windows 10 SDK (10.0.17763.0)
* Windows 8.1 SDK
* Windows Universal C Runtime

Exported config of working VS2017 setup:

`{`
	`"version": "1.0",`
	`"components": [`
		`"Microsoft.VisualStudio.Workload.NativeDesktop",`
		`"microsoft.visualstudio.component.debugger.justintime",`
		`"microsoft.visualstudio.component.vc.diagnostictools",`
		`"microsoft.visualstudio.component.vc.cmake.project",`
		`"microsoft.visualstudio.component.vc.testadapterforboosttest",`
		`"microsoft.visualstudio.component.vc.testadapterforgoogletest",`
		`"microsoft.visualstudio.component.windows81sdk",`
		`"microsoft.visualstudio.componentgroup.nativedesktop.winxp",`
		`"microsoft.visualstudio.component.vc.atlmfc",`
		`"microsoft.visualstudio.component.windows10sdk",`
		`"microsoft.visualstudio.component.vc.atl.arm64.spectre",`
		`"microsoft.visualstudio.component.vc.atlmfc.spectre",`
		`"microsoft.visualstudio.component.vc.mfc.arm.spectre",`
		`"microsoft.visualstudio.component.vc.runtimes.arm.spectre",`
		`"microsoft.visualstudio.component.vc.runtimes.arm64.spectre",`
		`"microsoft.visualstudio.component.vc.runtimes.x86.x64.spectre",`
		`"microsoft.visualstudio.component.vc.atl.arm",`
		`"microsoft.visualstudio.component.vc.mfc.arm",`
		`"microsoft.visualstudio.component.vc.mfc.arm64"`
	`]`
`}`

Note: if you save the above config as "c:\temp\virtio-win-drivers.vsconfig", you can use this Chocholatey command to setup VS2017 more easily:
`choco install -y visualstudio2017community --package-parameters "--config c:\temp\virtio-win-drivers.vsconfig"`

### [2] install WDK 1809
[Download link](https://go.microsoft.com/fwlink/?linkid=2026156)
* On last step confirm installing WDK VS2017 extension. If due to some reason this step fails, run it manually from C:\Program Files (x86)\Windows Kits\10\Vsix

### [3] install WDK for Windows 7 (required to build drivers for legacy OS, i.e. XP, WNET, Vista)
[Download link (ISO)](https://download.microsoft.com/download/4/A/2/4A25C7D5-EFBE-4182-B6A9-AE6850409A78/GRMWDK_EN_7600_1.ISO)
* run KitSetup.exe, use C:\WinDDK\7600.16385.1 as installation directory

### [4] install Microsoft Cryptographic Provider Development Kit (CPDK) (required to build virtio-rng, virtio-crypt drivers)
[Download link](https://download.microsoft.com/download/1/7/6/176909B0-50F2-4DF3-B29B-830A17EA7E38/CPDK_RELEASE_UPDATE/cpdksetup.exe)
* run cpdksetup.exe

### [5] install WinFsp 2020 (required both to build and use virtio-fs driver)

[Download link](https://github.com/billziss-gh/winfsp/releases/download/v1.6/winfsp-1.6.20027.msi)

* run winfsp-1.6.20027.msi and make sure the "Developer" option is checked.

Chocolatey: `choco install -y winfsp`

### [6] install Microsoft redistributables for VS2015, VS2013, VS2010

### [7] Register VS2017 once (initially it has 30 days limit)
* Run VS2017, select your preferred environment (for example C/C++), go to Help - About Microsoft Visual Studio - License status - Check for an updated license - log in with any Microsoft account - the license will be updated

## (Obsolete) Building Windows 8 (Windows 2012) drivers and up

* Download and install Visual Studio 2015 (VS15) 
and also download and install Windows 8.1 WDK http://msdn.microsoft.com/en-US/windows/hardware/gg454513 and Windows 10 WDK http://go.microsoft.com/fwlink/p/?LinkId=526733. 

* Download and install Windows 10 SDK - https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk

* Download and install CPDK (Cryptographic Provider Development Kit) v8.0 available from Microsoft:
https://www.microsoft.com/en-us/download/details.aspx?id=30688

    [Note: After CPDK installation you need to copy the CPDK dir
    to C:\Program Files (x86)\Windows Kits\8.1 from C:\Program Files (x86)\Windows Kits\8.0
    Because by default VS12 $WindowsSdkDir points to C:\Program Files (x86)\Windows Kits\8.1]

* Download and install Windows Assessment and Deployment Kit (Windows ADK) for Windows 8.1 Update
https://www.microsoft.com/en-us/download/details.aspx?id=39982

## (Obsolete) Building Windows XP, Windows 2003, Windows Vista, Windows 2008, Windows 7 and Windows 2008R2 drivers

* Download and install old DDK 7600.16385.1 (Windows 7 DDK) 
http://www.microsoft.com/en-us/download/details.aspx?id=11800

# How to build?

* Visual Studio build: load and build .sln either for each driver separately or virtio-win.sln in the root directory.
* Command line build: run buildAll.bat either for each driver separately or buildAll.bat in the root directory. Do not try to run from special cmd with VS/DDK environment. buildAll.bat by default builds the driver(s) for all targets and all architectures. It accepts command line options for more selective builds, see Tools/build.bat for more information. Example: "buildAll.bat Win8 64" will build only Win8 amd64 driver.
* The results of builds will be in "&lt;driver&gt;\Install" directory

# Known hints / constraints

* Paths hard-coded in *.bat or *.vcxproj

        hard-coded path to VS14:    "C:\Program Files (x86)\Microsoft Visual Studio 14.0"
        hard-coded path to DDK 7.1: "C:\winddk\7600.16385.1" (can be overriden, see below)

* DDK detection:

        <!-- Define the legacy DDK directory required for downlevel targets -->
        <DDKINSTALLROOT Condition="'$(DDKINSTALLROOT)' == ''">C:\WINDDK\</DDKINSTALLROOT>
        <DDKVER Condition="'$(DDKVER)' == ''">7600.16385.1</DDKVER>
        <LegacyDDKDir>$(DDKINSTALLROOT)$(DDKVER)</LegacyDDKDir>

* env version variables:

        <!-- Second component of driver version -->
        <_RHEL_RELEASE_VERSION_ Condition="'$(_RHEL_RELEASE_VERSION_)' == ''">6</_RHEL_RELEASE_VERSION_>
        <!-- Third component of driver version -->
        <_BUILD_MAJOR_VERSION_ Condition="'$(_BUILD_MAJOR_VERSION_)' == ''">101</_BUILD_MAJOR_VERSION_>
        <!-- Fourth component of driver version -->
        <_BUILD_MINOR_VERSION_ Condition="'$(_BUILD_MINOR_VERSION_)' == ''">58000</_BUILD_MINOR_VERSION_>
        
        STAMPINF_DATE

* SDV/DVL:

        sdv/dvl do not work, but is enabled by default
        SET _BUILD_DISABLE_SDV=Yes to disable

* misc

        pciserial and fwcfg
                only inf/cat files, no sources, no comments