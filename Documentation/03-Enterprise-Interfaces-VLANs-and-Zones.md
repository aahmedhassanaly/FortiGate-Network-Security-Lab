# Task 3 - Enterprise Interfaces, VLANs and Zones

## Scenario

The HQ network needs separate VLANs for users, servers, guests, and infrastructure management.

The FortiGate is used as the Layer 3 gateway for all VLANs. The Core and access switches are used mainly as Layer 2 devices.

## Objective

- Create VLAN interfaces on FortiGate.
- Configure the internal trunk between FortiGate and the Core.
- Create the required VLANs on the Core.
- Configure trunk links between the Core and access switches.
- Configure user access ports.
- Verify VLAN connectivity.
- Create a Users Zone for future firewall policies.

## VLAN Design

| VLAN | Name | Network | Gateway | Purpose |
|---|---|---|---|---|
| 10 | USERS | 10.10.10.0/24 | 10.10.10.1 | Employee devices |
| 20 | SERVERS | 10.10.20.0/24 | 10.10.20.1 | Servers |
| 30 | GUEST | 10.10.30.0/24 | 10.10.30.1 | Guest devices |
| 50 | MANAGEMENT | 10.10.50.0/24 | 10.10.50.1 | Infrastructure management |

## FortiGate Configuration

FortiGate `port2` is the internal trunk toward `CORE-HQ-01`.

The following VLAN interfaces were created:

- `VLAN10-USERS` - `10.10.10.1/24` - VLAN 10
- `VLAN20-SERVERS` - `10.10.20.1/24` - VLAN 20
- `VLAN30-GUEST` - `10.10.30.1/24` - VLAN 30
- `VLAN50-MANT` - `10.10.50.1/24` - VLAN 50

All VLAN interfaces use `port2` as the parent interface.

Management access on the VLAN interfaces is limited to ICMP:

    set allowaccess ping

The management VLAN does not allow HTTP or HTTPS access to the FortiGate.
<img width="1919" height="877" alt="image" src="https://github.com/user-attachments/assets/de89b110-e909-4b62-a0f4-e3170ec6ed8e" />

## Core Switch

The link between `FGT-HQ-01` and `CORE-HQ-01` was configured as an 802.1Q trunk.

Allowed VLANs:

    10,20,30,50

The following VLANs were created on the Core:

    VLAN 10 - USERS
    VLAN 20 - SERVERS
    VLAN 30 - GUEST
    VLAN 50 - MANAGEMENT

The Core does not use SVIs for these VLANs. The FortiGate provides the Layer 3 gateway for the VLANs.
<img width="1120" height="533" alt="image" src="https://github.com/user-attachments/assets/d911f529-f04a-4b1d-8500-4e4fc52001a9" />

## Access Switches

The uplinks between the Core and the access switches were configured as trunks.

Allowed VLANs:

    10,20,30,50

The client ports on SW1 and SW2 were configured as access ports in VLAN 10.

Current client placement:

    SW1
    ├── Linux11
    └── Linux6

    SW2
    ├── Linux8
    └── Linux7

## Users Zone

A FortiGate Zone named `ZONE-USERS` was created.

The Zone currently contains:

    VLAN10-USERS

The Zone is used for logical grouping and can be referenced by future firewall policies.

Because there is currently only one Users VLAN, the Zone does not provide an additional security function by itself.

## Verification

The FortiGate VLAN interfaces were verified using:

    diagnose netlink interface list VLAN10-USERS
    diagnose netlink interface list VLAN20-SERVERS
    diagnose netlink interface list VLAN30-GUEST
    diagnose netlink interface list VLAN50-MANT

All four interfaces reported:

    state=start
    flags=up

This confirms that the VLAN interfaces are operational.

VLAN 10 was also tested end-to-end.

Linux11 used:

    IP Address: 10.10.10.11/24
    Default Gateway: 10.10.10.1

Linux11 successfully reached the FortiGate VLAN 10 gateway.

Linux7 used:

    IP Address: 10.10.10.17/24
    Default Gateway: 10.10.10.1

Communication between Linux11 and Linux7 was successful.

This verified VLAN 10 connectivity across:

    Linux11
        |
       SW1
        |
    CORE-HQ-01
        |
    FGT-HQ-01

and across the Core to SW2.
<img width="903" height="830" alt="image" src="https://github.com/user-attachments/assets/e513b562-4acb-448a-9f35-a420006a4b0f" />
<img width="753" height="827" alt="image" src="https://github.com/user-attachments/assets/2e65990d-5e28-42ac-9d27-2423f71a665d" />

## Security Considerations

- The FortiGate is the Layer 3 gateway for the VLANs.
- Management access on `VLAN50-MANT` is limited to ping at this stage.
- Trunk links allow only the required VLANs.
- User ports are configured as access ports.
- Users, Servers, Guests, and Management are separated into different VLANs.
- No unnecessary Layer 3 configuration was added to the Core.

## Troubleshooting

The following checks were used during the implementation:

    show vlan brief
    show interfaces trunk

On FortiGate:

    show system interface VLAN10-USERS
    show system interface VLAN20-SERVERS
    show system interface VLAN30-GUEST
    show system interface VLAN50-MANT

    diagnose netlink interface list VLAN10-USERS
    diagnose netlink interface list VLAN20-SERVERS
    diagnose netlink interface list VLAN30-GUEST
    diagnose netlink interface list VLAN50-MANT

The main issue found during deployment was an access-switch uplink operating as `dynamic auto` instead of a static trunk. The uplink was changed to trunk mode and the required VLANs were allowed.

## Lessons Learned

- FortiGate can provide the default gateway for multiple VLANs using VLAN subinterfaces.
- The Core can remain Layer 2 when routing is centralized on the FortiGate.
- Trunk links must carry the required VLANs across the complete path.
- Access ports should be assigned to the correct user VLAN.
- A Zone is useful when multiple interfaces share the same security role, but it is not required for routing.

## Result

Task 3 is completed and verified.

The HQ VLAN structure, FortiGate VLAN gateways, trunk links, access ports, and Users Zone are configured.

Next task:

Task 4 - DHCP and Network Services
