# Task 5 — Firewall Policies and NAT

## Scenario

The HQ network now has multiple internal VLANs connected to the FortiGate.

The company needs controlled Internet access for internal networks.

The FortiGate will:

- Inspect traffic from internal VLANs.
- Apply firewall policies.
- Perform Source NAT for private IP addresses.
- Allow approved traffic to exit through the WAN interface.

The first functional test was performed using VLAN10.

---

## Objective

Configure Firewall Policies and Source NAT for the HQ VLANs.

The required policies are:

| Policy | Source Interface | Destination | NAT |
|---|---|---|---|
| POL-VLAN10-USERS-TO-INTERNET | ZONE-USERS | port1 | Enabled |
| POL-VLAN20-SERVERS-TO-INTERNET | VLAN20-SERVERS | port1 | Enabled |
| POL-VLAN30-GUEST-TO-INTERNET | VLAN30-GUEST | port1 | Enabled |
| POL-VLAN50-MANAGEMENT-TO-INTERNET | VLAN50-MANT | port1 | Enabled |

---

## Network Flow

The Internet traffic flow for VLAN10 is:

    Linux11
    10.10.10.100
         |
         v
    VLAN10-USERS
         |
         v
    ZONE-USERS
         |
         v
    FortiGate
         |
         | Source NAT
         v
    port1
    192.168.1.135
         |
         v
    Internet

---

## Baseline Test

Before creating the firewall policy, Internet connectivity from Linux11 was tested:

    ping -c 4 8.8.8.8

The test failed.

This was expected because there was no firewall policy allowing VLAN10 traffic to the Internet.

The FortiGate already had WAN connectivity through `port1`, but internal VLAN traffic was not yet allowed to use it.

---

## Policy 1 — VLAN10 Users

Policy name:

    POL-VLAN10-USERS-TO-INTERNET

Configuration:

- Incoming Interface: `ZONE-USERS`
- Outgoing Interface: `port1`
- Source: `all`
- Destination: `all`
- Schedule: `always`
- Service: `ALL`
- Action: `ACCEPT`
- NAT: Enabled
- Traffic Logging: Enabled

The `ZONE-USERS` interface zone contains:

    VLAN10-USERS

The zone was kept because it provides a logical security grouping for the user network.

---

## Policy 2 — VLAN20 Servers

Policy name:

    POL-VLAN20-SERVERS-TO-INTERNET

Configuration:

- Incoming Interface: `VLAN20-SERVERS`
- Outgoing Interface: `port1`
- Source: `all`
- Destination: `all`
- Schedule: `always`
- Service: `ALL`
- Action: `ACCEPT`
- NAT: Enabled

---

## Policy 3 — VLAN30 Guest

Policy name:

    POL-VLAN30-GUEST-TO-INTERNET

Configuration:

- Incoming Interface: `VLAN30-GUEST`
- Outgoing Interface: `port1`
- Source: `all`
- Destination: `all`
- Schedule: `always`
- Service: `ALL`
- Action: `ACCEPT`
- NAT: Enabled

---

## Policy 4 — VLAN50 Management

Policy name:

    POL-VLAN50-MANAGEMENT-TO-INTERNET

Configuration:

- Incoming Interface: `VLAN50-MANT`
- Outgoing Interface: `port1`
- Source: `all`
- Destination: `all`
- Schedule: `always`
- Service: `ALL`
- Action: `ACCEPT`
- NAT: Enabled

---

## NAT

The internal VLANs use private IP address ranges:

    10.10.10.0/24
    10.10.20.0/24
    10.10.30.0/24
    10.10.50.0/24

These addresses are not directly routable on the public Internet.

Source NAT allows FortiGate to translate the internal source address to the WAN-side address when traffic leaves through `port1`.

Example:

    Before NAT:

    Source: 10.10.10.100

    After NAT:

    Source: FortiGate WAN address

This allows internal clients to communicate with external destinations.
<img width="1914" height="827" alt="image" src="https://github.com/user-attachments/assets/a1ce481f-b769-4b73-b163-a76d62dd0484" />

---

