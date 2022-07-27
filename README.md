# IPSec Overview
This will serve as a general overview to IP Security, or `IPSec` functionality, as well as parts that are specific to Arista hardware, some different topology configurations, and finally, some troubleshooting commands specific to Arista platforms.

In general, IPSec VPNs are typically referred to as "tunnels", most often in an effort to logically view them as a method of passing traffic securely across an insecure network. In reality, IPSec VPNs are nothing more than packets that are encapsulated and encrypted, either by the endpoint, or by a device in the path, and then routed across the insecure network.  As a result, there are various pieces of this security mechanism that enable it to work.
## Components and Protocols
### Phases
In the formation of an IPSec VPN, there are two phases the endpoints go through before being able to encrypt and transmit traffic.  These are subsequently called `Phase 1` and `Phase 2`.
#### Phase 1
The first phase of this process is used to authenticate the identities of the endpoints participating in the VPN.  Additionally, once this first phase is complete, it allows for the second phase of the process to take place in an encrypted manner.  The phase 1 tunnel protects the control-plane traffic between the endpoints.

The following protcols could be in use for phase 1.

| Protocol | Name | Reference |
| ---- | ---- | ---- |
| ISAKMP | Internet Security Association and Key Management Protocol | Visit https://datatracker.ietf.org/doc/html/rfc2408 |
| IKEv1 | Internet Key Exchange version 1 | Visit https://datatracker.ietf.org/doc/html/rfc2409 |
| IKEv2 | Internet Key Exchange version 2 | Visit https://datatracker.ietf.org/doc/html/rfc7296 |

## IPSec VPN Types
# Arista Hardware Support

## Limitations by model
# Configurations

## VRF Support

## Example Topologies and Configurations

# Troubleshooting
