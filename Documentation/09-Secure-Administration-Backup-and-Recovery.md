# Task 9 — Secure Administration, Backup & Recovery

## Scenario

The FortiGate firewall is used as the main security device for the lab network.

A secure administration and recovery process is required to protect the firewall configuration and allow recovery after a configuration problem or device failure.

The task focuses on:

- Secure administrative access
- Trusted host restrictions
- Administrator session security
- Configuration backup
- Encrypted backup
- Recovery planning

---

## Objective

The objectives of this task were:

- Secure FortiGate administrative access.
- Restrict administrative access to the management network.
- Use HTTPS and SSH for administration.
- Configure a short administrator idle timeout.
- Create an encrypted configuration backup.
- Understand the difference between backup encryption and password masking.
- Define a practical recovery procedure.

---

## Environment

| Component | Configuration |
|---|---|
| Firewall | FGT-HQ-01 |
| FortiOS | 7.4.12 |
| Management Interface | port1 |
| Management Network | 192.168.1.0/24 |
| FortiGate Management IP | 192.168.1.135 |
| HTTPS | Port 443 |
| SSH | Port 22 |
| Admin Idle Timeout | 5 minutes |

---

## 1. Secure Administrative Access

FortiGate management access was configured through the management interface.

Enabled administrative services:

- HTTPS
- SSH
- PING

HTTP access is redirected to HTTPS.

Telnet was not enabled as an administrative service.

The FortiGate GUI is accessed using HTTPS instead of plain HTTP.

---

## 2. Trusted Hosts

Administrator access was restricted to the management network.

Trusted Host:

    Network: 192.168.1.0
    Netmask: 255.255.255.0

This allows administrators from the management LAN to access the administrator account.
<img width="1631" height="415" alt="image" src="https://github.com/user-attachments/assets/d4bdae53-912e-482e-bbaa-dde7b02458e6" />

### Security Note

Using the whole management subnet is practical for this lab.

In a real company, administration would normally be more restricted, for example:

- Dedicated management VLAN
- Jump server
- Administrator workstation subnet
- VPN management access

The goal is to reduce the number of systems that can attempt administrative access.

---

## 3. Administrator Session Security

The administrator idle timeout was configured to:

    5 minutes

This reduces the risk of leaving an authenticated FortiGate session open on an unattended workstation.
<img width="1014" height="664" alt="image" src="https://github.com/user-attachments/assets/76b54de4-e05d-45fd-8b2d-6414297fe7e4" />

---

## 4. Two-Factor Authentication Recovery Incident

Two-factor authentication was tested on the administrator account.

The FortiToken option was enabled during testing, which caused the administrator login to request a token code.

The FortiToken Mobile device was not available, so GUI and normal console access could not be used to complete the login.

The FortiGate was recovered using the original configuration backup created before the 2FA change.

### Lesson Learned

Security changes should be tested carefully before closing the existing administrative session.

A production environment should have:

- More than one administrative account when appropriate
- Appropriate recovery procedures
- Configuration backups
- Documented access to authentication systems
- A tested recovery process

A security feature should not be enabled on the only administrator account without confirming that recovery access is available.

---

## 5. Configuration Backup

A full FortiGate configuration backup was created after the lab configuration was restored.

Backup encryption was enabled.

### Backup Settings

- File format: Default
- Password Mask: Disabled
- Encryption: Enabled
- Encryption password: Stored securely outside GitHub

### Password Mask vs Encryption

**Password Mask**

Password Mask is intended to hide or mask sensitive password information in a configuration file.

The FortiGate interface indicates that this type of configuration is intended for Fortinet Support and should not be used as a normal restore configuration.

**Encryption**

Encryption protects the backup configuration file with a password.

The encryption password is required when restoring the encrypted configuration.

Therefore, the recovery backup used in this lab was created with:

    Password Mask: OFF
    Encryption: ON

---

## 6. Recovery Procedure

A backup is useful only if it can support recovery.

The documented recovery process is:

    FortiGate Failure
            ↓
    Deploy or replace FortiGate
            ↓
    Restore the last known-good configuration
            ↓
    Verify interfaces
            ↓
    Verify routing
            ↓
    Verify firewall policies
            ↓
    Verify NAT and VIP configuration
            ↓
    Test client connectivity
            ↓
    Return service to normal operation

The encrypted backup password must be available during the restore process.

---

## 7. Recovery and Change Management

Before making important FortiGate changes:

1. Create a configuration backup.
2. Record the planned change.
3. Make the configuration change.
4. Verify the result.
5. Keep the previous backup as a rollback point.

For risky security changes, recovery access should be confirmed before applying the change.
<img width="1900" height="822" alt="image" src="https://github.com/user-attachments/assets/d2b037ff-bd4c-4c44-ac4b-12cd8059f8d4" />

---

## 8. Verification

The following items were verified during the task:

- HTTPS administration available.
- SSH administration available.
- HTTP redirects to HTTPS.
- Trusted Host restriction configured for the management LAN.
- Administrator idle timeout set to 5 minutes.
- FortiGate configuration backup created.
- Backup encryption enabled.
- Backup password required for restore.
- Original configuration backup successfully restored after the 2FA lockout incident.
- Previous lab configuration was restored successfully.

---

## 9. Troubleshooting Example

### Problem

Administrator access was blocked after enabling FortiToken authentication.

### Evidence

The FortiGate login page requested a token code that was not available.

### Hypothesis

The administrator account was configured to require two-factor authentication.

### Recovery

The original known-good configuration backup was restored.

### Verification

After the restore:

- Administrator access worked again.
- The previous FortiGate settings were restored.
- The network configuration from the previous lab tasks was available again.

### Lesson Learned

Always maintain a known-good configuration backup before changing critical administrative or authentication settings.

---

## 10. Security Considerations

- Use HTTPS instead of HTTP for GUI administration.
- Use SSH instead of Telnet for CLI administration.
- Restrict administrator access using Trusted Hosts or a dedicated management network.
- Use a short administrative session timeout.
- Protect configuration backups with encryption.
- Never upload configuration backups or backup passwords to GitHub.
- Maintain a tested recovery procedure.
- Avoid depending on a single administrator authentication method.

---

## Result

Task 9 was completed successfully.

The FortiGate now has a basic secure administration and recovery process.

The lab demonstrates:

- Secure management access
- Trusted Host restriction
- Session timeout
- Configuration backup
- Encrypted backup
- Recovery after an authentication configuration problem
- Basic operational recovery planning

---

## Lessons Learned

1. Administrative security is part of firewall management, not an optional extra.
2. Trusted Hosts reduce the systems that can access the administrator account.
3. Backup encryption protects the configuration backup.
4. Password Mask and backup encryption have different purposes.
5. A backup should be created before risky configuration changes.
6. Recovery access must be planned before enabling strong authentication controls.
7. A successful backup is not enough; the administrator must understand how recovery will be performed.
