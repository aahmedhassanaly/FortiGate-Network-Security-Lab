# Enterprise FortiGate Network Security Lab

![FortiGate](https://img.shields.io/badge/Firewall-FortiGate-red)
![FortiOS](https://img.shields.io/badge/FortiOS-7.4.12-orange)
![Platform](https://img.shields.io/badge/Platform-EVE--NG-blue)
![Tasks](https://img.shields.io/badge/Tasks-16%20documented-blue)

A practical enterprise network security lab built with **FortiGate and EVE-NG**. The project documents the design, configuration, verification, and troubleshooting of common firewall and network-security services in a simulated headquarters environment.

The goal is not just to configure features, but to understand traffic flow, validate behavior with evidence, investigate failures, and document operational limitations honestly.

## Lab Environment

| Component | Details |
| --- | --- |
| Virtualization / network emulation | EVE-NG Community 6.2.0-4 |
| Firewall platform | FortiGate-VM64-KVM |
| FortiOS | 7.4.12, build 2902 |
| Primary site | HQ |
| Firewall resilience | Active-Passive HA configuration documented in Task 16 |
| WAN resilience | Dual-WAN / SD-WAN scenario documented in Task 15 |

> **Lab scope:** This is a simulated environment, not a production deployment. The two WAN paths share an upstream Cloud0/home-network path, so the lab does not prove resilience against a failure of that shared upstream. HA configuration and synchronization have been observed, but a controlled HA failover and end-to-end service-continuity test remain pending.

## VLAN Design

| VLAN | Name | Network | Purpose |
| ---: | --- | --- | --- |
| 10 | USERS | 10.10.10.0/24 | Employee endpoints |
| 20 | SERVERS | 10.10.20.0/24 | Internal server network |
| 30 | GUEST | 10.10.30.0/24 | Guest devices with restricted access |
| 50 | MANAGEMENT | 10.10.50.0/24 | Infrastructure management |

## Skills Covered

- **Firewalling and NAT:** policy design, source NAT, VIP/DNAT, DMZ publishing, least-privilege access.
- **Segmentation:** VLANs, zones, inter-VLAN policy control, guest isolation.
- **VPN:** site-to-site IPsec and remote-access IPsec, including IKE negotiation and user authentication.
- **Security inspection:** Web Filtering, Application Control, IPS, certificate inspection, and deep-inspection troubleshooting.
- **Resilience:** SD-WAN health checks and path selection; Active-Passive HA configuration and synchronization.
- **Operations:** secure administration, configuration backup/recovery, logs, sessions, packet captures, and CLI diagnostics.
- **Troubleshooting method:** problem → evidence → hypothesis → test → fix → verification.

## Task Index

| # | Task | Main focus | Status |
| ---: | --- | --- | --- |
| 01 | [Topology Design and Deployment](Documentation/01-Topology-Design-and-Deployment.md) | EVE-NG topology and baseline design | ✓ |
| 02 | [FortiGate Initial Deployment and Hardening](Documentation/02-FortiGate-Initial-Deployment-and-Hardening.md) | Initial setup and secure management | ✓ |
| 03 | [Enterprise Interfaces, VLANs and Zones](Documentation/03-Enterprise-Interfaces-VLANs-and-Zones.md) | VLAN interfaces, trunks, and zones | ✓ |
| 04 | [DHCP and Network Services](Documentation/04-DHCP-and-Network-Services.md) | DHCP and network connectivity | ✓ |
| 05 | [Firewall Policies and NAT](Documentation/05-Firewall-Policies-and-NAT.md) | Firewall policies and source NAT | ✓ |
| 06 | [Network Segmentation and Guest Isolation](Documentation/06-Network-Segmentation-and-Guest-Isolation.md) | Inter-VLAN access control | ✓ |
| 07 | [DMZ and Secure Web Server Publishing](Documentation/07-DMZ-and-Secure-Web-Server-Publishing.md) | VIP/DNAT and web publishing | ✓ |
| 08 | [Logging, Monitoring and Traffic Investigation](Documentation/08-Logging-Monitoring-and-Traffic-Investigation.md) | Logs, sessions, and packet capture | ✓ |
| 09 | [Secure Administration, Backup and Recovery](Documentation/09-Secure-Administration-Backup-and-Recovery.md) | Administrative hardening and recovery | ✓ |
| 10 | [Site-to-Site IPsec VPN](Documentation/10-Site-to-Site-IPsec-VPN.md) | HQ-to-Branch IPsec | ✓ |
| 11 | [Remote Access VPN](Documentation/11-Remote-Access-VPN.md) | IKEv2, EAP, and split tunneling | ✓ |
| 12 | [Web Filtering](Documentation/12-Web-Filtering.md) | FortiGuard and static URL filtering | ✓ |
| 13 | [Application Control](Documentation/13-Application-Control.md) | Application identification and policy enforcement | ✓ |
| 14 | [Security Inspection and Troubleshooting](Documentation/14-Security-Inspection-and-Troubleshooting.md) | IPS and SSL/SSH inspection | ✓ |
| 15 | [SD-WAN Multi-WAN Failover and Failback](Documentation/15-SD-WAN-Multi-WAN-Failover-and-Failback.md) | Health checks and WAN path selection | ✓ |
| 16 | [FortiGate High Availability — Active-Passive](Documentation/16-FortiGate-High-Availability-Active-Passive.md) | HA membership, heartbeat, configuration synchronization | ✓ |

## Repository Structure

```text
.
├── README.md
└── Documentation/
    ├── 01-Topology-Design-and-Deployment.md
    ├── 02-FortiGate-Initial-Deployment-and-Hardening.md
    ├── 03-Enterprise-Interfaces-VLANs-and-Zones.md
    ├── ...
    ├── 15-SD-WAN-Multi-WAN-Failover-and-Failback.md
    └── 16-FortiGate-High-Availability-Active-Passive.md
```

## Project Approach

Each task records the relevant scenario, objective, configuration, verification evidence, troubleshooting, and security considerations where applicable. Known limitations and tests that have not been performed are stated explicitly rather than presented as successful results.

## About

Built by **Ahmed Hassan Aly** as a hands-on network security portfolio project. The environment is simulated in EVE-NG for learning and demonstration. Configuration coverage is not, by itself, a claim of production readiness; operational proficiency requires repeatable testing, troubleshooting, recovery, and evidence-based verification.
