# IPSec Overview
This will serve as a general overview to IP Security, or `IPSec` functionality, as well as parts that are specific to Arista hardware, some different topology configurations, and finally, some troubleshooting commands specific to Arista platforms.

In general, IPSec VPNs are typically referred to as "tunnels", most often in an effort to logically view them as a method of passing traffic securely across an insecure network. In reality, IPSec VPNs are nothing more than packets that are encapsulated and encrypted, either by the endpoint, or by a device in the path, and then routed across the insecure network.  As a result, there are various pieces of this security mechanism that enable it to work.
## Components and Protocols
### Phases
In the formation of an IPSec VPN, there are two phases the endpoints go through before being able to encrypt and transmit traffic.  These are subsequently called `Phase 1` and `Phase 2`.
#### Phase 1
The first phase of this process is used to authenticate the identities of the endpoints participating in the VPN.  Additionally, once this first phase is complete, it allows for the second phase of the process to take place in an encrypted manner.  The phase 1 tunnel protects the control-plane traffic between the endpoints.

The following protcols could be in use for phase 1:

| Protocol | Name | Reference |
| ---- | ---- | ---- |
| ISAKMP | Internet Security Association and Key Management Protocol | https://datatracker.ietf.org/doc/html/rfc2408 |
| IKEv1 | Internet Key Exchange version 1 | https://datatracker.ietf.org/doc/html/rfc2409 |
| IKEv2 | Internet Key Exchange version 2 | https://datatracker.ietf.org/doc/html/rfc7296 |

The following are the network related details for the IKE Protocol:

| Network Protocol | Network Port | Notes |
| ---- | ---- | ---- |
| UDP | 500 | Used for establishment of VPN |
| UDP | 4500 | `NAT-Traversal` Used for esatablishment of VPN when one endpoint is behind a NAT device | 

&nbsp  

##### IKEv1 Modes and Stages
IKEv1 modes and stages are covered with existing in-depth documentation, so they won't be covered here because all IPSec VPN deployments should really be using IKEv2 at this point in time.
##### IKEv2 Stages
1. IKE_SA_INIT
- establish a secure channel between peers
- combines all info from IKEv1 MM1-4

2. IKE_AUTH
- authenticate remote peer
- includes SA and traffic selector/proxy-id to create Phase2 SA

After the initial IKEv2 stages, Child SAs are created for each proxy-id pair in the VPN tunnel.

#### Phase 2
The second phase of this process is used to authenticate and encrypt the data plane traffic to be sent between the VPN endpoints.  During this process there are two possible protocols used, `ESP` or `AH`.  Additionally, there is another security mechanism called `PFS` or `Perfect Forward Secrecy`, used to increase the security of the encryption keys used.  Once the phase 2 negotiation is complete, an `SA` or `Security Association` is created, which specifies the algorithms in use and which endpoints are part of that security association.  

| Protocol Name | Protocol Number | Notes |
| ---- | ---- | ---- |
| ESP (Encapsulating Security Payload) | 50 | Provides data integrity with authentication and confidentiality with encryption |
| AH (Authentication Header) | 51 | Provides only data integrity through authentication |

`NOTE: The ESP header adds an extra 50 to 57 bytes to the original packet.`

##### Security Association
The phase 2 security association contains various pieces of information of the complete VPN tunnel.  This includes the encryption and authentication algorithms in use, DH group, if PFS is enabled, and also importantly, the proxy-id pair for the security association.  The proxy-id pair is the source and destination prefix that are part of that specific security association, which plays an important part in mixing VPN types.

## IPSec VPN Types
This guide specifically references site to site IPSec VPNs, and therefore, there are two different types of IPSec VPNs that are deployed by the various OEMs.  They are `Policy Based` and `Route Based` (Used on Arista Devices) VPNs. 
### Policy Based VPNs
Policy based VPNs are named as such because they use a policy approach to send the matching traffic to the crypto engine for processing.  This typically involves invoking an ACL that is called in the VPN configuration, which has a source and destination prefix or set of prefixes that must match.  It's important to know, that even in a single, extended, ACL, which has multiple sequences, a security association with matching proxy-id pair will be created for each sequence.  Additionally, policy based VPNs typically do not use a dedicated tunnel interface, they typically source via an existing interface.

