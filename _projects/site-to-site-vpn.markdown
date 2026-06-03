---
layout: page
title: FortiGate Site-to-Site VPN Configuration and Troubleshooting
description: "Configured a site-to-site VPN tunnel between two FortiGate firewalls in a virtual lab, simulating a real-world branch office connection. Diagnosed and resolved common VPN misconfigurations including phase 1 and phase 2 failures."

tech: [FortiGate, VPN, Networking, Firewall]

---
## Objective

The aim of this lab was to design , configure and validate a site-to-site IPSec VPN tunnel between two FortiGate VM instances simulating network sites at different locations. Aside basic configurations, the lab included troubleshooting exercises, where misconfigurations were introduced across both sites and systematically diagnosed and resolved.


## Lab Setup 

| Component | Site A | Site B |
|:-------------|:--------:|--------------:|
| Firewall     |  FortiGate VM v7.4    | FortiGate VM v6.4         |
| Peer IP(WAN) |    192.168.209.132  | 192.168.209.131   |
| LAN Subnet   | 172.16.3.0/24   | 10.81.3.0/24  |
| LAN Gateway    |   172.16.3.1   |  10.81.3.1   |
| Endpoint ( LAN)   |  Ubuntu - 172.16.3.3  | Ubuntu - 10.81.3.6  |
| Platform  |   VMware   | GNS3 ( VMware-backed) |

The two sites intentionally run different FortiOS versions ( v7.4 and v6.4) to simulate a real-world scenario where branch offices may be on different firmware versions. 

## Network Topology

<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/net-config.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:550px; height:350px; object-fit:contain; max-width:100%;">
</div>
 
 The lab topology consisted of two FortiGate VMs connected through a shared transit network (*192.168.209.0/24*) simulating a WAN segment. Ubuntu endpoints were placed behind each FortiGate to simulate LAN hosts and serve as ping test points for tunnel validation. 


## Virtual Private Network (VPN)

A Virtual Private Network (VPN) is a technology that creates a secure, encrypted connection over a public network.The encrypted connection helps ensure that sensitive data is safely transmitted, preventing unauthorised people from eavesdropping on the traffic. 

Site-to-Site VPNs connnect entire networks together and allow users of these networks to access each other's resources.

#### *IPSec VPN Parameters*

**Phase 1**:
- establishes  a secure, authenticated channel between the two VPN devices.
- authenticates VPN peers using pre-shared key
- negotiates a set of encryption and authentication parameters(IKE proposal) and DH group for key exchange

```
Phase 1 — IKE Configuration
─────────────────────────────
IKE Version          : Version 2
Encryption           : DES
Authentication       : SHA1
DH Group             : Group 5
Key Lifetime         : 86400 seconds
Authentication Method: Pre-Shared Key
```

**Phase 2**:
- encrypts and transmits actual data between networks.
<br>

```
Phase 2 — IPSec Configuration
─────────────────────────────
Encryption           : DES
Authentication       : SHA1
Perfect Forward Secrecy (PFS): Off
Auto-negotiate     : On
Key Lifetime         : 86400 seconds
```
<br>


## Configuration Steps

It is good practice to use the Route monitor to verify that the remote IP address is not already in use and to identify the interface associated with the local IP address. 

#### *1. Create Custom IPSec Tunnel*
The tunnel was created via **VPN → VPN Tunnels → Create New → Custom** with the configuration details listed above.


<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/vpn-1.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:500px; height:300px; object-fit:contain; max-width:100%;">
       <p style="font-size:0.9em; color:#666; margin-top:10px;">
    <em> Defining network parameters </em>
     </p>
</div>


<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/vpn-2.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:500px; height:300px; object-fit:contain; max-width:100%;">
       <p style="font-size:0.9em; color:#666; margin-top:10px;">
    <em> Defining authentication and IKE version </em>
     </p>
</div>

<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/vpn-3.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:500px; height:300px; object-fit:contain; max-width:100%;">
       <p style="font-size:0.9em; color:#666; margin-top:10px;">
    <em> Phase 1 Proposal </em>
     </p>
</div>

