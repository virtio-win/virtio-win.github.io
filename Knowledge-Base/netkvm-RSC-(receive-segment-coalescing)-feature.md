# NetKVM: Receive Segment Coalescing (RSC)

## Overview

RSC is a hardware-based receive acceleration mechanism, reducing CPU load by offloading the task of combining multiple fragments of a large payload into a single packet which is then delivered to the OS network stack. When network transfers utilize large packets, this can provide a significant boost in throughput. One benchmark showed a 5x increase (from 7 Gbps to 35 Gbps) in host-to-guest data throughput.

RSC is supported by the network stack starting from Windows Server 2012 and Windows 8.

See https://technet.microsoft.com/en-us/library/hh997024(v=ws.11).aspx for further details.

## RSC for virtualized workloads

RSC is especially effective in virtualized workloads in these circumstances:

* Host-to-guest data transfer or guest-to-guest data transfer inside the same host. In this setup the data comes to guest system via its TAP device. If the transmitting network adapter is able to offload large data segments, those segments can be sent to the guest without fragmentation. This requires the receiving TAP to offload its TX checksumming and TX Large Segment Offload responsibilities.
* Data originating from an external network to the guest. In this case there are two network components responsible for segment coalescing: the receiving physical NIC and the TAP of the guest's network adapter. The first pass of coalescing segments is performed by the NIC (based on its own coalescing setting). Delivery of the segments to the guest is via the TAP and subject to the same considerations as the host-to-guest scenario above.

## Requirements

RSC can be enabled when all of the following are true:

1. On the host:
   1. The TAP device has the TX checksum offload enabled
   2. TCP Segmentation Offload is enabled
2. On the guest:
   1. The virtio driver is installed
   2. RSC is operational within the guest network stack.

## Host verification
The TAP device must have TX checksum offload and TCP Segmentation Offload (TSO) enabled.
It can be verified by running

```bash
ethtool -k <TAP device name>
```

The TAP device name is the QEMU command-line parameter `ifname=<TAP device name>` in `-netdev`.
Verify that tx-checksumming and tcp-segmentation-offload are set to on.

A working host output would be:
```text
tx-checksumming: on
  tx-checksum-ipv4: off [fixed]
  tx-checksum-ip-generic: on
  tx-checksum-ipv6: off [fixed]
  tx-checksum-fcoe-crc: off [fixed]
  tx-checksum-sctp: off [fixed]
tcp-segmentation-offload: on
  tx-tcp-segmentation: on
  tx-tcp-ecn-segmentation: off [requested on]
  tx-tcp6-segmentation: on
```

Further performance benefit can be gained by performing NIC coalescing:

```bash
ethtool -c <nic name>
```

Typical output:

```text
rx-usecs: 3
```

This parameter configures the maximum allowed time between packets for the NIC to combine in hardware.

**Note:** The unavailability of these features on the host may not indicate a misconfiguration on the host. The guest needs to request this functionality too.

## Guest verification
Receive Segment Coalescing (RSC) must be operational within the guest's network stack. You can verify this in PowerShell by running:


```powershell
Get-NetAdapterRsc | Format-List
```

This command will show the RSC status for your network adapters.
A working guest output would be:

```text
Name                 : Ethernet
InterfaceDescription : Red Hat VirtIO Ethernet Adapter
IPv4Enabled          : True
IPv6Enabled          : True
IPv4Supported        : True
IPv6Supported        : True
IPv4OperationalState : True
IPv6OperationalState : True
IPv4FailureReason    : NoFailure
IPv6FailureReason    : NoFailure
```

If Get-NetAdapterRsc shows IPv4/IPv6Supported=False and FailureReason=Capability, host/libvirt offloads are likely not enabled. Enable offloads via libvirt XML example as shown below.

### libvirt XML example
```xml
<driver name="vhost" txmode="iothread" ioeventfd="on" event_idx="on" queues="4" rx_queue_size="1024" tx_queue_size="1024">
    <host csum="on" gso="on" tso4="on" tso6="on" ecn="on" ufo="on" mrg_rxbuf="on"/>
    <guest csum="on" tso4="on" tso6="on" ecn="on" ufo="on"/>
</driver>
```

## Troubleshooting

Common causes and next steps:

* Capability: The host has not been configured for RSC.
* WFPCompatibility: The Windows Filtering Platform/firewall is preventing RSC. Temporarily disable the firewall to validate.

### Advanced tuning (environment-specific)

- Windows Defender/WFP: On some Windows Server 2025 setups, uninstalling the Windows Defender feature (not just disabling the firewall) was required for RSC to operate. Evaluate security/policy impact before changing.
- Optional registry tweak: NetworkThrottlingIndex
  ```reg
  HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\NetworkThrottlingIndex = 0xffffffff (DWORD)
  ```
  Requires reboot.

See [issue #1026](https://github.com/virtio-win/kvm-guest-drivers-windows/issues/1026) for more details.

## HCK RSC tests

HCK "RSC tests" is currently not part of any playlist.
For engineering purposes in order to pass this specific test and **only for this specific test** use a special addition to virtio-net-pci command line as follows: "**guest_rsc_ext=on,rsc_interval=1000000**". Note that this device parameter is not compatible with any other test as well as with regular netkvm functionality.