### Route Based VPNs
Route based VPNs, which are the type supported by Arista devices, are named as such because they use routes and the RIB to direct traffic to the crypto engine for processing.  Route based VPNs utilize dedicated tunnel interfaces, and importantly, the proxy-ids area a default prefix for source and destination of 0.0.0.0.  Having a 0.0.0.0/0.0.0.0 proxy-id pair ensures all traffic directed to the tunnel interface is encrypted into the VPN.  Another important function with route based VPNs that takes place, is that when the tunnel interface is enabled and up, and phase 2 completes, a route is automatically added to the routing table for the remote end of the tunnel through that endpoint.  One major benefit of route based VPNs is the ability to run a dynamic routing protocol over it, such as OSPF or BGP.

## IPSec VPN Modes
In addition to there being two different types of IPSec VPNs, there are also two different VPN modes, transport mode and tunnel mode.  Arista devices only support tunnel mode
### Transport Mode
Transport mode VPNs are typically VPNs built between two endpoints, or actual clients.  In this instance the IP header of the packet is original, and only the payload data is encrypted.  Because there is no new IP header, the configuration and routing of traffic is less complex. 
### Tunnel Mode
Tunnel mode VPNs are the usual VPNs built between VPN gateways, and as such, internal information like routing and prefixes are protected because the entire original packet is encrypted and the new IP header is added on top of it.  The only information that can be gleaned from intercepting any tunnel mode IPSec packets are the tunnel endpoint IPs.

# Arista Hardware Support
Two model Arista switches support IPSec, and more importantly, support the crypto processing in hardware.  The models supporting this are the 7020TR and 7020SR, and the 7280 with the `M` designation in the model number, like 7280CR3MK.
## Limitations by model
While IPSec VPN tunnels are supported on the two Arista platforms mentioned, there are some limitation and caveats for each model, the important ones outlined below.

### Arista 7020 Limitations
The platform specific limitations can be found here:   https://www.arista.com/en/support/toi/eos-4-22-0f/14302-ipsec-tunnels-on-eos

Some limitations and caveats of note:
- A TCAM profile must be defined in the CLI to enable the IPSec feature, and the profile must be made the active system profile.
- Only two combinations of authentication/encryption are supported:  SHA1/AES256 (CBC), SHA256/AES256 (CBC).
- ESP Fragmentation is not supported, tunnel interface MTU must be lowered to account for ESP packet overhead.

### Arista 7280CR3MK Limitations
The platform specific limitations can be found here:  https://www.arista.com/en/support/toi/eos-4-27-1f/14888-ipsec-tunnel-mode-support-on-7280cr3mk

Some limitations and caveats of note:
- The underlay interface used for resolving the tunnel endpoint must be a L3 sub-interface.
- NAT-T is not supported
- IPSec and MACSEC are mutually exclusive in the configuration, and only one can operate at a time.
## VRF Support
For both platforms, there is VRF support, and it is as follows:

1. Only the default VRF is supported for the underlay interface.
2. The tunnel interface can be in a different VRF.

## Example Topologies and Configurations
The following example topologies and configurations have been deployed and tested on actual hardware consisting of 3x Arista DCS-7280CR3MK-32P4S-F as the VPN endpoints, and various other non-crypto 7280 and 720XP access switches.  These topologies also use BGP peering across the VPN tunnel/s.

The configurations that are included are just for the relevant devices running the IPSec tunnel interfaces, and relative to the IPSec and routing operation.  Base and global config parts are left out.
### IPSec VPN with eBGP Routing
This topology is a simple site to site IPSec VPN between two Arista 7280CR3MKs.  With a single, directly connected link for the underlay transport between the switches and eBGP running across the tunnel interface to advertise the local subnets.

<img src="images/IPSec VPN-eBGP - Logical.png">
&nbsp  

#### Configurations
<details><summary>EOS14</summary><p>

