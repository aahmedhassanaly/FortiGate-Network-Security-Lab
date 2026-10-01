# Task 8 – Logging, Monitoring and Traffic Investigation

## Scenario

The company needs to monitor network traffic and investigate allowed and denied connections on the FortiGate firewall.

The goal is to understand how to use FortiGate logs and packet capture to find the source of a network problem.

---

## Objective

In this task, we will:

- Review Forward Traffic logs.
- Identify allowed traffic.
- Identify denied traffic.
- Identify the firewall policy responsible for the traffic.
- Check NAT information.
- Understand the difference between traffic logs and active sessions.
- Use packet sniffing to confirm what happens to packets.
- Follow a basic troubleshooting process.

---

## Environment

### FortiGate

- Hostname: `FGT-HQ-01`

### VLANs

- VLAN 10 – USERS
- VLAN 20 – SERVERS
- VLAN 30 – GUEST
- VLAN 50 – MANAGEMENT

### Test Devices

- Linux7: `10.10.10.101`
- Guest test device: `10.10.30.100`

---

# 1. Verify Allowed Internet Traffic

From Linux7, generate Internet traffic:

    ping -c 4 8.8.8.8

Open:

**Log & Report → Forward Traffic**

The traffic log showed:

- Source: `10.10.10.101`
- Source Interface: `VLAN10-USERS`
- Destination: `8.8.8.8`
- Destination Interface: `port1`
- Application: `PING`
- Protocol: `1`
- Service: `PING`
- Action: `accept`
- Policy: `POL-VLAN10-USERS-TO-INTERNET`
- NAT Translation: `snat`
- Source NAT IP: `192.168.1.135`

This confirms that the traffic was allowed by the firewall policy and NAT was applied before sending the traffic to the WAN.

---<img width="1650" height="859" alt="image" src="https://github.com/user-attachments/assets/f93e6306-2738-48d3-aff4-5ad5089a26eb" />


# 2. Investigate Denied Traffic

From Linux7, test access to the Guest network:

    ping -c 4 10.10.30.100

Open:

**Log & Report → Forward Traffic**

The traffic appeared as a denied Forward Traffic event.

Important information included:

- Source: `10.10.10.101`
- Source Interface: `VLAN10-USERS`
- Destination: `10.10.30.100`
- Destination Interface: `VLAN30-GUEST`
- Application: `PING`
- Protocol: `1`
- Action: `deny`
- NAT Translation: `noop`
- Sent Packets: `0`
- Received Packets: `0`

This confirms that the traffic entered the FortiGate but was not allowed to reach the Guest network.
<img width="1640" height="863" alt="image" src="https://github.com/user-attachments/assets/90e8c78a-d4c2-466d-bd90-f2b5b01aa44d" />

---

# 3. Session Table Investigation

The denied traffic was checked in the FortiGate session table.

The result was:

    total session: 0

This is expected for the tested denied traffic because an active session was not created for the denied connection.

This shows an important difference:

- A traffic log can record a denied event.
- An active session table shows current active sessions.

Therefore, a denied log does not mean that an active session must exist.

---

# 4. Packet Sniffer Verification

A packet capture was used to verify what actually reached the FortiGate.

Example:

    diagnose sniffer packet any 'host 10.10.30.100' 4 10 a

The capture showed packets such as:

    10.10.10.101 -> 10.10.30.100: icmp: echo request

There were no ICMP echo replies from the destination.

The packet capture confirms that the ICMP requests were seen by the FortiGate, while the traffic log showed that the traffic was denied.

---
<img width="1762" height="865" alt="image" src="https://github.com/user-attachments/assets/1fc0d667-d46a-436e-85ea-e8a2ed055d93" />

# 5. Troubleshooting Method

The investigation followed this process:

**Problem**

Linux7 cannot reach `10.10.30.100`.

**Evidence**

The Forward Traffic log shows:

    Action: deny

**Hypothesis**

The FortiGate is blocking the traffic between the Users and Guest networks.

**Test**

Check the firewall log and packet capture.

**Result**

The packet capture shows ICMP requests from:

    10.10.10.101 → 10.10.30.100

The Forward Traffic log shows the traffic as denied.

**Conclusion**

The FortiGate is receiving the traffic and blocking it according to the firewall policy.

---

# 6. Local Traffic Note

Traffic involving the FortiGate itself is different from Forward Traffic.

For example:

    10.10.10.101 → 10.10.10.1

Here `10.10.10.1` is the FortiGate gateway.

However, during this lab the gateway ping was not confirmed as a visible Local Traffic log entry.

Therefore, this documentation does not claim that every ping to a FortiGate interface must appear in the Local Traffic log.

The important verified results for this task are the Forward Traffic logs and packet capture described above.

---

# 7. Security Considerations

Logging is important for firewall administration because it helps identify:

- Allowed connections
- Denied connections
- Source and destination addresses
- NAT activity
- Firewall policies
- Network communication problems

Logs should be reviewed together with other evidence such as packet captures and configuration information.

---

# 8. Lessons Learned

- Forward Traffic logs show traffic passing through the FortiGate.
- Logs can show whether traffic was accepted or denied.
- NAT information helps identify how the source address was translated.
- Denied traffic does not necessarily create an active session.
- Packet capture provides direct evidence of packets seen by the FortiGate.
- Troubleshooting should be based on evidence instead of assumptions.

## Task Status

**Completed**

Verified:

- Allowed Internet traffic
- NAT logging
- Denied inter-VLAN traffic
- Firewall policy identification
- Session table behavior
- Basic traffic troubleshooting
