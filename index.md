## Welcome to GitHub Pages

🥶 🥶

??? attention
    The `service routing protocols model multi-agent` command requires a reboot to take effect.

##### Interfaces
```yaml

interface Ethernet1
   mtu 9214 #(1)!
   no switchport
   ip address 192.168.0.0/31
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 192.168.0.2/31
!
interface Ethernet3
   mtu 9214
   no switchport
   ip address 192.168.0.4/31
!
interface Ethernet4
   mtu 9214
   no switchport
   ip address 192.168.0.6/31
!
interface Ethernet5
   mtu 9214
   no switchport
   ip address 192.168.0.8/31
!
interface Ethernet6
   mtu 9214
   no switchport
   ip address 192.168.0.10/31
!
interface Ethernet7
   mtu 9214
   no switchport
   ip address 192.168.0.12/31
!
interface Ethernet8
   mtu 9214
   no switchport
   ip address 192.168.0.14/31
!
interface Loopback0
   description Globally Unique Address
   ip address 10.0.0.111/32
```
