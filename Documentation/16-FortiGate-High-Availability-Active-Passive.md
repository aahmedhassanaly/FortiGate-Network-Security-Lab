# Task 16 — FortiGate High Availability (Active-Passive)

## Scenario

The HQ network uses two FortiGate-VM appliances in an HA cluster. The goal is to provide firewall-device redundancy while keeping the existing SD-WAN and WAN management paths intact.

This task focuses on establishing and documenting Active-Passive HA. Active-Active is intentionally not enabled: it is not required to provide device failover and would add another variable before the HA failure scenarios have been verified.

## Objectives

- Form an Active-Passive HA cluster between `FGT-HQ-01` and `FGT-HQ-02`.
- Use a dedicated heartbeat link between the two FortiGate VMs.
- Verify cluster membership and configuration synchronization.
- Preserve the existing SD-WAN/WAN configuration and administrative access.
- Understand the distinction between firewall-device HA failover and SD-WAN WAN failover.
- Record the remaining validation items honestly; do not claim failover is proven until tested.

## Environment

| Component | Observed configuration |
|---|---|
| Platform | EVE-NG Community 6.2.0-4 |
| FortiOS | FortiOS 7.4.12, build 2902 |
| HA group name | `HQ-HA` |
| HA group ID | `16` |
| HA mode | Active-Passive (`a-p`) |
| Heartbeat interface | `port5` |
| Session pickup | Enabled |
| Override | Disabled |
| `FGT-HQ-01` serial | `FGVMSLTM26044832` |
| `FGT-HQ-02` serial | `FGVMSLTM26047560` |

## Existing WAN and management topology

| Device / interface | Address or role |
|---|---|
| `FGT-HQ-01 port4` | `172.16.10.2/30`, preferred WAN path through R-WAN1 |
| `R-WAN1 Gi0/1` | `172.16.10.1/30` |
| `R-WAN1 Gi0/0` | Cloud0/upstream address observed as `192.168.1.227/24` |
| `FGT-HQ-01 port1` | `172.16.20.2/30`, alternate WAN path through R-WAN2 in the documented SD-WAN design |
| `R-WAN2 Gi0/1` | `172.16.20.1/30` |
| `R-WAN2 Gi0/0` | Cloud0/upstream address observed as `192.168.1.104/24` |
| `FGT-HQ-01 port5` ↔ `FGT-HQ-02 port5` | HA heartbeat link |
| `FGT-HQ-01` GUI | Previously restored through `https://192.168.1.227` |
| `FGT-HQ-02` GUI | Previously accessible through `https://192.168.1.104:8444` before HA changes; current independent access must be revalidated |

**Topology note:** The two simulated WAN routers use the same Cloud0/upstream home network. This models two routed paths but does not prove redundancy against a shared upstream/ISP outage.

## HA configuration observed

The Primary's `show system ha` output showed:

```text
set group-id 16
set group-name "HQ-HA"
set mode a-p
set hbdev "port5" 0
set session-pickup enable
set ha-mgmt-status enable
set override disable
set monitor "port1" "port2" "port4"
```

The HA management reservation was configured to use `port4`. This is a known design concern because `port4` is also the WAN1/data interface. Do not treat the management reservation as a completed independent-management design. A dedicated unused interface and a separate management network should be evaluated before changing it.

## Configuration and observed results

- HA group `HQ-HA`, ID `16`, formed successfully in Active-Passive mode.
- `port5` heartbeat was observed up on both members.
- Both members were shown as synchronized with matching checksums.
- At the time of the captured GUI status, `FGT-HQ-02` was Primary (priority 100) and `FGT-HQ-01` was Secondary (priority 128). HA election and current role can change; use live status rather than assuming a permanent role from the hostname.
- The GUI path to `FGT-HQ-01` at `https://192.168.1.227` stopped responding after HA was introduced. Packet capture showed incoming TCP/443 SYNs on `port4` without a SYN-ACK. A host route for the management workstation `192.168.1.144/32` via `172.16.10.1` on `port4` was then added, after which the user reported GUI access restored.
- Static route used for that management return path:

```shell
config router static
    edit 0
        set dst 192.168.1.144 255.255.255.255
        set gateway 172.16.10.1
        set device "port4"
    next
end
```

The GUI restoration was user-confirmed. The route's persistence and behavior during HA role changes still need to be verified.

## Verification checklist

| Check | Status |
|---|---|
| HA cluster formed | Confirmed |
| Heartbeat on `port5` | Confirmed up |
| Configuration synchronization | Confirmed synchronized at captured time |
| GUI to `FGT-HQ-01` restored | User-confirmed after adding host route |
| Independent GUI access to `FGT-HQ-02` through a dedicated reserved management interface | Not completed |
| Controlled firewall-device failover test | Not yet verified in this task |
| User traffic continuity during HA failover | Not yet verified |
| WAN1-only failure while firewall remains healthy | Separate SD-WAN test; do not infer from HA status |
| WAN2-only failure and recovery | Separate SD-WAN test; do not infer from HA status |

## Active-Passive vs Active-Active decision

Keep this lab in **Active-Passive** for now.

- Active-Passive is sufficient to learn and validate firewall-device redundancy.
- Active-Active does not automatically split WAN1 and WAN2 between the two appliances, nor does it solve independent GUI management.
- Active-Active can distribute eligible session processing, but its behavior and benefits depend on traffic types and workload.
- Do not switch modes until baseline HA failover, recovery, management access, and traffic behavior have been tested and documented.

## Operational cautions

1. Do not test HA failover until you have console access or another recovery path to both VMs.
2. Save current FortiGate configurations before modifying HA, WAN interfaces, monitor interfaces, or management reservation.
3. Avoid using `port4` simultaneously as a production WAN path and a reserved management interface in the final design.
4. Keep device HA failover and SD-WAN WAN failover as separate tests with separate evidence.
5. The existing host route `192.168.1.144/32` is a management workaround; do not remove it until an alternative access path is confirmed.
6. Do not report HA failover as successful solely because the members are synchronized.

## Next steps

1. Verify the current cluster state with `get system ha status` and confirm both members remain in sync.
2. Plan dedicated management connectivity using an unused interface (candidate `port6` or `port7`, after checking actual interface configuration) and a shared management L2 segment.
3. Configure a unique management IP per member and validate HTTPS access to both devices, following Fortinet's reserved-management-interface guidance.
4. Perform a controlled Active-Passive failover test only after out-of-band/console access and configuration backups are confirmed.
5. Test SD-WAN failover/failback independently from HA failover.

## References

- [Fortinet Administration Guide — Out-of-band management with reserved management interfaces (FortiOS 7.4)](https://docs.fortinet.com/document/fortigate/7.4.2/administration-guide/313152/out-of-band-management-with-reserved-management-interfaces)
- [Fortinet Community — Enable GUI access to secondary unit using a reserved management interface](https://community.fortinet.com/fortigate-3/technical-tip-enable-gui-access-to-secondary-unit-in-fortigate-ha-cluster-using-a-reserved-management-interface-225908)
- [Task 15 — SD-WAN Multi-WAN Failover and Failback](15-SD-WAN-Multi-WAN-Failover-and-Failback.md)
