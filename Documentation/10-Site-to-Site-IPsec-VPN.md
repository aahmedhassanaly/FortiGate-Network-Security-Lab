# Task 10 – Site-to-Site IPsec VPN

## Scenario

The company has two locations:

- HQ Site
- Branch Site

Users at both sites need secure communication over the shared WAN network.

The VPN connects:

- HQ LAN: `10.10.10.0/24`
- Branch LAN: `10.20.10.0/24`

The WAN network is:

- HQ FortiGate: `192.168.1.135`
- Branch FortiGate: `192.168.1.136`

The VPN uses a pre-shared key for authentication.

---

## Objective

Configure and verify a Site-to-Site IPsec VPN between two FortiGate firewalls.

The main goals are:

- Connect HQ and Branch securely.
- Keep the original private IP addresses.
- Allow LAN-to-LAN communication.
- Verify Phase 1 and Phase 2.
- Test communication in both directions.
- Understand IKE version, Phase 1, and Phase 2.

---

## Topology

    WAN / NET
    192.168.1.0/24
           |
    +------+------+
    |             |
    |             |
    FGT-HQ-01     FGT-BRANCH-01
    .135          .136
    port1         port1
       \           /
        \ IPsec VPN
         \       /
          \     /
     10.10.10.0/24
          HQ
           
     10.20.10.0/24
         Branch

---
<img width="1792" height="798" alt="image" src="https://github.com/user-attachments/assets/f50f9783-bb0b-4993-bc76-158f07e5c3c6" />

## VPN Configuration

### HQ FortiGate

Tunnel name:

`HQ-TO-BRANCH`

Remote Gateway:

`192.168.1.136`

Outgoing Interface:

`port1`

Authentication:

`Pre-shared Key`

Local Network:

`10.10.10.0/24`

Remote Network:

`10.20.10.0/24`

NAT:

`No NAT between sites`

Internet Access:

`None`

---

### Branch FortiGate

Tunnel name:

`BRANCH-TO-HQ`

Remote Gateway:

`192.168.1.135`

Outgoing Interface:

`port1`

Authentication:

`Pre-shared Key`

Local Network:

`10.20.10.0/24`

Remote Network:

`10.10.10.0/24`

NAT:

`No NAT between sites`

Internet Access:

`None`

---

## Phase 1 and Phase 2

The VPN wizard automatically created both Phase 1 and Phase 2.

### Phase 1

Phase 1 establishes the secure IKE relationship between the two FortiGate devices.

It handles authentication and negotiation between the VPN peers.

The configured tunnel was created by the FortiGate Site-to-Site VPN wizard.

Verification showed:

`version: 1`

Therefore, the current lab tunnel uses:

`IKEv1`

We did not manually select IKEv2 during the wizard.

### Phase 2

Phase 2 creates the IPsec security association used for the actual protected traffic.

The Branch Phase 2 selectors are:

Local Address:

`BRANCH-TO-HQ_local`

Remote Address:

`BRANCH-TO-HQ_remote`

These represent:

`10.20.10.0/24` → `10.10.10.0/24`

The HQ side uses the opposite direction:

`10.10.10.0/24` → `10.20.10.0/24`

---

## Verification

The tunnel status was verified as UP.

CLI verification on the Branch FortiGate showed:

    diagnose vpn ike gateway list

Important results:

    version: 1
    interface: port1
    addr: 192.168.1.136:500 -> 192.168.1.135:500

    IKE SA: created 2/2 established 2/2
    IPsec SA: created 1/1 established 1/1

This confirms:

- IKE SA is established.
- Phase 1 is established.
- IPsec SA is established.
- Phase 2 is established.
- The VPN peer is reachable.

The negotiated proposal was:

`aes128-sha256`

The VPN also uses DPD to detect peer availability.

---

## Connectivity Test

### Branch to HQ

From `BRANCH-PC-01`:

    ping 10.10.10.100

Result:

`Success`

This confirms that Branch traffic can reach the HQ LAN through the IPsec tunnel.

### HQ to Branch

From the HQ client:

    ping 10.20.10.100

Result:

`Success`

This confirms that the VPN works in both directions.

---

## Important Concept

IKE Version and IPsec Phase are different concepts.

IKE Version:

- IKEv1
- IKEv2

Phase:

- Phase 1
- Phase 2

The current lab uses:

`IKEv1 + Phase 1 + Phase 2`

Phase 2 does not mean IKEv2.

The FortiGate VPN wizard automatically created the Phase 2 configuration for the Site-to-Site VPN.
<img width="1894" height="785" alt="image" src="https://github.com/user-attachments/assets/9d2f3967-cec7-45a7-b9a6-496d4f4a383a" />
<img width="1919" height="849" alt="image" src="https://github.com/user-attachments/assets/6d45a05f-937d-4eb5-b332-f2c4f239b973" />

---

### Verification

After both sides were configured:

- VPN status became UP.
- IKE SA became established.
- IPsec SA became established.
- Branch → HQ ping succeeded.
- HQ → Branch ping succeeded.

---

## Security Considerations

- Use a strong pre-shared key.
- Do not use NAT between the two private LANs.
- Use specific local and remote networks instead of allowing all traffic.
- Keep VPN configuration backed up.
- Monitor VPN status and IPsec events.
- Use strong encryption and integrity algorithms suitable for the environment.
- Restrict firewall policies between VPN networks to only required services in production.

The current lab focuses on connectivity and VPN administration. More restrictive VPN firewall policies can be applied when specific application requirements are defined.

---


## Result

The Site-to-Site IPsec VPN was successfully configured between HQ and Branch.

The final result provides:

- Secure site-to-site connectivity.
- No NAT between the private networks.
- Working Phase 1.
- Working Phase 2.
- Bidirectional LAN communication.
- Verified IKEv1 configuration.
- Verified IPsec Security Association.

The VPN is ready for further troubleshooting and security testing.

---

## Lessons Learned

- IKEv1 and IKEv2 are different IKE protocol versions.
- Phase 1 and Phase 2 are separate parts of the IPsec VPN process.
- Phase 2 is required for the actual protected IPsec traffic.
- The FortiGate VPN wizard can automatically create Phase 1, Phase 2, routes, and firewall policies.
- VPN connectivity must be verified from both sides.
- `diagnose vpn ike gateway list` can provide important information about IKE and IPsec Security Associations.
