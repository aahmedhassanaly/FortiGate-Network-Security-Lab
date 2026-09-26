# Task 2 — FortiGate Initial Deployment and Hardening

## Scenario

A new FortiGate firewall was deployed for the HQ network.

Before configuring VLANs, routing, firewall policies, and VPNs, the firewall must be prepared and secured.

## Objective

The main goals were:

- Configure the FortiGate hostname.
- Configure the management IP.
- Configure the default route.
- Configure time and NTP.
- Secure management access.
- Change the administrator password.
- Create a configuration backup.

## Configuration

### Hostname

The FortiGate hostname was configured as:

    FGT-HQ-01

### Management Interface

The initial management interface is `port1`.

    IP Address: 192.168.1.135/24
    Status: Up

The default gateway is:

    192.168.1.1

The default route uses `port1`.
<img width="1901" height="848" alt="image" src="https://github.com/user-attachments/assets/39702f87-1854-421b-aa7a-5ab63d44bdc2" />

### Time and NTP

The system time was configured using the Cairo time zone.

NTP was enabled to keep the FortiGate time accurate.

Accurate time is important for logs, troubleshooting, and security events.

### Secure Administration

Administrative access was configured as:

    HTTPS: Enabled
    SSH: Enabled
    PING: Enabled
    HTTP: Disabled
    Telnet: Disabled

HTTPS is used for secure GUI access.

SSH is used for secure CLI access.

HTTP and Telnet were disabled because they do not provide encrypted management sessions.

### Administrator Account

The local `admin` account was secured by changing the administrator password.

Two-factor authentication and trusted hosts were not configured at this stage.

### Configuration Backup

A baseline configuration backup was created after the initial hardening.

This backup provides a recovery point before making major network changes.

## Verification

The following items were verified:

    Hostname: FGT-HQ-01
    port1: 192.168.1.135/24
    Default Gateway: 192.168.1.1
    HTTPS: Working
    SSH: Enabled
    HTTP: Disabled
    Telnet: Disabled
    NTP: Configured
    Configuration Backup: Completed

## Security Considerations

The FortiGate is now using secure management protocols.

HTTP and Telnet are not used for administration.

The administrator password was changed, and a baseline configuration backup was created before continuing with the network configuration.

## Result

The initial FortiGate deployment and hardening was completed successfully.

The firewall is ready for the next stage of the enterprise network configuration.

## Next Step

Configure the internal VLAN architecture and enterprise interfaces.
