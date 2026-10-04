# Task 11 — Remote Access VPN

## Scenario

The company needs secure remote access for users who work outside the HQ network.

A Windows laptop will connect to the FortiGate using FortiClient and an IKEv2 IPsec VPN.

The VPN user must authenticate before accessing the HQ network.

---

## Objective

Configure and verify a remote-access IPsec VPN on FortiGate using:

- IKEv2
- IPsec
- EAP user authentication
- Local FortiGate user
- User group
- VPN client IP pool
- Split tunneling
- Firewall policy

The final goal is to allow an authenticated remote user to access the HQ Users network.

---

## Environment

### FortiGate

- Device: `FGT-HQ-01`
- WAN interface: `port1`
- WAN IP: `192.168.1.135`

### HQ Network

- Users VLAN: `10.10.10.0/24`
- Gateway: `10.10.10.1`

### Remote VPN

- VPN tunnel: `USERS-VPN`
- VPN client pool: `10.10.70.100 - 10.10.70.200`
- IKE version: IKEv2
- Phase 1 proposal: `AES128-SHA256`
- DH Group: `14`

### User Authentication

- User: `vpn-user`
- Group: `REMOTE-VPN-USERS`
- Authentication: EAP
- EAP identity: `send-request`

---

## Configuration

The remote-access VPN was created using the FortiGate IPsec VPN wizard.

The tunnel was initially created with IKEv1.

The Windows FortiClient configuration required IKEv2, so the tunnel was converted to IKEv2.

The Phase 1 configuration was then aligned with the FortiClient:

- IKEv2
- AES128-SHA256
- DH Group 14
- Pre-shared Key

EAP was enabled for user authentication:

- EAP: enabled
- EAP Identity: send-request
- User Group: `REMOTE-VPN-USERS`

The VPN client address pool was configured as:

`10.10.70.100 - 10.10.70.200`

Split tunneling was enabled so that the required HQ traffic uses the VPN.

The remote VPN firewall policy allows traffic from the VPN client range to the HQ Users network.
<img width="993" height="814" alt="image" src="https://github.com/user-attachments/assets/3593ae68-2e3e-4e13-af95-c0fa9104c891" />

---

## Authentication Model

Two different authentication concepts are used:

### Pre-Shared Key

The PSK authenticates the IPsec VPN tunnel.

### EAP

EAP provides user authentication.

The user authenticates with:

- Username: `vpn-user`
- Password: the local FortiGate user password

The user account belongs to:

`REMOTE-VPN-USERS`

---

## Verification

The first connection attempt failed with:

`Negotiation_Timeout`

The tunnel remained down.

The configuration was reviewed and EAP user authentication was found to be missing after converting the tunnel to IKEv2.

EAP was enabled and the user group was configured.

The connection was tested again.

FortiClient successfully connected to:

`USERS-VPN`

The client received:

`10.10.70.100`

The authenticated user was:

`vpn-user`

A connectivity test was performed from the VPN client to an HQ host:

`ping 10.10.10.100`

The ping was successful.

This confirmed that the VPN was not only connected but also able to carry traffic to the HQ Users network.
<img width="839" height="843" alt="image" src="https://github.com/user-attachments/assets/a1f03ca7-af08-4133-ac6f-4bdd294120cf" />
<img width="1340" height="669" alt="image" src="https://github.com/user-attachments/assets/a7637954-7dca-434e-b192-e042703440ba" />

---

## Troubleshooting

### Problem

FortiClient remained in:

`Connecting`

The tunnel later showed:

`Negotiation_Timeout`

### Investigation

The following were checked:

1. IKE version
2. Phase 1 proposal
3. DH Group
4. Phase 2 configuration
5. User authentication
6. FortiGate user group

### Root Cause

The tunnel had been converted from IKEv1 to IKEv2, but EAP user authentication was not configured.

### Fix

EAP was enabled and linked to:

`REMOTE-VPN-USERS`

### Result

The user authentication prompt appeared in FortiClient.

The VPN successfully connected and assigned:

`10.10.70.100`

---

## Security Considerations

- Use IKEv2 for modern remote-access VPN deployments where supported.
- Use strong encryption and authentication proposals.
- Use user authentication instead of relying only on a shared tunnel PSK.
- Use separate VPN user groups for access control.
- Limit VPN access with firewall policies.
- Do not give remote users unrestricted access to the entire internal network unless required.
- Use split tunneling carefully and only for approved traffic.
- Protect the VPN PSK and user credentials.
- Keep configuration backups before major VPN changes.

---

## Important Concepts Learned

### IKE

IKE (Internet Key Exchange) is used to negotiate the security parameters for IPsec.

IKEv1 and IKEv2 are different versions of the IKE protocol.

### Phase 1

Phase 1 establishes the IKE security association and negotiates the security parameters between the VPN peers.

### Phase 2

Phase 2 creates the IPsec Child SA and defines the traffic selectors used for protected traffic.

### EAP

EAP (Extensible Authentication Protocol) provides user authentication for the remote-access VPN.

### VPN vs Firewall Policy

The VPN provides secure connectivity to the FortiGate.

The firewall policy controls which internal resources the authenticated VPN user can access.

---

## Result

Remote Access VPN was successfully implemented and tested.

The final traffic path was:

`Windows PC → FortiClient → IKEv2/IPsec → FGT-HQ-01 → HQ VLAN10`

The VPN client received:

`10.10.70.100`

The authenticated user was:

`vpn-user`

Access to the HQ Users network was verified successfully.

## Lessons Learned

- IKEv2 and EAP have different roles.
- PSK and user credentials are not the same thing.
- A connected VPN does not automatically mean unrestricted network access.
- Firewall policies are required to control remote-user access.
- Troubleshooting should start with evidence instead of changing multiple VPN parameters at the same time.