## Verification

### VLAN10 Internet Test

After creating the firewall policy and enabling NAT, Linux11 tested Internet connectivity again:

    ping -c 4 8.8.8.8

The test was successful.

The final test showed:

    5 packets transmitted
    5 packets received
    0% packet loss

This verified that VLAN10 could reach the Internet through the FortiGate.

---
<img width="1003" height="871" alt="image" src="https://github.com/user-attachments/assets/9d4d499e-5225-4567-8a59-2a7e5d7c067c" />

## Policy Monitor Verification

The FortiGate Policy Monitor showed traffic for:

    POL-VLAN10-USERS-TO-INTERNET

Observed traffic included:

- Bytes: approximately 19.32 kB
- Packets: 230
- Hit Count: 2
- Active Sessions: 0

The traffic counters confirmed that the policy was not only configured but was actually processing traffic.

---

## CLI Verification

The following command was used to verify the firewall policies:

    show firewall policy

The configuration confirmed:

    POL-VLAN10-USERS-TO-INTERNET
    srcintf = ZONE-USERS
    dstintf = port1
    action = accept
    srcaddr = all
    dstaddr = all
    service = ALL
    nat = enable

The other policies were also confirmed:

    POL-VLAN20-SERVERS-TO-INTERNET
    srcintf = VLAN20-SERVERS
    dstintf = port1
    action = accept
    nat = enable

    POL-VLAN30-GUEST-TO-INTERNET
    srcintf = VLAN30-GUEST
    dstintf = port1
    action = accept
    nat = enable

    POL-VLAN50-MANAGEMENT-TO-INTERNET
    srcintf = VLAN50-MANT
    dstintf = port1
    action = accept
    nat = enable

---

## Important Verification Note

VLAN10 was functionally tested against the Internet.

VLAN20, VLAN30, and VLAN50 were verified through their firewall policy configuration, but they were not functionally tested because there are currently no client devices assigned to those VLANs.

Therefore:

- VLAN10 Internet access: Functionally verified
- VLAN20 Policy: Configuration verified
- VLAN30 Policy: Configuration verified
- VLAN50 Policy: Configuration verified

---

## Troubleshooting

### Problem

Linux11 could reach the FortiGate gateway but could not reach `8.8.8.8`.

### Evidence

Gateway connectivity worked:

    ping -c 4 10.10.10.1

Internet connectivity failed before the firewall policy was created.

### Cause

No firewall policy allowed traffic from the internal VLAN to the WAN interface.

### Fix

Created:

    POL-VLAN10-USERS-TO-INTERNET

with:

- Source interface: `ZONE-USERS`
- Destination interface: `port1`
- Action: `ACCEPT`
- NAT: Enabled

### Verification

Internet ping to `8.8.8.8` succeeded with:

    0% packet loss

---

## Security Considerations

The current policies use:

- Source: `all`
- Destination: `all`
- Service: `ALL`

This configuration is acceptable for the initial lab connectivity test, but it is broad.

In a production environment, firewall policies should normally be restricted based on:

- Source networks
- Destination networks
- Required services
- Security requirements
- User or application needs

Guest, server, and management traffic should not automatically receive the same security permissions as user traffic.

Further segmentation and security controls will be implemented in later tasks.

---

## Lessons Learned

- FortiGate firewall policies control whether traffic can pass between interfaces.
- A connected WAN interface does not automatically provide Internet access to internal VLANs.
- NAT is required when private internal addresses need to access external networks.
- Firewall policy counters are useful for proving that a policy is processing traffic.
- Policy configuration should be verified with CLI and GUI tools.
- Functional testing is different from configuration verification.
- Broad `ALL` policies are useful for initial testing but should be restricted for production use.

---

## Result

Task 5 was completed successfully.

The FortiGate now has separate Internet firewall policies for:

- Users
- Servers
- Guest
- Management

Source NAT is enabled on all four policies.

VLAN10 Internet connectivity was functionally verified with successful ICMP traffic to `8.8.8.8`.

The remaining VLAN policies are configured and ready for future functional testing when client devices are available.
