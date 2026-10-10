# Task 16 – FortiGate High Availability (Active-Passive)

## Scenario

The HQ network has two FortiGate VMs, FGT-HQ-01 and FGT-HQ-02. They are configured as an Active-Passive HA cluster to provide firewall-device redundancy.

The existing lab already uses two routed WAN paths:

- WAN1: FGT port4 → R-WAN1
- WAN2: FGT port1 → R-WAN2
- HA heartbeat: port5 ↔ port5

The goal of this task is to establish the HA cluster, verify synchronization, preserve GUI access, and prepare for a controlled failover test. HA failover and SD-WAN failover are separate tests.

---

## Objective

- Configure an Active-Passive HA cluster.
- Verify cluster membership, heartbeat, and configuration synchronization.
- Preserve the existing WAN/SD-WAN configuration and management access.
- Understand how HA differs from WAN failover.
- Record what is verified and what still needs testing.

---

## Environment

| Setting | Value |
|---|---|
| Platform | EVE-NG Community 6.2.0-4 |
| FortiOS | 7.4.12 build 2902 |
| HA group name | HQ-HA |
| HA group ID | 16 |
| Mode | Active-Passive (a-p) |
| Heartbeat | port5 |
| Session pickup | Enabled |
| Override | Disabled |

## Topology

    R-WAN1 ─── port4 ──┐
                       │
                  FGT-HQ-01
                       │ port5 (HA heartbeat)
                  FGT-HQ-02
                       │
    R-WAN2 ─── port1 ──┘

Note: This is a logical illustration of the lab's WAN and HA roles, not a physical wiring diagram. Both simulated WAN routers use the same Cloud0/upstream home network, so the lab does not represent two fully independent ISPs.

---<img width="759" height="692" alt="image" src="https://github.com/user-attachments/assets/803895dc-5d4d-4f5d-a8f4-d3ca05547d7a" />


## HA Configuration

The observed HA configuration on the cluster included:

    set group-id 0
    set group-name "HQ"
    set mode a-p
    set hbdev "port5" 0
    set session-pickup enable
    set ha-mgmt-status enable
    set override disable

The configured monitored interfaces were port1, port2, and port4.
<img width="1077" height="685" alt="image" src="https://github.com/user-attachments/assets/6997d190-e9d8-45bb-82fd-3c8208d34b4e" />

### Important management note

The current HA management reservation was set to port4. However, port4 is also used as the WAN1/data interface in this lab. This is not the intended final out-of-band management design. Do not change this reservation until a dedicated interface and shared management network have been confirmed.

---

## Verification

Observed results during this session:

| Test | Result |
|---|---|
| HA cluster formed | Passed |
| Heartbeat on port5 | Up on both members |
| Configuration synchronization | Both members showed Synchronized with matching checksums |
| FGT-HQ-01 GUI access | Restored after adding a host route |
| Independent GUI access to FGT-HQ-02 through a dedicated management interface | Not tested |
| Controlled HA failover and service continuity | Not tested yet |

### GUI access troubleshooting

The GUI for FGT-HQ-01 had previously been accessed through:

    https://192.168.1.227

During troubleshooting, TCP/443 SYN packets reached port4 but no SYN-ACK was observed. A host route was then added for the management workstation:

    config router static
        edit 0
            set dst 192.168.1.144 255.255.255.255
            set gateway 172.16.10.1
            set device "port4"
        next
    end

The user confirmed that GUI access opened afterward. This confirms the GUI was restored in the current state; persistence and behavior during HA role changes still need verification.

---

## Active-Passive vs Active-Active

The lab remains in Active-Passive mode.

- Active-Passive is appropriate for learning firewall-device redundancy first.
- Active-Active does not automatically assign WAN1 to one firewall and WAN2 to the other.
- Active-Active does not itself provide a separate GUI address for the secondary member.
- Do not change HA mode before the current configuration and failover behavior have been validated.

## Troubleshooting Notes

- Configuration synchronization alone does not prove that failover works.
- HA device failover and SD-WAN WAN failover address different failure scenarios.
- Preserve console access and export configuration backups before any failover or management-interface changes.
- Do not remove the host route to 192.168.1.144/32 until an alternative management path is confirmed.
- Avoid using port4 for both production WAN traffic and reserved management in the final design.

---

## Result

The Active-Passive HA cluster was formed successfully. The heartbeat and configuration synchronization were observed as healthy, and GUI access to FGT-HQ-01 was restored.

The task is **in progress**, not fully complete: independent management access to FGT-HQ-02 and a controlled HA failover test with traffic verification remain outstanding.
<img width="1508" height="688" alt="image" src="https://github.com/user-attachments/assets/992dba14-cd4d-4aef-8840-2eb8e3713b05" />

## References

- [Fortinet Administration Guide – Out-of-band management with reserved management interfaces](https://docs.fortinet.com/document/fortigate/7.4.2/administration-guide/313152/out-of-band-management-with-reserved-management-interfaces)
- [Fortinet Community – Enable GUI access to a secondary HA unit](https://community.fortinet.com/fortigate-3/technical-tip-enable-gui-access-to-secondary-unit-in-fortigate-ha-cluster-using-a-reserved-management-interface-225908)
- [Task 15 – SD-WAN Multi-WAN Failover and Failback](15-SD-WAN-Multi-WAN-Failover-and-Failback.md)
