Below are instructions for setting up EWDK 1809 for building using only MSBuild in cmd prompt, Windows 8.1 and Windows 10 only.

- @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
- refreshenv
- choco install -y windows-sdk-10-version-1809-all wget winfsp curl
- wget https://download.microsoft.com/download/9/F/9/9F9F709B-C0E6-4B89-90C2-CDE3059C61CF/EWDK_rs5_release_svc_prod2_17763_190129-1747.iso
- powershell mount-diskimage -imagepath "C:\EWDK_rs5_release_svc_prod2_17763_190129-1747.iso"
- powershell "$driveletter = (get-diskimage C:\EWDK_rs5_release_svc_prod2_17763_190129-1747.iso | get-volume).driveletter" ; robocopy $driveletter`:\ c`:\ /E"
- set MSBuildEmitSolution=1
- start LaunchBuildEnv.cmd

Note1: Verified building succeeds with Win8, Win8.1, Win10 x86 and x64, all Release, so far. Sample syntax:

`msbuild .\virtio-win.sln -p:Configuration=Win8.1%20Release -p:Platform="x64"`

Note2: For pre-Win8 building only UI-based Windows works, due to Win7 DDK dependencies.