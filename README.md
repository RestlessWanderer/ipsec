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

##### IKEv1 Modes and Stages
IKEv1 modes and stages are covered with in-depth documentation, so I won't cover it here because all IPSec VPN deployments should really be using IKEv2 at this point in time.
##### IKEv2 Stages


#### Phase 2
The second phase of this process is used to authenticate and encrypt the data plane traffic to be sent between the VPN endpoints.  During this process there are two possible protocols used, `ESP` or `AH`.  Additionally, there is another security mechanism called `PFS` or `Perfect Forward Secrecy`, used to increase the security of the encryption keys used.  Once the phase 2 negotiation is complete, an `SA` or `Security Association` is created, which specifies the algorithms in use and which endpoints are part of that security association.  

| Protocol Name | Protocol Number | Notes |
| ESP (Encapsulating Security Payload) | 50 | Provides data integrity with authentication and confidentiality with encryption
| AH (Authentication Header) | 51 | Provides only data integrity through authentication
## IPSec VPN Types
This guide specifically references site to site IPSec VPNs, and therefore, there are two different types of IPSec VPNs that are deployed by the various OEMs.  They are `Policy Based` and `Route Based` (Used on Arista Devices) VPNs. 
### Policy Based VPNs


### Route Based VPNs

## IPSec VPN Modes

### Transport Mode

### Tunnel Mode

# Arista Hardware Support
Two model Arista switches support IPSec, and more importantly, support the crypto processing in hardware.  The models supporting this are the 7020 and the 7280 with the `M` designation in the model number. 
## Limitations by model

## VRF Support

## Example Topologies and Configurations
The following example topologies and configurations have been deployed and tested on actual hardware consisting of 3x Arista DCS-7280CR3MK-32P4S-F as the VPN endpoints, and various other non-crypto 7280 and 720XP access switches.  These topologies also use BGP peering for the underlay transport network, and across the VPN tunnel/s.

The configurations that are included are just for the relevant devices running the IPSec tunnel interfaces, and relative to the IPSec and routing operation.  Base and global config parts are left out.
### IPSec VPN with eBGP Routing
This topology is a simple site to site IPSec VPN between two Arista 7280CR3Mks.  With a single, directly connected link for the underlay transport between the switches, eBGP is only running across the tunnel interface to advertise the local subnets.

<img src="images/IPSec VPN-eBGP - Logical.png">

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

### IPSec VPN with vrfs and eBGP Routing Version 2

<img src="images/IPSec VPN-VRF-eBGPv2 - Logical.png">

# Troubleshooting
