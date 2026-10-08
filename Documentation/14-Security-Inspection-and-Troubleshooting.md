# Task 14 — Security Inspection & Troubleshooting

## Objective

Configure and troubleshoot FortiGate IPS and SSL/SSH Inspection. The goal was to understand how security profiles inspect traffic, how traffic visibility affects inspection, and how to troubleshoot using logs and CLI diagnostics.

## Environment

- FortiGate: `FGT-HQ-01`
- Test client: `10.10.10.101`
- Users VLAN: `10.10.10.0/24`
- Internet interface: `port1`
- Main policy: `POL-VLAN10-USERS-TO-INTERNET`
- Web Filter: `WF-USERS-INTERNET`
- Application Control: `AC-USERS-INTERNET`

## Part 1 — IPS

### IPS Lab Sensor

Created:

`IPS-LAB-TEST`

Comment:

`IPS testing and detection lab for VLAN10 Internet traffic.`

Configuration:

- Critical severity filter
- Action: `Monitor`
- Packet Logging: `Disable`
- Status: `Enable`

The sensor was applied to `POL-VLAN10-USERS-TO-INTERNET`.
<img width="1872" height="907" alt="image" src="https://github.com/user-attachments/assets/3836826f-c799-45d7-9340-50005e185750" />

### IPS Verification

Normal Internet traffic did not generate visible IPS security events. Instead of assuming IPS was broken, the IPS engine was checked with:

```
diagnose ips signature hit 10
```

The output showed:

```
SIGNATURE PERFORMANCE: 4336 packets
```
<img width="1541" height="825" alt="image" src="https://github.com/user-attachments/assets/883a9424-ac55-45a5-b107-26a3140d2430" />

Several signatures had hits, including:

- Attack ID `35202` — 143 hits
- Attack ID `35366` — 143 hits
- Attack ID `50948` — 1059 hits
- Attack ID `50978` — 1059 hits

This confirmed that the IPS engine was processing traffic and performing signature matching.

Important: a signature hit counter is not the same as a confirmed attack or an IPS security event.

### EICAR Signature Investigation

The `Eicar.Virus.Test.File` signature was investigated.

Observed:

- Severity: Low
- Default Action: Pass
- Target: Server / Client
- OS: All

Because the lab sensor initially filtered only Critical signatures, EICAR was not included by that filter. A specific EICAR signature entry was added for controlled testing with Monitor action.

The HTTPS EICAR test was not used as the final IPS proof because the existing Certificate Inspection configuration does not provide full HTTPS payload visibility.

## Part 2 — Certificate Inspection

The main Internet policy used:

`certificate-inspection`

An HTTPS session from `10.10.10.101` was verified:

- Destination Port: `443`
- Service: `HTTPS`
- Application: `HTTPS.BROWSER`
- Application ID: `40568`
- Application Control: `AC-USERS-INTERNET`
- Policy: `POL-VLAN10-USERS-TO-INTERNET`
- Action: `accept`

The HTTPS session worked successfully.

Certificate Inspection provides TLS/certificate-level visibility without fully decrypting the HTTPS payload.

## Part 3 — Deep Inspection

The predefined FortiGate profile `deep-inspection` was inspected.

Important settings included:

- CA Certificate: `Fortinet_CA_SSL`
- HTTPS inspection
- SSH inspection
- SSL anomaly handling
- SSL exemptions

The predefined profile was not modified.

### Controlled Test Policy

The main policy was cloned as:

`POL-VLAN10-SSL-DEEP-TEST`

The test policy was restricted to:

`10.10.10.101`

SSL Inspection was changed from:

`certificate-inspection`

to:

`deep-inspection`

The existing Web Filter, Application Control, and IPS profiles remained enabled.

The test policy was placed above the general Internet policy so the test client matched it first.

### <img width="1919" height="768" alt="image" src="https://github.com/user-attachments/assets/111ab1d7-4043-45aa-9544-36135ca5d460" />
Deep Inspection Result

HTTPS traffic from `10.10.10.101` matched:

`Deep Inspection (12)`

The session used:

- Destination Port: `443`
- Service: `HTTPS`
- Application: `HTTPS.BROWSER`
- Action: `accept`

The Linux client then displayed:

`NET::ERR_CERT_AUTHORITY_INVALID`

The client did not trust the FortiGate CA:

`Fortinet_CA_SSL`

This proved that Deep Inspection was intercepting the HTTPS connection.

## Troubleshooting

The certificate error was analyzed using:

Problem → Evidence → Hypothesis → Test → Fix → Verify

Evidence showed that:

1. The client matched the Deep Inspection policy.
2. Deep Inspection was active.
3. FortiGate used `Fortinet_CA_SSL`.
4. The client did not trust that CA.
5. The browser generated a certificate authority error.

The correct production solution is to distribute and trust the inspection CA through controlled endpoint management, Group Policy, MDM, or another enterprise management method.

The test policy was disabled after verification, and normal HTTPS connectivity was restored.

The Deep Inspection policy was kept for future controlled lab testing.

## Key Lessons

1. IPS uses signatures and inspection engines to detect threats.
2. Monitor mode is useful before moving to Block.
3. A signature hit is not automatically a confirmed attack.
4. Security profiles must be attached to the correct firewall policy.
5. Certificate Inspection and Deep Inspection provide different levels of visibility.
6. Deep Inspection requires proper CA trust on managed clients.
7. HTTPS visibility affects what Web Filter, Application Control, and IPS can inspect.
8. Certificate errors after Deep Inspection can be caused by CA trust problems.
9. Do not disable certificate validation to solve SSL inspection problems.
10. Do not enable Deep Inspection for all users without controlled testing.
11. Troubleshooting should start with evidence instead of changing multiple settings at once.

## Final Configuration

### Main Internet Policy

`POL-VLAN10-USERS-TO-INTERNET`

- Web Filter: `WF-USERS-INTERNET`
- Application Control: `AC-USERS-INTERNET`
- IPS: `IPS-LAB-TEST`
- SSL Inspection: `certificate-inspection`

### Lab Deep Inspection Policy

`POL-VLAN10-SSL-DEEP-TEST`

- Scope: `10.10.10.101`
- SSL Inspection: `deep-inspection`
- Final state: Disabled

## Verification

Successfully verified:

- IPS sensor creation and policy attachment
- IPS engine processing and signature hit diagnostics
- HTTPS traffic with Certificate Inspection
- Deep Inspection policy matching
- FortiGate CA usage during Deep Inspection
- Client certificate trust failure
- Restoration of normal HTTPS after disabling the test policy

## Result

**Task 14 — COMPLETE**

The task demonstrated IPS, SSL/SSH Inspection, Certificate Inspection, Deep Inspection, certificate trust, security profile interaction, and practical troubleshooting.

The main skill gained was understanding the relationship between:

**Policy → Visibility → Inspection → Evidence → Troubleshooting → Verification**
