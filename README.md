# Active Directory Home Lab

A virtual Windows domain built in Oracle VirtualBox to practice Tier 1 help desk work: account management, Group Policy, file share permissions, and troubleshooting. Every task was then documented as a ticket in Jira Service Management, from customer request to resolution.

## Lab Environment

| Machine | OS | Role |
|---|---|---|
| DC | Windows Server 2019 | Domain controller for mydomain.com running Active Directory, DNS, DHCP, and NAT routing |
| CLIENT1 | Windows 10 | Domain joined workstation used to test every change as an end user |

The DC has two network adapters: one connected to the internet through VirtualBox NAT, and one on an internal network shared with CLIENT1. CLIENT1 gets its IP address from the DC's DHCP server and reaches the internet through the DC. A PowerShell script populated the domain with 1,000 test users in the _USERS OU.

**Tools used:** Oracle VirtualBox, Active Directory Users and Computers, Group Policy Management, PowerShell, Command Prompt (ipconfig, nslookup, ping), Jira Service Management

## Help Desk Tasks

### 1. Onboarding a New Hire
Created the account for HelpDesk User in ADUC with a temporary password and "User must change password at next logon" enabled. Signed in on CLIENT1 and confirmed the forced password change.

![New user created in ADUC](03_user_created.png)

### 2. Password Reset
Located the user in ADUC using Find, reset the password, and required a new password at next logon. Confirmed the user could sign in and set a new password.

![Password reset in ADUC](10_password_reset_done.png)

### 3. Account Lockout Policy
Configured the Default Domain Policy to lock an account after 3 failed sign in attempts for 10 minutes. Ran gpupdate and confirmed the account locked after the third failed attempt.

![Lockout policy settings](18_lockout_settings_after.png)

### 4. Unlocking an Account
Unlocked the account from the Account tab in ADUC and confirmed the user could sign in again.

![Unlocking the account](23_unlock_account.png)

### 5. Blocking Control Panel with Group Policy
Created a GPO that prohibits access to Control Panel and linked it to the _EMPLOYEES OU. The policy did not apply at first because HelpDesk User was still in the default Users container, which a GPO cannot be linked to. After moving the user into _EMPLOYEES, the restriction applied on CLIENT1.

![Moving the user into the _EMPLOYEES OU](26_move_user_to_employees_ou.png)
![Restriction message on CLIENT1](34_restrictions_message.png)

### 6. Shared Folder Access with a Security Group
Created a Marketing security group, added HelpDesk User, and shared a Marketing folder with Modify permissions granted to the group rather than to individual users. Confirmed access by creating a test file from CLIENT1.

![Share permissions for the Marketing group](39_share_permissions.png)
![Test file created by a Marketing member](43_test_txt_created.png)

### 7. Offboarding
Disabled the departing user's account instead of deleting it, which preserves group memberships and file ownership for auditing. Confirmed the sign in was blocked on CLIENT1.

![Disabled account blocked at sign in](47_disabled_login_blocked.png)

## Troubleshooting

### Guest Additions install failed for a standard user
**Issue:** CLIENT1 had display problems because VirtualBox Guest Additions were missing, and the installer failed when run as a standard domain user.
**Cause:** Installing drivers requires administrator rights.
**Fix:** Ran the installer with Run as administrator using domain admin credentials. The install completed and the display issue was resolved.

### Marketing folder open to all domain users
**Issue:** While testing the Marketing share, I found the folder was inheriting a broad Users permission from C:\, which would have let every domain user open it.
**Cause:** NTFS permission inheritance from the parent drive.
**Fix:** Disabled inheritance on the Marketing folder and removed the inherited Users entries, leaving access only to the Marketing group and administrators. Signed in as a domain user outside Marketing and confirmed access was denied.

![Inheritance disabled on the Marketing folder](41_inheritance_removed.png)
![Access denied for a user outside Marketing](45_access_denied.png)

## Ticket Documentation in Jira Service Management

All 12 tasks and incidents were logged in Jira Service Management: 7 service requests and 5 incidents. Each ticket was submitted through the customer portal, assigned, worked with an internal note documenting the steps and screenshots, answered with a customer facing reply, and resolved within SLA. The permissions incident is linked to the original folder access request, and the three network incidents (SUP-10 to SUP-12) have internal notes structured around the six step troubleshooting methodology.

![Customer request form](jira_01_request_form.png)
![Resolved ticket with SLAs and internal notes](jira_03_ticket_resolved.png)
![Customer view of the resolved request](jira_04_customer_resolved.png)
![All 12 tickets resolved](jira_05_all_tickets_resolved.png)

## Network Troubleshooting

I deliberately broke three things on CLIENT1, then diagnosed and fixed each one using the CompTIA six step troubleshooting methodology. Each fix was logged as a Jira ticket (SUP-10 to SUP-12). VirtualBox snapshots were taken first as a rollback option.

### Scenario 1: Wrong DNS server (SUP-10)

**Symptom:** User could not reach mydomain.com or shared drives by name. **Diagnosis:** Ping to the DC by IP worked, but nslookup timed out. ipconfig /all showed the DNS server set to 172.16.0.99 instead of the DC. **Fix:** Restored automatic DNS so the client receives the DC through DHCP.

![DNS broken](50_dns_broken.png)
![DNS fixed](51_dns_fixed.png)

### Scenario 2: DHCP service stopped (SUP-11)

**Symptom:** No network access at all. **Diagnosis:** ipconfig /renew could not reach a DHCP server, and the client assigned itself a 169.254 address with no gateway. The DHCP Server service on the DC was stopped. **Fix:** Started the DHCP Server service and renewed the client lease.

![DHCP broken](52_dhcp_broken.png)
![DHCP fixed](53_dhcp_fixed.png)

### Scenario 3: Network adapter disabled (SUP-12)

**Symptom:** Sudden loss of all network access. **Diagnosis:** ipconfig returned no adapter information at all. Network Connections showed the Ethernet adapter disabled. **Fix:** Enabled the adapter.

![Adapter broken](54_adapter_broken.png)
![Adapter fixed](55_adapter_fixed.png)

### Quick reference

| Symptom | Likely cause |
|---|---|
| Ping by IP works, names fail | DNS |
| Address starts with 169.254 | DHCP unreachable |
| ipconfig shows no adapter | Adapter disabled or disconnected |

## Skills Demonstrated

Active Directory user and account management, Group Policy creation and linking, OU structure and how it affects policy scope, share and NTFS permissions, least privilege access through security groups, secure offboarding, network troubleshooting of DNS, DHCP, and adapter faults with ipconfig, nslookup, and ping, structured troubleshooting using the CompTIA methodology, and ticket documentation in an ITSM tool.