```
ip security
   ike policy ike-pol
      encryption aes256
   
   sa policy sa-pol
      sa lifetime 2 hours
      pfs dh-group 14
   
   profile vpn
      ike-policy ike-pol 
      sa-policy sa-pol 
      connection start
      shared-key 7 0005010F174F0A
      dpd 30 15 clear

vlan 10
   name vlan10

vlan 30
   name vlan30      

interface Ethernet31/1
   no switchport

interface Ethernet31/1.100
   encapsulation dot1q vlan 100
   ip address 192.1.1.1/30

interface Tunnel0
   mtu 1394
   ip address 10.255.255.253/30
   tunnel mode ipsec
   tunnel source 192.1.1.1
   tunnel destination 192.1.1.2
   tunnel ipsec profile vpn

interface Vlan10
   ip address 10.10.10.1/24

interface Vlan30
   ip address 30.30.30.1/24

router bgp 65100
   router-id 10.255.255.253
   neighbor 10.255.255.254 remote-as 65200
   neighbor 10.255.255.254 maximum-routes 0
   network 10.10.10.0/24
   network 30.30.30.0/24
```

</p></details>
&nbsp  
<details><summary>EOS15</summary><p>

```
ip security
   ike policy ike-pol
      encryption aes256
   
   sa policy sa-pol
      sa lifetime 2 hours
      pfs dh-group 14
   
   profile vpn
      ike-policy ike-pol 
      sa-policy sa-pol 
      shared-key 7 03054902151B20
      dpd 30 15 clear

vlan 20
   name vlan20

vlan 40
   name vlan40

interface Ethernet31/1
   no switchport
!
interface Ethernet31/1.100
   encapsulation dot1q vlan 100
   ip address 192.1.1.2/30

interface Tunnel0
   mtu 1394
   ip address 10.255.255.254/30
   tunnel mode ipsec
   tunnel source 192.1.1.2
   tunnel destination 192.1.1.1
   tunnel ipsec profile vpn

interface Vlan20
   ip address 20.20.20.1/24

interface Vlan40
   ip address 40.40.40.1/24

router bgp 65200
   router-id 192.1.1.2
   neighbor 10.255.255.253 remote-as 65100
   neighbor 10.255.255.253 maximum-routes 0
   network 20.20.20.0/24
   network 40.40.40.0/24
```

</p></details>
&nbsp  

### IPSec VPN with vrfs and eBGP Routing Version 1

<img src="images/IPSec VPN-VRF-eBGPv1 - Logical.png">
&nbsp  

#### Configurations
<details><summary>EOS14</summary><p>

```
ip security
   ike policy ph1-pol
      encryption aes256
   
   sa policy ph2-pol
      sa lifetime 2 hours
   
   profile test
   
   profile vpn
      ike-policy ph1-pol 
      sa-policy ph2-pol 
      shared-key 7 1218171E011F0D
      dpd 30 15 clear

vlan 10
   name vlan10

vlan 30
   name vlan30

interface Ethernet31/1
   no switchport

interface Ethernet31/1.101
   encapsulation dot1q vlan 101
   ip address 192.2.2.2/30

interface Tunnel1
   mtu 1394
   ip address 10.255.255.6/30
   tunnel mode ipsec
   tunnel source 192.2.2.2
   tunnel destination 192.2.2.1
   tunnel ipsec profile vpn

interface Vlan10
   ip address 10.10.10.1/24

interface Vlan30
   ip address 30.30.30.1/24

router bgp 65102
   router-id 10.255.255.6
   neighbor 10.255.255.5 remote-as 65101
   network 10.10.10.0/24
   network 30.30.30.0/24
```

</p></details>
&nbsp  
<details><summary>EOS15</summary><p>

