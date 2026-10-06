# 12 - Web Filtering

## Scenario

The company wants to control and monitor web access for users in the Users network.

The FortiGate should:

- Block dangerous or restricted web categories.
- Monitor selected categories.
- Warn users when accessing warning-based categories.
- Allow normal web traffic.
- Log web activity for investigation.
- Support custom URL filtering.

## Objective

Configure and verify FortiGate Web Filtering for the Users Internet policy.

The main goal is to understand how FortiGuard Web Filtering, Static URL Filtering, logging, and warning actions work in a real firewall environment.

## Environment

- FortiGate: `FGT-HQ-01`
- Users network: `10.10.10.0/24`
- Users interface: `VLAN10-USERS`
- Internet interface: `port1`
- Firewall policy: `POL-VLAN10-USERS-TO-INTERNET`
- Web Filter profile: `WF-USERS-INTERNET`
- Test client: `10.10.10.101`

## Configuration

### Web Filter Profile

Profile:

`WF-USERS-INTERNET`

Feature set:

`Flow`

FortiGuard Category Based Filter was configured with different actions:

- Selected dangerous categories: `Block`
- Selected categories: `Monitor`
- Selected categories: `Warning`
- Logging enabled for the configured FortiGuard categories.

Examples:

- Child Sexual Abuse → Block
- Terrorism → Block
- Dating → Warning
- Other selected potentially liable categories → Monitor

### Safe Search

Safe Search enforcement was enabled for supported search engines.

### Static URL Filter

A Static URL Filter was configured to demonstrate custom URL control.

Example:

- `www.youtube.com`
- Action: `Monitor`

This demonstrates that administrators can apply custom rules in addition to FortiGuard category-based filtering.

### URL Logging

The Web Filter profile was configured with:

`log-all-url enable`

This allows normal allowed web requests to appear in Web Filter logs.

Other web logging options were also enabled by the profile.

## Firewall Policy

The existing Users Internet policy was updated with the Web Filter profile:

`POL-VLAN10-USERS-TO-INTERNET`

Important security profile configuration:

- UTM status: Enabled
- Web Filter: `WF-USERS-INTERNET`
- SSL/SSH inspection: Certificate Inspection
- Traffic logging: All
- NAT: Enabled
<img width="1635" height="856" alt="image" src="https://github.com/user-attachments/assets/9ed8930d-7efe-4ffd-8c75-100bcde4ae41" />

## Verification

### 1. Blocked Website

A website classified by FortiGuard as a blocked category was tested.

Example:

`eg1xbet.com`

Result:

- Category: Gambling
- Action: Blocked
- Web Filter event appeared in Security Events.

This confirmed that FortiGuard category filtering was working.

### 2. Allowed Website

Normal websites were accessed after enabling URL logging.

Example:

`google.com`

The Web Filter log showed:

- Action: Passthrough
- Event Type: `ftgd_allow`

This confirmed that allowed URLs can also be logged when `log-all-url` is enabled.

### 3. Warning Category

A Dating category website was tested.

Example:

`tinder.com`

The log showed:

- Category ID: `15`
- Category: Dating
- Security Level: Warning
- Message: URL belongs to a category with warnings enabled
- Event Type: `ftgd_blk`
- Action: Blocked

Important lesson:

The Web Filter GUI Action filter does not use `Warning` as a separate Action value.

A Warning event can therefore appear with:

`Action = Blocked`

The Event Type, Log ID, Security Level, and Message must be checked to understand the real event.

### 4. Web Filter Log Investigation

Important Web Filter log fields include:

- Source IP
- Destination IP
- Hostname
- URL
- Category ID
- Category
- Action
- Security Level
- Profile
- Event Type
- Log ID
- Rating Method

Example:

`ftgd_allow` indicates an allowed FortiGuard web filtering event.

The log details are more useful than filtering only by the Action field.

## Troubleshooting

### Problem

Allowed URLs were not appearing in Web Filter Security Events.

### Investigation

The Web Filter profile was checked using:

`show full-configuration webfilter profile WF-USERS-INTERNET`

The following configuration was found:

`set log-all-url disable`

### Fix

The setting was changed to:

`set log-all-url enable`

After this change, allowed URLs such as Google and YouTube-related domains appeared in the Web Filter logs.

### Important Lesson

Global Event Logging was already configured as `All`.

The problem was inside the Web Filter profile itself.

This demonstrates the troubleshooting process:

Problem → Evidence → Hypothesis → Test → Fix → Verify

## Important Concepts Learned

### Action vs Event Type vs Severity

These fields must not be confused.

Example:

`Action = Blocked`

does not always mean the FortiGuard category itself was configured as a normal Block rule.

For a Warning category, the log can show:

- Security Level: Warning
- Event Type: `ftgd_blk`
- Message: category with warnings enabled
- Action: Blocked

Therefore, investigate the complete log entry instead of looking only at Action.

### FortiGuard Category Filtering

FortiGate can use FortiGuard web categories to classify websites and apply actions such as:

- Block
- Monitor
- Warning

### Static URL Filtering

Static URL filtering allows administrators to create custom URL rules instead of relying only on FortiGuard categories.

### URL Logging

Logging all URLs provides better visibility for investigation, but it can create a large amount of log data in a real environment.

It should therefore be enabled based on operational and storage requirements.
<img width="1919" height="855" alt="image" src="https://github.com/user-attachments/assets/b245d2b6-bef7-4869-a8a1-9908e29a1409" />
<img width="1906" height="860" alt="image" src="https://github.com/user-attachments/assets/6242b982-f16a-483c-a603-6a53e93afd57" />

## Security Considerations

- Block high-risk categories that are not required for business use.
- Use Monitor for categories that need visibility before enforcing a block.
- Use Warning when users should receive a warning before accessing certain categories.
- Keep web filtering logs available for investigation.
- Avoid enabling excessive logging on every policy without a reason.
- Use SSL inspection carefully when deeper HTTPS inspection is required.
- Review FortiGuard categorization before creating many custom URL rules.

## Result

Web Filtering was successfully configured and verified on the Users Internet policy.

The lab demonstrated:

- FortiGuard category filtering
- Block actions
- Monitor/Passthrough behavior
- Warning categories
- Static URL filtering
- Safe Search
- URL logging
- Web Filter log investigation
- Troubleshooting of missing allowed-URL logs

Task 12 is complete.

## Next Task

Task 13 - Application Control
