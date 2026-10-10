# Enterprise FortiGate Network Security Lab

![FortiGate](https://img.shields.io/badge/Firewall-FortiGate-red)
![FortiOS](https://img.shields.io/badge/FortiOS-7.4.12-orange)
![EVE-NG](https://img.shields.io/badge/Platform-EVE--NG-blue)
![Tasks](https://img.shields.io/badge/Tasks-15%2F15%20completed-brightgreen)

A hands-on enterprise network security lab built with **FortiGate** and **EVE-NG**. It simulates a company HQ that needs secure Internet access, network segmentation, a DMZ for public services, site-to-site and remote-access VPNs, security inspection, centralized logging, and multi-WAN resilience.

Every task is documented the way it would be in a real environment: scenario, objective, configuration, verification, troubleshooting, and security considerations.

## Network Topology

<p align="center">
  <img src="https://github.com/user-attachments/assets/0600dc57-cbda-4f33-88a5-936234ea1fda" alt="Enterprise FortiGate Network Topology" width="900">
</p>

<p align="center"><em>HQ topology built in EVE-NG: FortiGate perimeter firewall, Core Layer 3 switch, access switches, internal clients, and a separate DMZ web server.</em></p>

## Environment

| Component | Details |
| --------- | ------- |
| Firewall | FortiGate (`FGT-HQ-01`), FortiOS 7.4.12 |
| Platform | EVE-NG |
| Internal design | Core Layer 3 switch + two access switches |
| DMZ | Dedicated interface to a public web server |

### VLAN Design

| VLAN | Name | Network | Purpose |
| ---- | ---- | ------- | ------- |
| 10 | USERS | 10.10.10.0/24 | Employee devices |
| 20 | SERVERS | 10.10.20.0/24 | Internal servers |
| 30 | GUEST | 10.10.30.0/24 | Guest devices (isolated) |
| 50 | MANAGEMENT | 10.10.50.0/24 | Infrastructure management |

## Skills Demonstrated

- **Network security:** firewall policies, Source NAT, VIP/DNAT, DMZ design, zone-based segmentation, guest isolation
- **VPN:** site-to-site IPsec (IKEv2) and remote-access IPsec with EAP authentication and split tunneling
- **Security profiles:** Web Filtering, Application Control, IPS, SSL/SSH Inspection
- **Resilience:** SD-WAN with health checks, failover, and failback
- **Operations:** logging and traffic investigation, secure administration, configuration backup and recovery
- **Troubleshooting:** CLI diagnostics, log analysis, and root-cause investigation

## Tasks

| # | Task | Focus | Status |
| - | ---- | ----- | ------ |
| 01 | [Topology Design and Deployment](Documentation/01-Topology-Design-and-Deployment.md) | EVE-NG enterprise topology | ✅ |
| 02 | [FortiGate Initial Deployment and Hardening](Documentation/02-FortiGate-Initial-Deployment-and-Hardening.md) | Hostname, NTP, secure management | ✅ |
| 03 | [Enterprise Interfaces, VLANs and Zones](Documentation/03-Enterprise-Interfaces-VLANs-and-Zones.md) | VLAN interfaces, trunks, zones | ✅ |
| 04 | [DHCP and Network Services](Documentation/04-DHCP-and-Network-Services.md) | DHCP and core network services | ✅ |
| 05 | [Firewall Policies and NAT](Documentation/05-Firewall-Policies-and-NAT.md) | Internet access policies, Source NAT | ✅ |
| 06 | [Network Segmentation and Guest Isolation](Documentation/06-Network-Segmentation-and-Guest-Isolation.md) | Least-privilege inter-VLAN control | ✅ |
| 07 | [DMZ and Secure Web Server Publishing](Documentation/07-DMZ-and-Secure-Web-Server-Publishing.md) | VIP/DNAT publishing | ✅ |
| 08 | [Logging, Monitoring and Traffic Investigation](Documentation/08-Logging-Monitoring-and-Traffic-Investigation.md) | Log analysis, traffic tracing | ✅ |
| 09 | [Secure Administration, Backup and Recovery](Documentation/09-Secure-Administration-Backup-and-Recovery.md) | Admin hardening, backup/restore | ✅ |
| 10 | [Site-to-Site IPsec VPN](Documentation/10-Site-to-Site-IPsec-VPN.md) | HQ to Branch, IKEv2 Phase 1/2 | ✅ |
| 11 | [Remote Access VPN](Documentation/11-Remote-Access-VPN.md) | IKEv2, EAP, split tunneling | ✅ |
| 12 | [Web Filtering](Documentation/12-Web-Filtering.md) | FortiGuard and static URL filtering | ✅ |
| 13 | [Application Control](Documentation/13-Application-Control.md) | Application-level policy | ✅ |
| 14 | [Security Inspection and Troubleshooting](Documentation/14-Security-Inspection-and-Troubleshooting.md) | IPS, SSL/SSH inspection | ✅ |
| 15 | [SD-WAN Multi-WAN Failover and Failback](Documentation/15-SD-WAN-Multi-WAN-Failover-and-Failback.md) | Health checks, failover, failback | ✅ |

## Repository Structure

```text
.
├── README.md
└── Documentation/
    ├── 01-Topology-Design-and-Deployment.md
    ├── ...
    └── 15-SD-WAN-Multi-WAN-Failover-and-Failback.md
```

## About

Built by **Ahmed Hassan Aly** as a practical network security portfolio project. The lab is a simulation in EVE-NG and is intended for learning and demonstration.
