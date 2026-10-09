# Task 15 — SD-WAN Multi-WAN Failover and Failback

## Scenario

The HQ network needs continued Internet connectivity if its preferred WAN path becomes unavailable. FortiGate must monitor both WAN paths, select the preferred healthy path, switch to the alternate path during a failure, and return to the preferred path after recovery.

This task builds on the existing enterprise VLANs and firewall policies. It focuses on WAN path selection, health monitoring, NAT, and evidence-based verification.

## Objectives

- Connect FortiGate to two routed WAN paths.
- Add both WAN interfaces to the SD-WAN zone.
- Configure an Internet health check using a reachable target.
- Set the preferred path and alternate path.
- Route VLAN10 Internet traffic through SD-WAN with NAT enabled.
- Test normal connectivity, failover, and failback.
- Preserve and verify administrative access while changing WAN connectivity.
- Document the limitations of the simulated WAN design.

## Environment

| Component | Configuration |
|---|---|
| Platform | EVE-NG |
| Firewall | `FGT-HQ-01` |
| Preferred path | FortiGate `port4` → `R-WAN1` |
| Alternate path | FortiGate `port1` → `R-WAN2` |
| Users network | VLAN10 — `10.10.10.0/24` |
| SD-WAN zone | `virtual-wan-link` |
| SD-WAN rule | `SDWAN-INTERNET-FAILOVER` |
| Performance SLA | `WANINTERNETHEALTH` |
| Health-check target | `8.8.8.8` using Ping |

## Topology and Addressing

```text
                    Cloud0 / Home Network
                       192.168.1.0/24
                         /        \
                     R-WAN1      R-WAN2
                       |            |
                 172.16.10.1   172.16.20.1
                       |            |
                 FGT port4      FGT port1
                       \            /
                    FortiGate SD-WAN
                    virtual-wan-link
                            |
                       VLAN10 Users
                       10.10.10.0/24
```

| Device / Interface | Address | Purpose |
|---|---|---|
| FGT-HQ-01 `port4` | `172.16.10.2/30` | Preferred WAN path |
| R-WAN1 `Gi0/1` | `172.16.10.1/30` | WAN1 transit gateway |
| R-WAN1 `Gi0/0` | `192.168.1.227/24` observed via DHCP | Cloud0 / upstream |
| FGT-HQ-01 `port1` | `172.16.20.2/30` | Alternate WAN path |
| R-WAN2 `Gi0/1` | `172.16.20.1/30` | WAN2 transit gateway |
| R-WAN2 `Gi0/0` | `192.168.1.104/24` observed via DHCP | Cloud0 / upstream |

**Topology limitation:** Both WAN routers connect to the same Cloud0 and upstream home network. The lab demonstrates SD-WAN path selection and failover between two routed paths, but it does not simulate two independent ISPs or prove resilience to a shared upstream outage.

## Configuration

### 1. WAN Router Connectivity and NAT

Both Cisco IOSv routers use their Cloud0-facing `Gi0/0` as the NAT outside interface and their FortiGate-facing `Gi0/1` as the inside interface. PAT overload is configured on each router for its respective transit subnet:

- R-WAN1: `172.16.10.0/30`
- R-WAN2: `172.16.20.0/30`

The WAN2 router successfully reached its upstream gateway `192.168.1.1` and Internet test target `8.8.8.8`.

### 2. FortiGate SD-WAN Members and Routing

The SD-WAN zone `virtual-wan-link` contains these members:

| Member | Gateway | Preference |
|---|---|---|
| `port4` | `172.16.10.1` | Preferred; cost 1 |
| `port1` | `172.16.20.1` | Alternate; cost 5 |

The default route was changed to use the SD-WAN zone instead of the old standalone gateway route. The rule `SDWAN-INTERNET-FAILOVER` was configured to prefer `port4` and use `port1` when the preferred path was unavailable.

The Performance SLA `WANINTERNETHEALTH` uses Ping to monitor `8.8.8.8`. The health check provides reachability/performance information for path selection; a configured member alone should not be treated as proof that end-to-end Internet connectivity is healthy.

### 3. VLAN10 Internet Firewall Policy

A firewall policy was configured to allow Users-zone traffic through the SD-WAN zone.

| Setting | Value |
|---|---|
| Incoming interface | `ZONE-USERS` |
| Outgoing interface | `virtual-wan-link` |
| Source | `VLAN10-USERS address` |
| Destination | `all` |
| Schedule | `always` |
| Service | `ALL` |
| Action | `ACCEPT` |
| NAT | Enabled |

