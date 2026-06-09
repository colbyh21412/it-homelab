# Lab 06 — Event Viewer Investigation

**Date:** June 2026

**Platform:** VirtualBox

**Environment:** lab.local domain (built in Lab 01)

---

## Objective

Use Windows Event Viewer to investigate security events on the domain controller. Practice filtering logs by Event ID to identify failed logins, account lockouts, and account management changes the way a real help desk technician would.

---

## Environment

| Machine | Role | IP Address |
|---|---|---|
| DC01 | Domain Controller | 192.168.1.10 (static) |
| CLIENT01 | Domain-joined workstation | 192.168.1.20 (static) |

---

## What is Event Viewer?

Event Viewer is a Windows tool that records system, application, and security events in logs. For IT support and security, the **Security log** is the most important — it captures every login attempt, account change, and privilege use on the machine.

On a Domain Controller the Security log captures events from across the entire domain, making it the first place to look when investigating user issues.

---

## Key Event IDs Reference

| Event ID | Description | When to Use |
|---|---|---|
| 4624 | Successful logon | Verify a user logged in successfully |
| 4625 | Failed logon | Investigate wrong password attempts |
| 4740 | Account locked out | Find which machine is causing lockouts |
| 4720 | User account created | Audit new account creation |
| 4726 | User account deleted | Audit account deletions |
| 4732 | User added to security group | Track group membership changes |

---

## Logon Types Reference

| Type | Description |
|---|---|
| 2 | Interactive — user logged in at the physical machine |
| 3 | Network — accessing a shared resource over the network |
| 4 | Batch — scheduled task ran |
| 5 | Service — a Windows service started |
| 7 | Unlock — user unlocked a locked workstation |
| 10 | Remote Interactive — RDP login |

---

## What I Did

### Step 1 — Opened Event Viewer and Navigated to Security Log

- Opened Event Viewer on DC01 (Start → search Event Viewer)
- Expanded **Windows Logs → Security**
- Observed the large volume of security events recorded automatically

### Step 2 — Investigated Failed Login Attempts (Event ID 4625)

- Clicked **Filter Current Log** in the Actions pane
- Filtered by Event ID `4625`
- Found failed login events showing:

| Field | Value |
|---|---|
| Account Name | ajohnson |
| Failure Reason | Unknown user name or bad password |
| Workstation Name | WIN-8KOVU5CPQ5A (CLIENT01) |

This confirmed failed login attempts from CLIENT01 during our account lockout testing in Lab 02.

### Step 3 — Investigated Successful Logins (Event ID 4624)

- Filtered Security log by Event ID `4624`
- Identified the following Logon Types in the log:

| Logon Type | Description | Example Found |
|---|---|---|
| 3 | Network logon | Accessing shared folders |
| 5 | Service logon | Windows services starting |
| 7 | Unlock | Administrator unlocking DC01 |

### Step 4 — Investigated Account Lockout Events (Event ID 4740)

- Filtered by Event ID `4740`
- Found lockout event showing:

| Field | Value |
|---|---|
| Account Name | Administrator |
| Caller Computer Name | WIN-8KOVU5CPQ5A (CLIENT01) |

This shows exactly which machine caused the lockout — critical information for resolving lockout tickets in a real environment.

### Step 5 — Investigated Account Creation Events (Event ID 4720)

- Filtered by Event ID `4720`
- Found system generated account creation:

| Field | Value |
|---|---|
| Account Name | WDAGUtilityAccount |
| Created By | MINWINPC$ |

Note: WDAGUtilityAccount is automatically created by Windows Defender Application Guard — this is a normal system event.

### Step 6 — Investigated Group Membership Changes (Event ID 4732)

- Filtered by Event ID `4732`
- Found group membership change showing:

| Field | Value |
|---|---|
| Subject Account Name | Administrator |
| Group Name | Users |

This shows Administrator adding an account to the Users group during domain setup.

### Step 7 — Enabled Advanced Audit Policy on CLIENT01

- Opened Group Policy Management on DC01
- Edited Restrict Standard Users GPO
- Navigated to:

```
Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Audit Policy
```

- Enabled **Audit logon events** for both **Success** and **Failure**
- Ran `gpupdate /force` on CLIENT01 to apply

---

## Real World Help Desk Scenario — Lockout Investigation

When a user calls saying they are locked out, here is the Event Viewer investigation workflow:

```
1. Open Event Viewer on DC01
2. Navigate to Windows Logs → Security
3. Filter by Event ID 4740
4. Find the user's account name in the results
5. Note the Caller Computer Name — this is the machine causing lockouts
6. Go to that machine and check for:
   - Saved credentials in Credential Manager
   - Mapped network drives using old password
   - Mobile device with synced account using old password
   - Background applications authenticating with cached credentials
7. Clear old credentials and unlock the account in ADUC
8. Document the resolution in the help desk ticket
```

This workflow applies to real corporate environments and is a standard help desk procedure.

---

## Issues Encountered and How I Fixed Them

### Issue 1 — Could not find Type 2 interactive logon events in Security log

- **Cause:** Domain Controllers handle interactive logons differently than workstations and Type 2 events were not appearing in the standard filter
- **Fix:** Used custom XML query to filter by both Event ID and Logon Type. Also checked CLIENT01's Event Viewer where interactive logons are recorded

### Issue 2 — Audit policy not capturing all logon events on CLIENT01

- **Cause:** Default audit policy was not configured to capture all logon event types
- **Fix:** Enabled Audit logon events for Success and Failure in both Group Policy and Local Group Policy on CLIENT01

---

## Key Commands and Paths Used

### Event Viewer Navigation

```
eventvwr                    # Open Event Viewer from Run dialog
Windows Logs → Security     # Location of security events
Filter Current Log          # Filter events by Event ID
```

### Audit Policy Location

```
Computer Configuration → Policies → Windows Settings → 
Security Settings → Local Policies → Audit Policy → 
Audit logon events
```

---

## What I Learned

- How to navigate Windows Event Viewer and filter Security logs by Event ID
- The most important security Event IDs for help desk work
- The different Windows logon types and what each one means
- How Event ID 4740 identifies which machine is causing account lockouts
- How to use custom XML queries to filter events by multiple criteria
- How audit policy controls which events get recorded in the Security log
- How Domain Controllers capture domain-wide security events
- How Event Viewer is the first tool to use when investigating security incidents

---

## Real World Application

Event Viewer is used daily in IT support and security roles:

- **Help desk:** Investigating account lockouts, failed logins, and access issues
- **Sysadmin:** Auditing account changes and group membership modifications
- **Security:** Detecting brute force attacks, unauthorized access attempts, and suspicious activity

Being able to quickly filter and read Event Viewer logs is a core skill that separates entry-level techs from more experienced ones and is commonly tested in IT interviews.

---

## Next Steps

- Lab 07 — Bulk User Creation
