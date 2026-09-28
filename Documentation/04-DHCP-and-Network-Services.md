# Task 4 — DHCP and Network Services

## Scenario

The company needs centralized DHCP services for the HQ network.

The FortiGate firewall will provide DHCP services for the internal VLANs. This allows clients to receive their IP address, subnet mask, default gateway, and other DHCP information automatically.

The existing FortiLink DHCP server must remain unchanged.

---

## Objective

Configure and verify DHCP services on FortiGate for the four HQ VLANs:

| VLAN | Name | Network | Gateway | DHCP Range |
|---|---|---|---|---|
| 10 | USERS | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.100–10.10.10.200 |
| 20 | SERVERS | 10.10.20.0/24 | 10.10.20.1 | 10.10.20.100–10.10.20.200 |
| 30 | GUEST | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.100–10.10.30.200 |
| 50 | MANAGEMENT | 10.10.50.0/24 | 10.10.50.1 | 10.10.50.100–10.10.50.200 |

Lease time:

- 604800 seconds
- 7 days

---

## DHCP Configuration

### VLAN10 — USERS

FortiGate DHCP configuration:

- Interface: `VLAN10-USERS`
- Network: `10.10.10.0/24`
- Default Gateway: `10.10.10.1`
- DHCP Range: `10.10.10.100–10.10.10.200`
- Netmask: `255.255.255.0`
- Lease Time: 7 days
- DNS Service: Local

### VLAN20 — SERVERS

FortiGate DHCP configuration:

- Interface: `VLAN20-SERVERS`
- Network: `10.10.20.0/24`
- Default Gateway: `10.10.20.1`
- DHCP Range: `10.10.20.100–10.10.20.200`
- Netmask: `255.255.255.0`
- Lease Time: 7 days
- DNS Service: Local

### VLAN30 — GUEST

FortiGate DHCP configuration:

- Interface: `VLAN30-GUEST`
- Network: `10.10.30.0/24`
- Default Gateway: `10.10.30.1`
- DHCP Range: `10.10.30.100–10.10.30.200`
- Netmask: `255.255.255.0`
- Lease Time: 7 days
- DNS Service: Local

### VLAN50 — MANAGEMENT

FortiGate DHCP configuration:

- Interface: `VLAN50-MANT`
- Network: `10.10.50.0/24`
- Default Gateway: `10.10.50.1`
- DHCP Range: `10.10.50.100–10.10.50.200`
- Netmask: `255.255.255.0`
- Lease Time: 7 days
- DNS Service: Local
<img width="1919" height="843" alt="image" src="https://github.com/user-attachments/assets/1dd0c92f-6c39-44c7-bdcf-0d049cfdd3a3" />

---



## Verification

### Linux11

Linux11 was configured to request an IP address using the Alpine Linux DHCP client:

    udhcpc -i eth0

Linux11 received:

    IP Address: 10.10.10.100
    Gateway: 10.10.10.1
    Lease Time: 604800 seconds

The gateway was tested successfully:

    ping -c 4 10.10.10.1

The ping was successful.

---

### Linux7

Linux7 was also configured to request an IP address using DHCP:

    udhcpc -i eth0

Linux7 received:

    IP Address: 10.10.10.101
    Gateway: 10.10.10.1
    Lease Time: 604800 seconds

Linux7 successfully reached the FortiGate gateway.

Linux7 also successfully reached Linux11 at:

    10.10.10.100

This verified connectivity across:

    Linux7
        |
       SW2
        |
    CORE-HQ-01
        |
    FortiGate
        |
    VLAN10-USERS

---
<img width="1017" height="827" alt="image" src="https://github.com/user-attachments/assets/d33ba561-f2dd-4a0f-9917-252a680efc94" />
<img width="954" height="812" alt="image" src="https://github.com/user-attachments/assets/f1002d11-30d9-48cb-97fc-53da9c65ac55" />

## FortiGate DHCP Lease Verification

The FortiGate command:

    execute dhcp lease-list

showed the following active leases:

    10.10.10.100
    00:50:00:00:0b:00
    udhcp 1.37.0

    10.10.10.101
    00:50:00:00:07:00
    udhcp 1.37.0

Both clients had valid DHCP lease expiry times approximately seven days after assignment.

---

## Final DHCP Configuration Verification

The FortiGate configuration was checked with:

    show system dhcp server

The output confirmed DHCP servers for:

- `VLAN10-USERS`
- `VLAN20-SERVERS`
- `VLAN30-GUEST`
- `VLAN50-MANT`

Each VLAN has the correct:

- Interface
- Default gateway
- Subnet mask
- DHCP address range

The existing `fortilink` DHCP configuration was also present and unchanged.

---


## Troubleshooting

### DHCP Client Does Not Receive an IP

Check the following:

1. Confirm the client interface is up.
2. Confirm the switch access port is assigned to the correct VLAN.
3. Confirm the switch uplinks are trunks.
4. Confirm the VLAN is allowed on the trunk.
5. Confirm the FortiGate VLAN interface is up.
6. Confirm the DHCP server is configured for the correct interface.
7. Check active DHCP leases on FortiGate.

For Alpine Linux, the DHCP client used in this lab is:

    udhcpc -i eth0

`dhclient` was not installed on the Alpine image.

---

## Security Considerations

- DHCP is provided separately for each VLAN.
- Each VLAN has its own IP address range.
- The FortiLink DHCP server was not modified.
- No Internet access was opened for internal VLANs during this task.
- Firewall policies and NAT will be configured separately.
- Management VLAN DHCP is kept separate from user and guest networks.

---

## Lessons Learned

- FortiGate can provide centralized DHCP services for multiple VLANs.
- DHCP must be configured on the correct FortiGate VLAN interface.
- A DHCP lease confirms that the client can reach the DHCP service through the Layer 2 path.
- DHCP configuration and Internet access are separate functions.
- DHCP lease verification is useful for troubleshooting client connectivity.
- Alpine Linux uses `udhcpc` as the DHCP client in this lab.

---

## Result

Task 4 was completed successfully.

The FortiGate DHCP service is configured for all four HQ VLANs.

Functional DHCP testing was completed successfully on VLAN10 using Linux11 and Linux7.

The remaining VLANs were verified through their FortiGate DHCP configurations and are ready for future client testing.
