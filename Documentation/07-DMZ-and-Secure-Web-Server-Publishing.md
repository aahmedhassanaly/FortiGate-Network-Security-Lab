# Task 7 - DMZ and Secure Web Server Publishing

## Scenario

The company needs to publish a web server to external users without placing the server inside the internal user or server VLANs.

A separate DMZ network is used to isolate the public-facing web server.

The lab uses FortiGate to provide:

- DMZ network connectivity
- Destination NAT (DNAT)
- Virtual IP (VIP)
- HTTP firewall policy
- Traffic logging
- Secure separation between the DMZ and internal networks

---

## Objective

Build a DMZ network and publish a web server through FortiGate using a VIP.

The main flow is:

    External Client
          |
        port1
          |
      FortiGate
          |
        VIP/DNAT
          |
        port3
          |
     WEB-DMZ-01
     10.10.60.10

---

## Network Design

### DMZ Network

- Network: `10.10.60.0/24`
- FortiGate port3: `10.10.60.1/24`
- WEB-DMZ-01: `10.10.60.10/24`
- Default Gateway: `10.10.60.1`

### WAN / Lab External Address

- FortiGate port1: `192.168.1.135/24`

This address is used as the simulated external/WAN address in the lab.

Important:

`192.168.1.135` is a private IP address. It is not a real public Internet address.

---

# Task 7.1 - Prepare the DMZ Server

The WEB-DMZ-01 interface was initially down.

Enable the interface:

    ip link set eth0 up

Verify the interface:

    ip addr show eth0

The interface must be UP.

---

# Task 7.2 - Configure FortiGate DMZ Interface

Configure FortiGate `port3` as the DMZ interface.

Configuration:

- Interface: `port3`
- Addressing: Manual
- IP Address: `10.10.60.1/24`
- Administrative Access: PING
- Role: LAN
- Status: Enabled

The DMZ interface is independent from the VLAN trunk on `port2`.

This keeps the public-facing server in a separate network.
<img width="1778" height="705" alt="image" src="https://github.com/user-attachments/assets/9a604d78-9ed9-47af-aa20-c203a9e071e7" />

---

# Task 7.3 - Configure the Web Server IP

Configure WEB-DMZ-01:

    ip addr add 10.10.60.10/24 dev eth0

Configure the default route:

    ip route add default via 10.10.60.1

Verify:

    ip addr show eth0
    ip route

Expected result:

- IP address: `10.10.60.10/24`
- Default gateway: `10.10.60.1`

---

# Task 7.4 - Verify DMZ Connectivity

Test the FortiGate DMZ interface:

    ping -c 4 10.10.60.1

The ping was successful.

The server was also tested for Internet connectivity:

    ping -c 4 8.8.8.8

This failed because no general DMZ-to-Internet policy was created.

This was intentional.

The current task focuses on secure inbound web publishing rather than giving the DMZ server unrestricted Internet access.

---

# Task 7.5 - Create the Web Server VIP

Create a Virtual IP named:

    VIP-WEB-DMZ

The VIP configuration is:

- External IP: `192.168.1.135`
- External Interface: `port1`
- Port Forwarding: Enabled
- External Port: `80`
- Mapped IP: `10.10.60.10`
- Mapped Port: `80`

The VIP performs destination NAT.

Traffic received on:

    192.168.1.135:80

is translated to:

    10.10.60.10:80

Important:

The mapped IP must be the actual server IP.

It must NOT be the network address `10.10.60.0`.

---
<img width="1914" height="849" alt="image" src="https://github.com/user-attachments/assets/92d7d8ab-9a3f-469b-b737-6a8bc3c7c4ed" />

# Task 7.6 - Create the Inbound Firewall Policy

Create:

    POL-WAN-TO-WEB-DMZ

Configuration:

- Incoming Interface: `port1`
- Outgoing Interface: `port3`
- Source: `all`
- Destination: `VIP-WEB-DMZ`
- Schedule: `always`
- Service: `HTTP`
- Action: `ACCEPT`
- NAT: OFF
- Logging: All Sessions

The destination of the policy is the VIP object, not the real DMZ server IP.

The policy controls whether the DNAT traffic is allowed.

The VIP performs the destination translation.
<img width="1919" height="805" alt="image" src="https://github.com/user-attachments/assets/3762c51f-e9e9-48ba-a74f-8040289adecc" />

---

# Task 7.7 - Test Web Server Listener

