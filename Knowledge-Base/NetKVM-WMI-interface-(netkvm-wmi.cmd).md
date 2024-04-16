**[netkvm-wmi.cmd](https://github.com/virtio-win/kvm-guest-drivers-windows/blob/master/NetKVM/DebugTools/WMI/netkvm-wmi.cmd)** is a simple developer-oriented tool that allows access to NetKVM driver to query or control the behavior of the driver.

It is based on WMI interface of the network driver: WMI commands are converted to custom OID that the OS sends to the NetKVM driver.

In case of multiple NetKVM devices (multiple network adapters) each command of the netkvm-wmi.cmd is executed for all the devices.

**The batch file requires administrative permissions, run it from administrator command line.**

Examples of most useful commands are:
* query initial configuration

`` netkvm-wmi.cmd cfg`` 
* query all run-time statistics:

`` netkvm-wmi.cmd stat`` 
* query run-time RSS statistics

`` netkvm-wmi.cmd rss``
* query run-time TX statistics

`` netkvm-wmi.cmd tx``
* query run-time RX statistics

`` netkvm-wmi.cmd rx``
* reset only TX | RX | RSS statistics

`` netkvm-wmi.cmd reset tx``

`` netkvm-wmi.cmd reset rx``

`` netkvm-wmi.cmd reset rss``

* reset all the statistics

`` netkvm-wmi.cmd reset``

* set logging level of the driver to 3 at runtime:

`` netkvm-wmi.cmd debug 3``

For the list of available commands run **[netkvm-wmi.cmd](https://github.com/virtio-win/kvm-guest-drivers-windows/blob/master/NetKVM/DebugTools/WMI/netkvm-wmi.cmd)** without parameters.

To add new commands for development/testing purposes see the implementation of existing WMI commands:
* https://github.com/virtio-win/kvm-guest-drivers-windows/blob/master/NetKVM/Common/netkvm.mof
* vendor-specific commands in https://github.com/virtio-win/kvm-guest-drivers-windows/blob/master/NetKVM/wlh/ParaNdis6_Oid.cpp