<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/vpn-4.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:500px; height:300px; object-fit:contain; max-width:100%;">
       <p style="font-size:0.9em; color:#666; margin-top:10px;">
    <em> Setting local and remote addresses for the Phase 2 selectors </em>
     </p>
</div>


<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/vpn-5.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:500px; height:300px; object-fit:contain; max-width:100%;">
       <p style="font-size:0.9em; color:#666; margin-top:10px;">
    <em> Phase 2 parameters  </em>
    </p>
</div>


The process was repeated on Site B mirroring all parameters with the sources and destination subnets reversed. 
<br>

#### *2. Create Address Objects*
Address objects were created on both FortiGate virtual machines to represent  each LAN. Using named address objects rather than raw IP addresses in the firewall policies improves readability and makes it easier to change and manage policies in the future. 

<div style="display:flex; gap:15px;">
  <figure style="width:50%; margin:0;">
    <img src="/assets/projects/VPN/address-A.png"  
         style="width:100%; height: 350px;object-fit:contain;">
    <figcaption style="text-align:center; font-size:0.8rem; color:#666; margin-top:0.4rem;"> Address object (LAN_A) </figcaption>
  </figure>

  <figure style="width:50%; margin:0;">
    <img src="/assets/projects/VPN/address-B.png"  
         style="width:100%; height: 350px; object-fit:contain;">
    <figcaption style="text-align:center; font-size:0.8rem; color:#666; margin-top:0.4rem;"> Address object (LAN_B)</figcaption>
  </figure>
</div>

<br>

#### *3. Set Static Route* 
A Static route was added on each FortiGate to direct traffic bound for remote LAN through the VPN tunnnel interface rather than the default WAN gateway. 

<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/static-route.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:450px; height:300px; object-fit:contain; max-width:100%;">
    
</div>
<br>

#### *4. Create Firewall Policies*

A firewall policy was created on Site A to allow traffic from Site A to Site B across the tunnel. NAT was disabled and security profiles including AntiVirus and IPS were applied. Logging of security events was enabled.

<div style="display:flex; gap:20px;">
  <figure style="width:50%; margin:0;">
    <img src="/assets/projects/VPN/policy-1.png"  
         style="width:100%; height: 400px;object-fit:contain;">

  </figure>

  <figure style="width:50%; margin:0;">
    <img src="/assets/projects/VPN/policy-2.png"  
         style="width:100%; height: 400px; object-fit:contain;">
   
  </figure>
</div>

<p style="font-size:0.9em; color:#666; margin-top:10px; text-align:center;">
    <em> Creating  the firewall Policy </em>
</p>

Another policy was created to allow traffic from Site B to Site A, thus allowing bi-directional flow of traffic between both sites. 

<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/both policies.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:700px; height:200px; object-fit:contain; max-width:100%;">
</div>
This process was repeated on FortiGate Site B.

<br>

## Tunnel Verification

Once both sites were configured, the VPN tunnel was confirmed as <strong> Up </strong> under **VPN → VPN Tunnels**

<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/tunnel-up.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:700px; height:200px; object-fit:contain; max-width:100%;">
</div>

Connectivity was validated  from the Ubuntu endpoint behind Site A by pinging key IP addresses progressively, first the local FortiGate LAN interface, then the WAN interface and lastly the remote site's LAN interface and endpoint.

<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ping-test-1.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:450px; height:300px; object-fit:contain; max-width:100%;">
</div>


<div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ping-test-2.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:450px; height:300px; object-fit:contain; max-width:100%;">
</div>

The same actions were repeated on the Kali Linux endpoint behind Site B.


## Troubleshooting

Following the initial configuration, a deliberate troubleshooting exercise where multiple misconfigurations were introduced on both FortiGate instances. The task was to identify, diagnose, and resolve each issue. 

### Issues Found on FortiGate Site A
**1. Auto-negotiate was disabled on Phase 2**

  Auto-negotiation is essential since it automatically establishes and maintains the secure IPSec tunnel without requiring constant, manual human intervention. If this option is disabled and Phase 2's Security Association (SA) expires, traffic would drop until the SA is manually re-established.

  *Solution*: Auto-negotiate was re-enabled on the Phase 2 selector.
  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-autonegA.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:400px; height:300px; object-fit:contain; max-width:100%;">
  </div> 

