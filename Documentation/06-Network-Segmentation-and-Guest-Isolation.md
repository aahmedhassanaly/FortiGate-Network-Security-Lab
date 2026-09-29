# Task 6 — Network Segmentation & Guest Isolation

## Scenario

The network has different VLANs for Users, Servers, Guest devices, and Management.

The Guest VLAN should have Internet access, but it should not be able to access internal networks.

The goal of this task is to configure and verify Guest isolation using FortiGate firewall policies.

---

## Objective

- Allow Guest users to access the Internet.
- Block Guest access to the Users network.
- Block Guest access to the Servers network.
- Block Guest access to the Management network.
- Verify the firewall behavior with real traffic.

---

## Network Design

| VLAN | Name | Network | Gateway | Purpose |
|---|---|---|---|---|
| 10 | USERS | 10.10.10.0/24 | 10.10.10.1 | User devices |
| 20 | SERVERS | 10.10.20.0/24 | 10.10.20.1 | Servers |
| 30 | GUEST | 10.10.30.0/24 | 10.10.30.1 | Guest devices |
| 50 | MANAGEMENT | 10.10.50.0/24 | 10.10.50.1 | Infrastructure management |

For testing, Linux11 was temporarily connected to VLAN 30 and received:

    10.10.30.100/24

---

## Firewall Policies

Three explicit deny policies were created:

| Policy | Source | Destination | Action |
|---|---|---|---|
| DENY-GUEST-TO-USERS | VLAN30-GUEST | ZONE-USERS | DENY |
| DENY-GUEST-TO-SERVERS | VLAN30-GUEST | VLAN20-SERVERS | DENY |
| DENY-GUEST-TO-MANAGEMENT | VLAN30-GUEST | VLAN50-MANT | DENY |

The existing Guest Internet policy remains:

    POL-VLAN30-GUEST-TO-INTERNET

    Source: VLAN30-GUEST
    Destination: port1
    Action: ACCEPT
    NAT: Enabled

The Guest Internet policy is different from the internal deny policies because the destination interface is different.

---
<img width="1901" height="1012" alt="image" src="https://github.com/user-attachments/assets/d162c5a2-6eda-4e25-800c-efc627ad20ed" />

## Verification

Linux11 was temporarily placed in VLAN 30 and received an IP address using DHCP.

IP address:

    10.10.30.100

### Guest to Users

    ping -c 4 10.10.10.1

Result:

    Failed

Traffic from the Guest network to the Users network was blocked.

### Guest to Servers

    ping -c 4 10.10.20.1

Result:

    Failed

Traffic from the Guest network to the Servers network was blocked.

### Guest to Management

    ping -c 4 10.10.50.1

Result:

    Failed

Traffic from the Guest network to the Management network was blocked.

### Guest to Internet

    ping -c 4 8.8.8.8

Result:

    Successful

Guest users can access the Internet while internal network access is blocked.
<img width="1013" height="832" alt="image" src="https://github.com/user-attachments/assets/8a6e0a87-2623-48fd-a92b-958935f640c5" />

---

## Final Traffic Behavior

    VLAN30-GUEST
          |
          +----> VLAN10 USERS       DENY
          |
          +----> VLAN20 SERVERS     DENY
          |
          +----> VLAN50 MANAGEMENT  DENY
          |
          +----> Internet           ALLOW + NAT

---

## Troubleshooting

The first tests showed that Guest devices could not reach internal VLAN gateways.

This was expected because there were no policies allowing Guest-to-internal traffic.

Instead of relying only on the default deny behavior, explicit firewall policies were created for Guest isolation.

This provides clearer security rules and makes traffic investigation easier.

The firewall policy Hit Count was also checked after the tests to confirm that the deny policies received traffic.

---

## Security Considerations

Guest devices should not have access to internal networks.

Keeping Guest traffic isolated reduces the risk of unauthorized access to:

- User devices
- Servers
- Management interfaces

Internet access is still allowed through the existing Guest Internet policy.

The firewall rules should be reviewed regularly when the network changes.

---

## Lessons Learned

- VLAN separation alone is not the complete security control.
- FortiGate firewall policies control traffic between different interfaces.
- Guest networks should be isolated from internal networks.
- Explicit deny policies make the security design easier to understand and troubleshoot.
- Real traffic tests are required to verify firewall behavior.

---

## Result

Task 6 was completed successfully.

Guest devices can access the Internet but cannot access the Users, Servers, or Management networks.
