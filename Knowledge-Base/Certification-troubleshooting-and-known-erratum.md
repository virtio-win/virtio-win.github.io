# General

## Windows Server 2025 / Windows 11 24H2 / Windows 11 25H2

### Display capture tests* GPU tests (3 on Client)

#### Error
```
Cannot generate display to capture mappings without a display engine, capture card, and toolbox. 
Unable to determine any display output to display capture device mappings - tests cannot continue.
No board detected!
```

#### Solution 

Errata/Filter#166032 for `boolean(//Device[starts-with(@Name,"VMware")] | //Device[starts-with(@Name,"Red Hat VirtIO")])`

### Display capture tests - Display Mode Change Performance (viogpudo driver)

#### Error
```
AreNotEqual(targetsToTest.size(), (size_t)0): Failed to find any targets to test, please check the system configuration and connected monitors - Values (0, 0)
```

#### Solution

Errata/Filter: #166032 (version 9)

**Note:** Filter #166032 covers both Windows 11 24H2 and Windows 11 25H2.

### NdisStudio* network tests (5 on Server, 3 on Client)

#### Error
```
Desktop: Failed to install NdisStudio.inf (Error: 0x800B0101 - A required certificate is not within its validity period when verifying against the current system clock or the timestamp in the signed file.)
An unexpected error occurred while validating driver package. Catalog = NdisStudio.cat, Error = 0x800B0101
```

#### Solution

Request errata from Microsoft or wait for a new HLK kit with a solution

### NDISTest 6.5 - [2 Machine] - GlitchFree Device on Win2025 / Win11 24H2

#### Error 

```
“Analyzer detected XXX errors with packets per DPC = XXX”
The test has 3 iterations (packets per DPC = 10,100,1000), each iteration fail with the same error, only number of errors during the iteration may differ
```

#### Solution 

```
Errata/filter#175565 to include 'boolean(//Devnode[
or contains(.,''PCI\VEN_1AF4&DEV_1000'')
or contains(.,''PCI\VEN_1AF4&DEV_1041'')
{"Field":"KitVersion","MatchType":2,"Values":["10.1.26100"]}'
```

### NDISTest 6.0 - [2 Machine] - 2c_Mini6Stress test

#### Solution

Errata/Filter: #235476

### NDISTest 6.5 - [2 Machine] - CheckConnectivity with AutoFail

#### Solution

Manual errata #239888 (Expiration Date: 8/13/2026) for Red Hat Company, Red Hat VirtIO NetKVM Drivers for Windows Server 2025 and Red Hat VirtIO NetKVM Drivers for Windows 11 model

Request errata from Microsoft or wait for a new HLK kit with a solution

### DXGI Gamma Ramps (viogpu driver)

#### Solution

Errata/filters: 
 - #5226 (version 8) 
 - #5229 (version 9). 
  
 **Note:** Versions are mandatory because it does not have any HLK kit or Windows version limitation.

## Windows Server 2022

### **USB Generic HID Test** error "No MUTT devices were detected"

#### Error
`PM No MUTT devices were detected `

#### Solution
Unfortunately MS didn't grant errata for this test and the redirection MUTT device to test VM is needed in order to pass the test.

In order to pass the test use the following instructions when using Super MUTT V3.0 device connected to the host USB3 port:
1. HCK SUT QEMU should have a HMP monitor on local telnet port 10005 and "-device usb-xhci"
2. Run the script (see below)
3. Schedule the test
4. Stop the script after the test passed

Redirection script
```
#! /bin/bash

hmon_port=10005
muttbus=0
muttaddr=0

renew_device() {
  { echo "device_del m1" ; sleep 1 ; echo "device_add usb-host,id=m1,hostbus=$muttbus,hostaddr=$muttaddr"; sleep 1; } | telnet localhost $hmon_port
}

for (( ; ; ))
do
lsusb | grep 045e | sed 's/Bus //g;s/Device //g;s/://' > lsusb.txt
cat lsusb.txt
read _muttbus _muttaddr xx <<< $(cat lsusb.txt)
bus=$((10#$_muttbus))
addr=$((10#$_muttaddr))
echo $bus $addr
if [ "$bus" != "$muttbus" ] || [ "$addr" != "$muttaddr" ] ; then
  muttbus=$bus
  muttaddr=$addr
  renew_device
fi
sleep 1
done
```

### **USB Type-C ACPI Validation** error

#### Error
`AreEqual(g_numberOfTypeCPorts, expectedNumberOfTypeCPorts): Compare number of marked Type-C ports (_UPC in ACPI) to expected number of Type-C ports. Please make sure the parameter 'NumberOfUsbTypeCPorts' has been set correctly. If this is a USB Dual-Role system that uses the URS driver, connect a USB device to each Dual-Role capable connector and re-run the test. - Values (0, 3)`

#### Test documentation:
https://docs.microsoft.com/en-us/windows-hardware/test/hlk/testref/b3c41a3f-b844-4c2d-b115-dad51a37f123

#### Solution
Set test job parameter _NumberOfUsbTypeCPorts_ to 0. 

### RSC test

For Server 2022, please use this playlist - "HLK Version 21H2 CompatPlaylist x64 ARM64 server". 

#### MS recommendation
https://techcommunity.microsoft.com/t5/windows-hardware-certification/windows-11-amp-server-2022-hlk-kit-guidance-for-creating-new/ba-p/2567481
