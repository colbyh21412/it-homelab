# Lab 10 — Printer Deployment via Logon Script

**Date:** June 2026

**Platform:** VirtualBox

**Environment:** lab.local domain (built in Lab 01)

---

## Objective

Share a printer from DC01 and automatically deploy it to client workstations using a domain logon script, simulating how printers are commonly deployed in small to medium business environments.

---

## Environment

| Machine | Role | IP Address |
|---|---|---|
| DC01 | Domain Controller / Print Server | 192.168.1.10 (static) |
| CLIENT01 | Domain-joined workstation | 192.168.1.20 (static) |

---

## What I Did

### Step 1 — Created a Shared Printer on DC01

- Opened Settings → Bluetooth & devices → Printers & scanners
- Added a printer manually using **Add a printer using a TCP/IP address or hostname**
- Selected driver: Microsoft PCL6 Class Driver
- Named the printer: `Lab Network Printer`
- Enabled sharing with share name: `LabPrinter`

### Step 2 — Fixed Printer Port Configuration

The printer was initially created pointing to a placeholder IP address (`192.168.1.99`), which caused connection errors when other machines tried to use the shared printer.

**Fix:**
- Opened Lab Network Printer → Properties → Ports tab
- Removed the invalid IP-based port
- Added and selected the **FILE:** port (a local virtual port, similar to how Microsoft Print to PDF operates)
- Applied changes

This resolved the "Windows couldn't connect to the printer" error that occurred when attempting to use the shared printer from another machine.

### Step 3 — Created a Logon Script to Map the Printer

Created/edited `logon.bat` in:
```
C:\Windows\SYSVOL\sysvol\lab.local\scripts
```

**Script content:**
```batch
@echo off
echo Welcome to the lab.local domain, %USERNAME%!
rundll32 printui.dll,PrintUIEntry /in /n "\\192.168.1.10\LabPrinter"
```

The `rundll32 printui.dll,PrintUIEntry /in /n` command installs a network printer connection using the UNC path to the shared printer.

### Step 4 — Assigned the Logon Script

- Opened Active Directory Users and Computers
- Navigated to ajohnson's Profile tab
- Confirmed Logon script field was set to `logon.bat`

### Step 5 — Tested and Debugged the Deployment

Initial testing showed the printer was not appearing for ajohnson after login, despite the manual command working correctly when run as Administrator.

**Debugging process:**
1. Added a test line to the script to write a file to the user's desktop, confirming whether the script ran at all
2. Removed `@echo off` and added `pause` to make the script visible during execution
3. Discovered the CMD window briefly flashed and closed without executing — indicating the script was being blocked

### Step 6 — Discovered and Resolved a GPO Conflict

**Root cause identified:**

The **Restrict Standard Users** GPO (created in Lab 03) had the setting:

```
User Configuration → Policies → Administrative Templates → System →
Prevent access to the command prompt
  └── "Disable the command prompt script processing also?" = Yes
```

This setting blocks ALL `.bat` and `.cmd` script execution for restricted users — including logon scripts — not just interactive Command Prompt access.

**Fix:**
- Changed "Disable the command prompt script processing also?" from **Yes** to **No**
- Ran `gpupdate /force` on CLIENT01
- Signed out and back in as ajohnson

### Step 7 — Verified Successful Deployment

- Signed in as `LAB\ajohnson` on CLIENT01
- Logon script executed visibly, mapping the printer
- Opened the Printers folder using:
```
shell:::{2227A280-3AEA-1069-A2DE-08002B30309D}
```
- **Lab Network Printer** appeared in the list

### Step 8 — Cleaned Up the Script

Removed debug lines, restoring the script to its production form:

```batch
@echo off
echo Welcome to the lab.local domain, %USERNAME%!
rundll32 printui.dll,PrintUIEntry /in /n "\\192.168.1.10\LabPrinter"
```

---

## Issues Encountered and How I Fixed Them

### Issue 1 — "Windows couldn't connect to the printer" error

- **Cause:** Printer was configured with a TCP/IP port pointing to a non-existent IP address (192.168.1.99)
- **Fix:** Changed the printer port to the local **FILE:** port, which does not require network communication to a physical device