The Alpine DMZ server did not have Nginx or another normal web server installed.

For lab testing, BusyBox `nc` was used as a temporary TCP listener.

Check the available options:

    nc --help

Start an IPv4 listener on the DMZ server:

    nc -lv -p 80 -s 10.10.60.10

Expected result:

    listening on 10.10.60.10:80

This listener is only for testing.

It is not a production web server.

---

# Task 7.8 - Verify FortiGate to DMZ Server

Test TCP/80 directly from FortiGate:

    execute telnet 10.10.60.10 80

Expected result:

    Trying 10.10.60.10...
    Connected to 10.10.60.10.

This test was successful.

This proves that FortiGate can reach the web server on TCP/80.

---

# Task 7.9 - Verify DNAT Traffic

A Windows client was used to test:

    curl http://192.168.1.135

The FortiGate sniffer showed:

    192.168.1.144 -> 10.10.60.10.80: syn

This proved that the VIP translated the destination from the FortiGate WAN address to the DMZ server.

The FortiGate also showed the traffic using:

    POL-WAN-TO-WEB-DMZ

with an `Accept` result.

---

# Important Test Limitation

The Windows client `192.168.1.144` was on the same `192.168.1.0/24` network as FortiGate port1.

Therefore, this test was a Hairpin/U-Turn NAT scenario, not a normal Internet-to-DMZ test.

A real external test requires either:

- A real public IP on the WAN side, or
- An external client connected to a separate simulated WAN network inside EVE-NG.

The lab did not add an external client because the current objective was to learn and configure DMZ, VIP, DNAT, and firewall policy without adding unnecessary topology complexity.

Therefore, the configuration was verified partially, but a true external end-to-end publishing test was not performed.

---

# Troubleshooting Performed

## Problem 1 - Web server interface was down

Evidence:

    eth0 was DOWN

Fix:

    ip link set eth0 up

---

## Problem 2 - Web server had no IP address

Fix:

    ip addr add 10.10.60.10/24 dev eth0

---

## Problem 3 - Incorrect VIP mapped address

The VIP initially used:

    10.10.60.0

This is the network address, not the server address.

Correct value:

    10.10.60.10

---

## Problem 4 - No TCP listener

The initial `nc` listener was not bound to the required IPv4 address.

The final test used:

    nc -lv -p 80 -s 10.10.60.10

This confirmed a listener on the server IP.

---

## Problem 5 - External test failed

The external Windows client could not reach:

    192.168.1.135

Reason:

`192.168.1.135` is a private lab WAN address and is not reachable as a public Internet address.

This does not mean that the VIP configuration itself is incorrect.

---

# Verification

The following items were successfully verified:

- `port3` configured as the DMZ interface
- DMZ network `10.10.60.0/24`
- WEB-DMZ-01 configured with `10.10.60.10`
- FortiGate can reach the server on TCP/80
- VIP configured for HTTP publishing
- VIP performs DNAT to `10.10.60.10`
- HTTP firewall policy allows traffic to the VIP
- Firewall logs show accepted HTTP sessions
- DMZ does not have unrestricted Internet access

---

# Security Considerations

A DMZ is used to reduce the risk of exposing internal networks when a public-facing server is compromised.

Important security principles:

- Keep public servers in a separate network.
- Publish only required services.
- Use specific firewall policies.
- Avoid allowing unrestricted DMZ-to-internal traffic.
- Avoid giving the DMZ unrestricted Internet access unless required.
- Log inbound connections.
- Use a real web server with security updates in production.
- Use HTTPS instead of plain HTTP for real applications.
- Review and restrict management access to the DMZ server.

The current HTTP listener is only a lab validation tool and should not be considered production-ready.

---

# Lessons Learned

This task demonstrated the difference between:

- DMZ
- VIP
- DNAT
- Firewall Policy
- Port Forwarding
- Hairpin NAT
- External publishing

A VIP does not automatically allow traffic.

The VIP performs the address translation, while the firewall policy controls whether the traffic is permitted.

The final traffic flow for normal external publishing is:

    External Client
          |
          v
    FortiGate port1
          |
          v
    VIP / DNAT
          |
          v
    FortiGate port3
          |
          v
    WEB-DMZ-01
    10.10.60.10:80

---

# Task Status

Task 7 - DMZ and Secure Web Server Publishing

Status: Configuration completed and partially verified.

The remaining limitation is the lack of a separate external client/public IP for a true Internet-to-DMZ end-to-end test.