```
ip security
   ike policy ph1-pol
      encryption aes256
   
   sa policy ph2-pol
      sa lifetime 2 hours
      pfs dh-group 14
   
   profile vpn
      ike-policy ph1-pol 
      sa-policy ph2-pol 
      connection start
      shared-key 7 0207165218120E
      dpd 30 15 clear

vlan 20
   name vlan20

vlan 30
   name vlan30

vlan 40
   name vlan40

vrf instance warehouse

interface Ethernet30/1
   description transport_to_EOS16
   no switchport

interface Ethernet30/1.100
   encapsulation dot1q vlan 100
   ip address 192.1.1.1/30

interface Ethernet31/1
   description transport_to_EOS14
   no switchport

interface Ethernet31/1.101
   encapsulation dot1q vlan 101
   ip address 192.2.2.1/30

interface Tunnel0
   mtu 1394
   vrf warehouse
   ip address 10.255.255.1/30
   tunnel mode ipsec
   tunnel source 192.1.1.1
   tunnel destination 192.1.1.2
   tunnel ipsec profile vpn

interface Tunnel1
   mtu 1394
   ip address 10.255.255.5/30
   tunnel mode ipsec
   tunnel source 192.2.2.1
   tunnel destination 192.2.2.2
   tunnel ipsec profile vpn

interface Vlan20
   ip address 20.20.20.1/24

interface Vlan30
   vrf warehouse
   ip address 30.30.30.1/24

interface Vlan40
   ip address 40.40.40.1/24

ip routing vrf warehouse

router bgp 65101
   neighbor 10.255.255.6 remote-as 65102
   network 20.20.20.0/24
   network 40.40.40.0/24
   
   vrf warehouse
      neighbor 10.255.255.2 remote-as 65103
      network 30.30.30.0/24
```

</p></details>
&nbsp  
<details><summary>EOS16</summary><p>

```
ip security
   ike policy ph1-pol
      encryption aes256
   
   sa policy ph2-pol
      sa lifetime 2 hours
   
   profile vpn
      ike-policy ph1-pol 
      sa-policy ph2-pol 
      shared-key 7 1304051B181805
      dpd 15 30 clear

vlan 50
   name vlan50

vlan 60
   name vlan60

vlan 70
   name vlan70

interface Ethernet30/1
   no switchport

interface Ethernet30/1.100
   encapsulation dot1q vlan 100
   ip address 192.1.1.2/30

interface Tunnel0
   mtu 1394
   ip address 10.255.255.2/30
   tunnel mode ipsec
   tunnel source 192.1.1.2
   tunnel destination 192.1.1.1
   tunnel ipsec profile vpn

interface Vlan50
   ip address 50.50.50.1/24

interface Vlan60
   ip address 60.60.60.1/24

interface Vlan70
   ip address 70.70.70.1/24

router bgp 65103
   neighbor 10.255.255.1 remote-as 65101
   network 50.50.50.0/24
   network 60.60.60.0/24
   network 70.70.70.0/24
```

</p></details>
&nbsp  

### IPSec VPN with vrfs and eBGP Routing Version 2

<img src="images/IPSec VPN-VRF-eBGPv2 - Logical.png">
&nbsp  

#### Configurations
<details><summary>EOS14</summary><p>

```
ip security
   ike policy ph1-pol
      encryption aes256
   
   sa policy ph2-pol
      sa lifetime 2 hours

   profile vpn
      ike-policy ph1-pol 
      sa-policy ph2-pol 
      shared-key 7 1218171E011F0D
      dpd 30 15 clear

vlan 10
   name vlan10

vlan 30
   name vlan30

interface Ethernet31/1
   no switchport

interface Ethernet31/1.101
   encapsulation dot1q vlan 101
   ip address 192.2.2.2/30

interface Tunnel1
   mtu 1394
   ip address 10.255.255.6/30
   tunnel mode ipsec
   tunnel source 192.2.2.2
   tunnel destination 192.2.2.1
   tunnel ipsec profile vpn

interface Vlan10
   ip address 10.10.10.1/24

interface Vlan30
   ip address 30.30.30.1/24

router bgp 65102
   router-id 10.255.255.6
   neighbor 10.255.255.5 remote-as 65101
   network 10.10.10.0/24
   network 30.30.30.0/24
```

</p></details>
&nbsp  
<details><summary>EOS15</summary><p>