### Issue 2 — Additional Drivers option grayed out

- **Cause:** Related to the invalid printer port preventing Windows from validating the printer configuration
- **Fix:** Resolved automatically once the port was changed to FILE:

### Issue 3 — Logon script file saved with hidden .txt extension

- **Cause:** Notepad appended `.txt` to files named `logon.bat`, resulting in `logon.bat.txt`, which Group Policy did not recognize as the assigned script
- **Fix:** Used `Get-ChildItem` to identify the actual filenames, then `Rename-Item` to correct the extension:

```powershell
Rename-Item "C:\Windows\SYSVOL\sysvol\lab.local\scripts\logon.bat.txt" "logon.bat"
```

### Issue 4 — Logon script not executing for standard users (Major Finding)

- **Cause:** A previously configured GPO setting — "Disable the command prompt script processing also?" set to Yes — silently prevented all batch script execution for users affected by the Restrict Standard Users policy, including logon scripts assigned via Active Directory
- **Symptoms:** No errors were generated. The CMD window briefly flashed during login and closed immediately without running any commands
- **Fix:** Changed the setting to **No** in Group Policy Management, then ran `gpupdate /force` and re-tested

This was discovered through systematic debugging: first confirming the script was assigned correctly, then confirming it was attempting to run, then identifying that script *processing* itself was being blocked at the policy level — separate from Command Prompt *access*.

---

## Key Commands Used

```
rundll32 printui.dll,PrintUIEntry /in /n "\\192.168.1.10\LabPrinter"   # Map a network printer via command line
shell:::{2227A280-3AEA-1069-A2DE-08002B30309D}                          # Open Printers folder directly via File Explorer
gpupdate /force                                                          # Apply Group Policy changes immediately
```

```powershell
Get-ChildItem "C:\Windows\SYSVOL\sysvol\lab.local\scripts"               # List files in the scripts folder
Rename-Item "logon.bat.txt" "logon.bat"                                  # Fix incorrect file extension
Get-Content "C:\Windows\SYSVOL\sysvol\lab.local\scripts\logon.bat"       # View script contents
```

---

## What I Learned

- How to share a printer on a Windows Server and configure it for network access
- How printer ports work and why the FILE: port is useful for virtual/lab printers
- How to deploy a network printer connection via logon script using rundll32 and printui.dll
- How to assign a logon script to a user account in Active Directory
- The importance of verifying file extensions when creating scripts in SYSVOL
- How GPO settings can have unintended side effects across unrelated functionality
- How "Disable the command prompt script processing also?" affects ALL batch scripts, not just interactive CMD access
- A systematic debugging approach: confirm assignment → confirm execution → confirm what is blocking execution
- How to use shell folder shortcuts to access system folders when Control Panel access is restricted

---

## Real World Application

Printer deployment is one of the most common help desk tasks in any office environment. While modern environments often use Group Policy Preferences or print management consoles for this, logon scripts remain widely used in smaller organizations and legacy environments.

This lab also demonstrates a critical real-world skill: **diagnosing GPO conflicts**. In real corporate environments, multiple GPOs are often layered together over time, and security policies can have unintended consequences on unrelated functionality — such as a "block command prompt" policy silently breaking all logon scripts. Being able to methodically isolate which policy is causing an issue, without simply disabling everything, is a valuable troubleshooting skill that goes beyond entry level.

---

## Lab Series Summary

This completes a 10-lab Active Directory home lab series covering:

| # | Lab |
|---|---|
| 01 | Active Directory Domain Setup |
| 02 | Account Lockout Policy |
| 03 | Group Policy Restrictions |
| 04 | DHCP Setup |
| 05 | Shared Folders and Permissions |
| 06 | Event Viewer Investigation |
| 07 | Bulk User Creation |
| 08 | User Offboarding |
| 09 | RDP Troubleshooting |
| 10 | Printer Deployment via Logon Script |

Together, these labs simulate the core responsibilities of an entry-level IT support role within a small business Active Directory environment, including a help desk ticketing system (osTicket) for tracking and resolving issues.