<br>

**2. Incorrect Static Route Destination**

  The static route destination had been changed from *10.81.3.0/24* to an incorrect subnet *10.8.13.0/24*, hence traffic destined for Site B's LAN would not be routed through the VPN tunnel interface. This would cause cross-site connectivity failure even though the tunnel is up. 

  *Solution*: Static route destination was corrected.
  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-staticA.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:400px; height:300px; object-fit:contain; max-width:100%;">
  </div> 
<br>

**3. NAT enabled on firewall policy**

  With NAT enabled, packets from LAN_A would have their source IP translated to Site A's WAN IP before entering the tunnel. Since Site B's firewall policy expects source addresses from 172.16.3.0/24, the translated WAN IP would not match and traffic would be dropped. 

  *Solution* : NAT was disabled on the FGT_A to FGT_B policy

  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-policyA.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:700px; height:250px; object-fit:contain; max-width:100%;">
  </div> 

<br>

**4. A duplicate address object with a similar name was created.**

  A new address object ,*LAN__B* , was created and referenced in the firewall policy, meaning destination matching would fail for all traffic from *LAN_A* to the actual *LAN_B* subnet. 

  *Solution*: The duplicate address object was removed and the firewall policy was updated to refer to the correct subnet.

  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-addressA.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:400px; height:300px; object-fit:contain; max-width:100%;">
  </div> 

<br>

### Issues Found on FortiGate Site B
**5. Additional DH Groups and changed key lifetime**

  Additional DH groups were selected in addition to group 5 and the key lifetime was changed from 86400 to 8628 seconds. New encryption and authentication methods were included as well.  These mismatch configurations would cause IKE negotiations to fail since both VPN peers must agree on identicak Phase 1 proposals.

  *Solution* : Reset  Phase 1 proposal to match Site A. 
  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-dhgroupB.png' | relative_url }}" 
      style="width:400px; height:200px; object-fit:contain; max-width:100%;">
  </div> 

<br>

**6. NAT enabled on Site B policy**

  NAT was also enabled on the inbound VPN policy on Site B.

  *Solution*: NAT was disabled on the policy.
  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-NATB.png' | relative_url }}" 
        style="width:700px; height:150px; object-fit:contain; max-width:100%;">
  </div> 
<br>

**7. Incorrect remote IP in Phase 2**

  The remote address in the Phase 2 selectors was set to *17.21.63.0/255.255.255.0* instead of *172.16.3.0/255.255.255.0*

  *Solution* : Remote address was corrected to 172.16.3.0/255.255.255.0

  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-remIP-B.png' | relative_url }}" 
        style="width:400px; height:200px; object-fit:contain; max-width:100%;">
  </div> 
<br>

**8. Change in IKE version**

   The IKE version was changed from Version 2 to Version 1 on Site B. IKE version 1 and version 2 are not directly compatible as a result, Phase 1 negotiation would fail entirely, preventing the tunnel from coming up.

  <div style="text-align:center;">
  <img src="{{ '/assets/projects/VPN/ts-ikeB.png' | relative_url }}" 
      alt="Network Configuration" 
      style="width:400px; height:200px; object-fit:contain; max-width:100%;">
  </div> 
<br>

## Key Takeaway

Phase 1 and Phase 2 parameters must match exactly on both peers. Any mismatch in encryption algorithm, authentication , IKE version or key lifetime will quietly prevent the tunnel from being set up. Also, NAT must be disabled on VPN policies so traffic is not dropped due to mismatched source IP addresses. Auto -negotiate should always be enabled on Phase 2 to automate the renewal of Phase 2 Security Association (SA) when it expires.  FortiGate logs are an essential tool in troubleshooting system misconfigurations.

<br>

##### *This lab was conducted collaboratively with a colleague as part of a knowledge-sharing session at work. I configured Phase 1 and Phase 2 parameters on FGT-Site-A, set the static route and firewall policy and diagnosed and resolved misconfigurations on Site A* 