# Task 01 —  Topology Design & Deployment

## Scenario

The company is preparing a new HQ network. The network needs a secure
firewall, internal network, access switches, and a separate DMZ for
public services.

## Objective

Build the base enterprise topology in EVE-NG before starting the
configuration.

## Topology

The lab contains:

- FortiGate as the perimeter firewall
- Core Layer 3 switch
- Two access switches
- Internal client machines
- A separate DMZ web server
- External Internet connection

The FortiGate has three main connections:

- WAN connection to the Internet
- Internal connection to the Core switch
- DMZ connection to the Web Server

## Design

The internal network will later contain:

- VLAN 10 — Users
- VLAN 20 — Servers
- VLAN 30 — Guest
- VLAN 50 — Management

The DMZ will be used for public-facing services.

## Verification

The following topology requirements were completed:

- FortiGate connected to the Internet
- FortiGate connected to the Core switch
- FortiGate connected directly to the DMZ server
- Core switch connected to the access switches
- Client machines connected to the access switches
- Internal and DMZ networks are physically separated

## Configuration Status

No IP addresses, VLANs, routing, firewall policies, NAT,
or security policies were configured in this task.

## Security Considerations

The DMZ is separated from the internal network so that public-facing
services can be controlled by the FortiGate.

## Result

The base enterprise topology is ready for FortiGate configuration
and internal network design.
<img width="1221" height="776" alt="image" src="https://github.com/user-attachments/assets/0600dc57-cbda-4f33-88a5-936234ea1fda" />
