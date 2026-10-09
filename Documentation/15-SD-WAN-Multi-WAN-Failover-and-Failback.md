# Task 15 — SD-WAN Multi-WAN Failover and Failback

## Objective

Configure FortiGate SD-WAN with two routed WAN paths, monitor reachability to an Internet target, provide automatic path selection, and verify failover and failback from a VLAN10 client.

## Environment

- Platform: EVE-NG
- Firewall: `FGT-HQ-01`
- Primary path: `port4` through `R-WAN1`
- Backup path: `port1` through `R-WAN2`
- VLAN10 subnet: `10.10.10.0/24`
- SD-WAN zone: `virtual-wan-link`
- Health-check target: `8.8.8.8` using Ping

## WAN Topology and Addressing

| Device / Interface | Address | Role |
|---|---|---|
| FGT-HQ-01 `port4` | `172.16.10.2/30` | Primary WAN1 |
| R-WAN1 `Gi0/1` | `172.16.10.1/30` | WAN1 inside interface |
| FGT-HQ-01 `port1` | `172.16.20.2/30` | Backup WAN2 |
| R-WAN2 `Gi0/1` | `172.16.20.1/30` | WAN2 inside interface |
| R-WAN1 `Gi0/0` | DHCP: `192.168.1.227/24` observed | Cloud0 / upstream |
| R-WAN2 `Gi0/0` | DHCP: `192.168.1.104/24` observed | Cloud0 / upstream |

Both routers use Cloud0 and the same upstream home network. This is a lab simulation of two WAN paths, not two physically independent Internet providers.

## Configuration Summary

### 1. WAN Routers

Both Cisco IOSv routers were configured with:

- `Gi0/0` connected to Cloud0 and configured as NAT outside.
- `Gi0/1` configured as the inside interface.
- PAT overload using the respective Cloud0-facing interface.
- R-WAN1 inside subnet: `172.16.10.0/30`.
- R-WAN2 inside subnet: `172.16.20.0/30`.

Connectivity checks from R-WAN2 succeeded:
- Ping to `192.168.1.1`: successful.
- Ping to `8.8.8.8`: 5/5 replies.

### 2. FortiGate SD-WAN

Configured through the FortiGate GUI:

- SD-WAN zone: `virtual-wan-link`.
- Member `port4`, gateway `172.16.10.1`.
- Member `port1`, gateway `172.16.20.1`.
- Default route uses `virtual-wan-link`.
- SD-WAN rule: `SDWAN-INTERNET-FAILOVER`.
- WAN preference was configured using member costs; `port4` was preferred with cost 1 and `port1` used as the higher-cost alternative (cost 5).
- Performance SLA: `WANINTERNETHEALTH`, target `8.8.8.8`, protocol Ping.

Observed SLA sample:
- `port1`: approximately 46.86 ms latency, 0% packet loss, 6.85 ms jitter.
- `port4`: approximately 46.57 ms latency, 3.45% packet loss, 6.98 ms jitter.

These are point-in-time lab measurements, not guaranteed performance values.

### 3. VLAN10 Internet Policy

Created a firewall policy for VLAN10 Internet access:

- Incoming interface: `ZONE-USERS`.
- Outgoing interface: `virtual-wan-link`.
- Source: `VLAN10-USERS address` (as selected in the GUI).
- Destination: `all`.
- Schedule: `always`.
- Service: `ALL`.
- Action: `ACCEPT`.
- NAT: enabled.
- Existing security-profile settings were retained/selected where available.

### 4. Management Access

To preserve GUI access after moving FortiGate `port1` away from Cloud0, management access was tested through R-WAN1 at:

`https://192.168.1.227`

R-WAN1 forwards HTTPS to the FortiGate WAN1-side address `172.16.10.2:443`. FortiGate has a host route for the management workstation `192.168.1.144/32` via `172.16.10.1` on `port4`.

## Verification and Results

| Test | Result |
|---|---|
| R-WAN2 to upstream gateway `192.168.1.1` | Passed |
| R-WAN2 to Internet target `8.8.8.8` | Passed |
| FortiGate to WAN2 gateway `172.16.20.1` | Passed, 5/5 replies |
| VLAN10 client to gateway `10.10.10.1` | Passed |
| VLAN10 client to `8.8.8.8` | Passed |
| VLAN10 client DNS lookup | Passed |
| Management GUI through R-WAN1 | Passed |
| Failover after primary `port4` path was made unavailable | SD-WAN selected `port1` |
| Failback after `port4` was restored | SD-WAN selected `port4` again |

The failover and failback behavior was observed in the GUI and connectivity tests. The lab demonstrated automatic path selection between the two configured WAN paths.

## Troubleshooting Notes

- The old default gateway `192.168.1.1` on FortiGate `port1` became invalid after `port1` was changed to `172.16.20.2/30`; the default route was updated to use the SD-WAN zone.
- A test-only host route to `8.8.8.8/32` via `port4` was present during early troubleshooting. Review and remove this route if it remains, so it does not override normal SD-WAN route selection.
- A host route to the management workstation `192.168.1.144/32` via `port4` supports the return path for GUI management through R-WAN1. Do not remove it without first planning and testing an alternative management path.
- FortiGate GUI access was maintained through R-WAN1 during the migration.
- The two WAN routers share Cloud0 and the same upstream network. A Cloud0 or upstream home-network failure would affect both paths; the lab does not prove resilience to an independent ISP outage.

## Outcome

**Completed:** SD-WAN members, health check, WAN preference, VLAN10 Internet policy, and GUI-tested failover/failback were configured and verified in the EVE-NG lab.

## Evidence to Capture

Add screenshots to the repository when available:
1. SD-WAN members showing `port1` and `port4`.
2. `SDWAN-INTERNET-FAILOVER` rule and member cost/preference.
3. `WANINTERNETHEALTH` SLA results.
4. Static route using `virtual-wan-link`.
5. VLAN10-to-SD-WAN firewall policy with NAT enabled.
6. GUI showing `port1` selected during failover and `port4` selected after failback.