NAT is enabled on this policy because client traffic is being sent toward the simulated Internet through the WAN routers. Existing security-profile settings were retained where applicable.

### 4. Preserve Management Access

During the WAN migration, FortiGate GUI access was maintained through R-WAN1 at:

`https://192.168.1.227`

R-WAN1 forwards HTTPS to FortiGate `172.16.10.2:443`. A host route for the management workstation `192.168.1.144/32` via `172.16.10.1` on `port4` supports the return path.

**Change-safety note:** Do not remove this management route or alter the preferred WAN path without confirming an alternative management route and testing access first.

## Verification and Results

| Test | Observed result |
|---|---|
| R-WAN2 → upstream gateway `192.168.1.1` | Passed |
| R-WAN2 → Internet target `8.8.8.8` | Passed (5/5 replies) |
| FortiGate → WAN2 gateway `172.16.20.1` | Passed (5/5 replies) |
| VLAN10 client → gateway `10.10.10.1` | Passed |
| VLAN10 client → `8.8.8.8` | Passed |
| VLAN10 client DNS lookup | Passed |
| FortiGate GUI through R-WAN1 | Passed |
| Preferred `port4` path made unavailable | SD-WAN selected `port1` |
| `port4` restored | SD-WAN selected `port4` again |

Failover and failback were observed in the FortiGate GUI and connectivity tests. SLA latency, packet loss, and jitter values varied between samples; they are point-in-time lab measurements and are not treated as fixed performance guarantees.

## Troubleshooting Notes

### Default route after changing the WAN interface

**Problem:** The old gateway `192.168.1.1` was no longer the correct next hop on FortiGate `port1` after the interface was changed to `172.16.20.2/30`.

**Resolution:** The default route was updated to use `virtual-wan-link`, with the WAN gateways configured on their respective SD-WAN members.

### Test-specific route can bypass SD-WAN

A host route to `8.8.8.8/32` via `port4` was used during earlier troubleshooting. Check whether it still exists. If it remains, evaluate and remove it when safe, because a more-specific route can direct the health-check destination outside the intended SD-WAN path-selection design.

Do not remove the management workstation route `192.168.1.144/32` as part of this cleanup; it serves a separate management-access purpose.

### Troubleshooting method

For any failed connectivity test, follow:

**Problem → Evidence → Hypothesis → Test → Fix → Verify**

Check the client gateway and DNS resolution, FortiGate policy match and NAT, SD-WAN rule/member state, SLA status, routing table, and the relevant router's upstream reachability. Change one variable at a time and retest.

## Security and Operational Considerations

- Use a health-check target that meaningfully tests the required upstream reachability; successful reachability to a router's directly connected gateway alone does not prove Internet access.
- Ensure the SD-WAN rule, Performance SLA, and default route work together as intended.
- Apply least-privilege source, destination, and service settings to production firewall policies instead of allowing `ALL` services by default.
- Keep management access separate from user Internet forwarding and verify the return path before WAN changes.
- Capture baseline and failure-state evidence before declaring failover successful.
- Treat both simulated WAN paths as sharing a common failure domain because they use the same Cloud0/upstream network.

## Key Lessons

1. SD-WAN centralizes path selection across multiple WAN members.
2. A Performance SLA monitors a target; member configuration alone does not guarantee end-to-end health.
3. Failover and failback must be tested deliberately, not inferred from configuration alone.
4. The firewall policy, NAT, routing, SD-WAN rule, and health check must all align for client traffic to work.
5. A more-specific static route can affect the expected path and should be reviewed during troubleshooting.
6. Management access and its return route must be protected during WAN changes.
7. Two logical WAN paths do not provide full ISP redundancy when both depend on the same upstream network.

## Evidence Checklist

Capture and add screenshots showing:

1. SD-WAN members `port4` and `port1` with their gateways.
2. `SDWAN-INTERNET-FAILOVER` rule and member preference/cost.
3. `WANINTERNETHEALTH` status and SLA measurements.
4. Default route through `virtual-wan-link`.
5. VLAN10-to-SD-WAN firewall policy with NAT enabled.
6. `port1` selected during failover.
7. `port4` selected after failback.
8. Successful VLAN10 client connectivity before and after path switching.

## Result

**Task 15 — COMPLETE**

FortiGate SD-WAN was configured with two routed WAN paths, a Performance SLA, preferred-path selection, and a VLAN10 Internet policy. Client connectivity, failover to `port1`, and failback to `port4` were observed and verified in the EVE-NG lab.

The result demonstrates SD-WAN operation between two simulated WAN paths. Independent-ISP redundancy remains outside the scope of this topology.