```
ip security
   ike policy ph1-pol
      encryption aes256
   !
   sa policy ph2-pol
      sa lifetime 2 hours
      pfs dh-group 14
   !
   profile vpn
      ike-policy ph1-pol 
      sa-policy ph2-pol 
      connection start
      shared-key 7 0207165218120E
      dpd 30 15 clear
    
vlan 20
   name vlan20

vlan 30
   name vlan30

vlan 40
   name vlan40

vrf instance vpn

interface Ethernet30/1
   description transport_to_EOS16
   no switchport

interface Ethernet30/1.100
   encapsulation dot1q vlan 100
   ip address 192.1.1.1/30

interface Ethernet31/1
   description transport_to_EOS14
   no switchport

interface Ethernet31/1.101
   encapsulation dot1q vlan 101
   ip address 192.2.2.1/30

interface Tunnel0
   mtu 1394
   vrf vpn
   ip address 10.255.255.1/30
   tunnel mode ipsec
   tunnel source 192.1.1.1
   tunnel destination 192.1.1.2
   tunnel ipsec profile vpn

interface Tunnel1
   mtu 1394
   vrf vpn
   ip address 10.255.255.5/30
   tunnel mode ipsec
   tunnel source 192.2.2.1
   tunnel destination 192.2.2.2
   tunnel ipsec profile vpn

interface Vlan20
   vrf vpn
   ip address 20.20.20.1/24

interface Vlan30
   ip address 30.30.30.1/24

interface Vlan40
   vrf vpn
   ip address 40.40.40.1/24

ip routing vrf vpn

router bgp 65101
   vrf vpn
      neighbor 10.255.255.2 remote-as 65103
      neighbor 10.255.255.6 remote-as 65102
      network 20.20.20.0/24
      network 40.40.40.0/24
```

</p></details>
&nbsp  
<details><summary>EOS16</summary><p>

```
ip security
   ike policy ph1-pol
      encryption aes256
   
   sa policy ph2-pol
      sa lifetime 2 hours
   
   profile vpn
      ike-policy ph1-pol 
      sa-policy ph2-pol 
      shared-key 7 1304051B181805
      dpd 15 30 clear

vlan 50
   name vlan50

vlan 60
   name vlan60

vlan 70
   name vlan70

interface Ethernet30/1
   no switchport

interface Ethernet30/1.100
   encapsulation dot1q vlan 100
   ip address 192.1.1.2/30

interface Tunnel0
   mtu 1394
   ip address 10.255.255.2/30
   tunnel mode ipsec
   tunnel source 192.1.1.2
   tunnel destination 192.1.1.1
   tunnel ipsec profile vpn

interface Vlan50
   ip address 50.50.50.1/24

interface Vlan60
   ip address 60.60.60.1/24

interface Vlan70
   ip address 70.70.70.1/24

router bgp 65103
   neighbor 10.255.255.1 remote-as 65101
   network 50.50.50.0/24
   network 60.60.60.0/24
   network 70.70.70.0/24
```

</p></details>
&nbsp  

# Troubleshooting
The following commands can be used to troubleshoot VPN tunnels:

### show interface tunnel `x`
<img src="images/sho int tun - down.png">

<img src="images/sho int tun.png" width="334" height="184">


### show ip route
<img src="images/sho ip route.png">

Test

### show ip security connection
<img src="images/sho ip sec conn.png">

### show ip security connection tunnel `x` detail
<img src="images/sho ip sec conn tun1 det.png" width="1000" height="730">


### show ip security policy
<img src="images/sho ip sec policy.png">

### show ip security profile
<img src="images/sho ip sec profile.png">

### show ip security security-association
<img src="images/sho ip sec security-association.png">

### show hardware tcam profile 


### show license
<img src="images/sho license.png" width="350" height="375">

### show kernel ipsec
<img src="images/sho kernel ipsec.png">

### show ip security daemon
<img src="images/sho ip sec daemon - pre.png">
<img src="images/sho ip sec daemon - post.png">

### show platform fap ipsec interface tunnel `x`



