# Lab 03 — Group Policy Restrictions
**Date:** June 2026

**Platform:** VirtualBox

**Environment:** lab.local domain (built in Lab 01)

## Objective
Configure Group Policy Objects (GPOs) to restrict standard user access to specific system tools and resources on CLIENT01, while ensuring Domain Admins retain full access.

## Environment
| Machine | Role | IP Address |
| :--- | :--- | :--- |
| DCO1    | Domain Controller | 192.168.1.10 |
| CLIENT01 | Domain-joined Workstation | 192.168.1.20 |

## GPO Created
**Name:** Restrict Standard Users

**Linked to:** _Branches → Houston → Users OU

**Security Filtering:** Domain Users (standard users only — Domain Admins excluded)

## What I Did
### Step 1 — Created the Restrict Standard Users GPO

•  Opened Group Policy Management on DC01

•  Right-clicked _Branches → Houston → Users OU

•  Selected Create a GPO in this domain and link it here

•  Named it Restrict Standard Users

•  Set Security Filtering to Domain Users to exclude Domain Admins from the policy



### Step 2 — Blocked Control Panel and Windows Settings

**Navigated to:**
User Configuration → Policies → Administrative Templates → Control Panel

• Set Prohibit access to Control Panel and PC Settings to Enabled

Result: Standard users receive "This operation has been cancelled due to restrictions" when attempting to open Control Panel or Windows Settings

### Step 3 — Disabled Command Prompt for Standard Users
**Navigated to:**
> User Configuration → Policies → Administrative Templates → System

•  Set Prevent access to the command prompt to Enabled

•  Set "Disable the command prompt script processing also?" to Yes

Result: Standard users receive "The command prompt has been disabled by your administrator" when attempting to open CMD or run batch scripts
### Step 4 — Blocked USB and Removable Storage
**Navigated to:**
User Configuration → Policies → Administrative Templates → System → Removable Storage Access
•  Set All Removable Storage classes: Deny all access to Enabled

Result: Standard users cannot read from or write to USB drives or removable storage devices
### Step 5 — Tested All Restrictions on CLIENT01
•  Ran gpupdate /force on CLIENT01 to apply policies
•  Signed in as LAB\ajohnson and tested each restriction:

| Restriction | Result |
| :--- | :--- |
| Control Panel | Blocked |
| Windows Settings | Blocked |
| Command Prompt | Blocked |
| USB/Removable Storage | Blocked |

### Step 6 — AppLocker Configuration (Work in Progress)
Attempted to configure AppLocker to block standard users from running unauthorized executables outside of C:\Windows and C:\Program Files.
Steps completed:

•  Created default Executable Rules in AppLocker

•  Created a Deny rule for Authenticated Users with path *

•  Added exceptions for C:\Program Files\* and C:\Windows\*

•  Named rule Block Unauthorized Executables

•  Set enforcement mode to Enforce rules

•  Enabled and set Application Identity service to Automatic on CLIENT01

•  Linked GPO to Houston → Workstations OU for Computer Configuration to apply

Status: AppLocker rules are confirmed received by CLIENT01 via Get-AppLockerPolicy -Effective -Xml showing EnforcementMode="Enabled" for Exe rules, executables outside approved paths are still running. Further troubleshooting is required.

## Issues Encountered and How I Fixed Them
Issue 1 — GPO applying to Domain Admins and blocking Control Panel on DC01

**Cause:** Workstation-Hardening-Policy GPO created in a previous session with another tool was linked to the root of lab.local and applied to all authenticated users, including Administrator

**Fix:** Deleted the Workstation-Hardening-Policy GPO entirely. Cleared residual local group policy settings by running:

•  RD /S /Q "%WinDir%\System32\GroupPolicyUsers"
•  RD /S /Q "%WinDir%\System32\GroupPolicy"
•  gpupdate /force
•  Issue 2 — Restrict Standard Users GPO not applying to ajohnson

**Cause:** GPO was linked to the root of lab.local and Security Filtering only showed Domain Admins

**Fix:** Deleted the incorrect link, relinked GPO to _Branches → Houston → Users OU, and changed Security Filtering to Domain Users

Issue 3 — AppLocker not blocking unauthorized executables

**Cause:** Application Identity service was set to Manual and not running
**Status:** Still under investigation — AppLocker policy is being received by CLIENT01, but not fully enforcing. Will revisit in a future session.


## Key Commands Used
Group Policy
gpupdate /force                          # Force immediate GPO update on client
gpresult /r                              # View applied GPOs on current machine
Get-AppLockerPolicy -Effective -Xml      # View effective AppLocker policy
Get-Service AppIDSvc                     # Check Application Identity service status
Start-Service AppIDSvc                   # Start Application Identity service
Set-Service AppIDSvc -StartupType Automatic  # Set service to start automatically

## What I Learned

How to create and link GPOs to specific OUs to target only certain users
How to block Control Panel, Windows Settings, and Command Prompt via GPO
How to block USB and removable storage access via GPO
How Security Filtering controls which users a GPO applies to
How Computer Configuration policies require computer objects in the linked OU to apply correctly
How AppLocker requires the Application Identity (AppIDSvc) service to be running to enforce rules
How unintended GPOs from previous configurations can cause unexpected restrictions across the domain
The importance of always checking gpresult /r when troubleshooting unexpected policy behavior


## Real World Application
Group Policy restrictions are used in every corporate environment to:

Prevent users from changing system settings
Stop unauthorized software from running
Protect against data theft via USB drives
Reduce the attack surface on workstations

Help desk technicians need to understand GPOs because many user complaints ("I can't open Control Panel", "my USB drive isn't working") are caused by intentional policy restrictions rather than technical faults.

## Next Steps

Revisit AppLocker and resolve the enforcement issue
Lab 04 — DHCP Setup
