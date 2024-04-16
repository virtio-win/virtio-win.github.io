For any kind of build (EWDK or Visual Studio) there are environment variables affecting the result of the build

### Exclude legacy operating systems

By default the build process create drivers for all the operating systems:
* Legacy ones: XP, 2003, Vista, 2008, 2008R2, Win 7
* Active ones: Win 8 (same as 2012), Win 8.1 (same as 2012R2), Win 10 (same as 2016 and 2019)

Build for legacy operating systems does not make too much sense today. In order to exclude them from the build, use the environment variable

**VIRTIO_WIN_NO_LEGACY=1**

### Generate INF files compatible with DIFx framework
By default the build generate INF files with exact OS ornamentation, i.e drivers for Win10 contain suffix ".10.0".

For example "%VENDOR% = NetKVM, NTamd64.10.0"

This completely correct ornamentation allows operating system to find proper driver when only top-level directory is selected in manual installation. However if automatic driver installation procedure uses DIFx (like WIX does) to install specific drivers for the current OS, the framework fails to install selected driver (DIFx supports only drivers without OS ornamentation or with OS specifier up to 6.3). In order to generate INF files compatible with DIFx, use the environment variable

**WIN10_INF_DIFX_COMPAT=1**
