# FortiGate Network Security Lab

**Portfolio Project — Network Security / IT Infrastructure**

A hands-on FortiGate lab built in **EVE-NG** to practice enterprise firewall administration, network segmentation, VPNs, security inspection, SD-WAN, and high availability.

The focus is practical implementation, traffic-flow understanding, verification, troubleshooting, and clear technical documentation.

## Environment

| Component | Details |
|---|---|
| Network emulation | EVE-NG Community 6.2.0-4 |
| Firewall | FortiGate-VM64-KVM |
| FortiOS | 7.4.12 (build 2902) |
| Scenario | Enterprise headquarters (HQ) |
| Resilience topics | Dual-WAN / SD-WAN and Active-Passive HA |

## Network Topology

<!-- Add the final topology image in this section. -->

<p align="center">
  <em>Topology diagram will be added here.</em>
</p>

## VLAN Design

| VLAN | Name | Network | Purpose |
|---:|---|---|---|
| 10 | USERS | 10.10.10.0/24 | Employee endpoints |
| 20 | SERVERS | 10.10.20.0/24 | Internal servers |
| 30 | GUEST | 10.10.30.0/24 | Guest devices with restricted access |
| 50 | MANAGEMENT | 10.10.50.0/24 | Infrastructure management |

## Tasks

| # | Task | Status |
|---:|---|:---:|
| 01 | [Topology Design and Deployment](Documentation/01-Topology-Design-and-Deployment.md) | ✅ |
| 02 | [FortiGate Initial Deployment and Hardening](Documentation/02-FortiGate-Initial-Deployment-and-Hardening.md) | ✅ |
| 03 | [Enterprise Interfaces, VLANs and Zones](Documentation/03-Enterprise-Interfaces-VLANs-and-Zones.md) | ✅ |
| 04 | [DHCP and Network Services](Documentation/04-DHCP-and-Network-Services.md) | ✅ |
| 05 | [Firewall Policies and NAT](Documentation/05-Firewall-Policies-and-NAT.md) | ✅ |
| 06 | [Network Segmentation and Guest Isolation](Documentation/06-Network-Segmentation-and-Guest-Isolation.md) | ✅ |
| 07 | [DMZ and Secure Web Server Publishing](Documentation/07-DMZ-and-Secure-Web-Server-Publishing.md) | ✅ |
| 08 | [Logging, Monitoring and Traffic Investigation](Documentation/08-Logging-Monitoring-and-Traffic-Investigation.md) | ✅ |
| 09 | [Secure Administration, Backup and Recovery](Documentation/09-Secure-Administration-Backup-and-Recovery.md) | ✅ |
| 10 | [Site-to-Site IPsec VPN](Documentation/10-Site-to-Site-IPsec-VPN.md) | ✅ |
| 11 | [Remote Access VPN](Documentation/11-Remote-Access-VPN.md) | ✅ |
| 12 | [Web Filtering](Documentation/12-Web-Filtering.md) | ✅ |
| 13 | [Application Control](Documentation/13-Application-Control.md) | ✅ |
| 14 | [Security Inspection and Troubleshooting](Documentation/14-Security-Inspection-and-Troubleshooting.md) | ✅ |
| 15 | [SD-WAN Multi-WAN Failover and Failback](Documentation/15-SD-WAN-Multi-WAN-Failover-and-Failback.md) | ✅ |
| 16 | [FortiGate High Availability — Active-Passive](Documentation/16-FortiGate-High-Availability-Active-Passive.md) | ✅ |

Detailed task notes and implementation evidence are available in the [Documentation](Documentation/) directory.

## Core Skills

### Firewall & Network
- Firewall policies and source NAT
- Virtual IPs (VIPs), DNAT, and DMZ publishing
- VLANs, zones, inter-VLAN policies, and guest isolation
- DHCP and network services

### VPN & Security
- Site-to-site IPsec VPN
- Remote-access IPsec VPN
- Web Filtering and Application Control
- IPS and SSL inspection fundamentals

### Resilience & Operations
- SD-WAN health checks and path selection
- Active-Passive HA configuration and synchronization
- Secure administration and configuration backup
- Traffic logs, session inspection, packet captures, and CLI troubleshooting

## Troubleshooting Method

The lab follows a repeatable troubleshooting workflow:

```text
Problem
  ↓
Collect evidence
  ↓
Form a hypothesis
  ↓
Test
  ↓
Apply a fix
  ↓
Verify
  ↓
Document
```

## Lab Scope & Limitations

This is a simulated learning environment, not a production deployment.

- The two WAN paths share an upstream Cloud0/home-network path; therefore, the setup does not demonstrate resilience to failure of that shared upstream.
- HA membership and configuration synchronization were observed, but a controlled HA failover and end-to-end service-continuity test remain to be completed.
- Some task-specific validation was limited by the available lab clients and test setup. Refer to each task's documentation for its exact scope and evidence.

## Repository Structure

```text
.
├── README.md
└── Documentation/
    ├── 01-Topology-Design-and-Deployment.md
    ├── 02-FortiGate-Initial-Deployment-and-Hardening.md
    ├── ...
    ├── 15-SD-WAN-Multi-WAN-Failover-and-Failback.md
    └── 16-FortiGate-High-Availability-Active-Passive.md
```

## About

Built by **Ahmed Hassan Aly** as a hands-on network security portfolio project. The goal is to demonstrate practical firewall and infrastructure skills through implementation, verification, troubleshooting, and documentation—not merely feature configuration.
