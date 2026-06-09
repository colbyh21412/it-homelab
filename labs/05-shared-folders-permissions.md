# Lab 05 - Shared Folders and Permissions

Date: June 2026

Platform: VirtualBox

Environment: lab.local domain (built in Lab 01)

## Objective

Create shared network folders on DC01 and configure NTFS permissions so that only the correct security group can access each department folder. Test access from CLIENT01 as different domain users to verify permissions are working correctly.

## Environment

| Machine | Role | IP Address |
| --- | --- | --- |
| DC01 | Domain Controller/File Server | 192.168.1.10 (static) |
| CLIENT01 | Domain-joined workstation | 192.168.1.20 (static) |

## Folder Structure Created
C:\Shares\

├── Helpdesk\       ← Helpdesk group only

├── Accounting\     ← Accounting group only

└── ITSupport\      ← ITSupport group only

## What I did
  ### Step 1 - Created the Folder Structure
  
    • Opened File Explorer on DC01
    
    • Navigated to C: drive
    
    • Created a new folder called Shares
    
    • Inside Shares created three subfolders: Helpdesk, Accounting, ITSupport
    
  ### Step 2 - Shared the Shares Folder
  
    • Right-clicked C:\Shares → Properties → Sharing tab → Advanced Sharing
    
    • Checked Share this folder
    
    • Share name: Shares
    
    • Clicked Permissions
    
    • Removed Everyone from the permissions list
    
    • Added Domain Users with Read permission
    
    • Clicked OK → OK → Close
    
  ### Step 3 - Set NTFS Permissions on Each Subfolder

    For Each subfolder, inheritance was disabled and permissions were set explicitly:
    
      1. Right-clicked folder → Properties → Security tab → Advanced
      
      2. Clicked Disable inheritance → Convert inherited permissions into explicit permissions
      
      3. Removed Users from the permissions list
      
      4. Added the corresponding security group with Modify permission
      
      5. Verified Administrators and SYSTEM have Full Control

**Final permissions for each folder:**

  | Folder | Group | Permission |
  | --- | --- | --- |
  | Helpdesk | Helpdesk | Modify
  | Helpdesk | Administrators | Full Control
  | Helpdesk | SYSTEM | Full Control
  | Accounting | Accounting | Modify
  | Accounting | Administrators | Full Control
  | Accounting | SYSTEM | Full Control
  | ITSupport | ITSupport | Modify
  | ITSupport | Administrators | Full Control
  | ITSupport | SYSTEM | Full Control
  
  ### Step 4 - Tested Access from CLIENT01

  **Accessed shared folders from CLIENT01 by navigating to:**

  \\192.168.1.10\Shares

  Results:

  | User | Group | Helpdesk | Accounting | ITSupport |
  | --- | --- | --- | --- | --- |
  | ajohnson | Helpdesk | Access | Denied | Denied |
  | bmartinez | Accounting | Denied | Access | Denied |
  | cwalker | ITSupport | Denied | Denied | Access |

  All permissions working correctly - each user can only access their department folder.

## Share vs NTFS Permissions

## Issues Encountered and How I Fixed Them

Issue 1 — All users could access all three folders initially

**Cause:** The Users group was still listed in the NTFS permissions due to inherited permissions from the parent folder

**Fix:** Disabled inheritance on each subfolder and converted to explicit permissions, then removed the Users group from each folder

Issue 2 — Could not remove Users group without disabling inheritance first

**Cause:** "You can't remove Users because this object is inheriting permissions from its parent" error

**Fix:** Used Advanced Security Settings → Disable inheritance → Convert inherited permissions into explicit permissions, then removed Users group

Issue 3 — Accounting group accidentally added to ITSupport folder

**Cause:** Wrong group added during permissions setup

**Fix:** Opened ITSupport folder Security tab → removed Accounting group

Issue 4 — CLIENT01 kept losing network connectivity and reverting to APIPA address

**Cause:** Network adapter instability with DHCP on Bridged Adapter

**Fix:** Set CLIENT01 back to static IP (192.168.1.20) with DNS pointing to DC01 (192.168.1.10) for stability

Issue 5 — cwalker could not log in due to domain connectivity loss

**Cause:** CLIENT01 lost domain connectivity after network adapter issues

**Fix:** Reset network adapter settings, confirmed ping to DC01 succeeded, then domain login worked correctly

## Key Commands and Paths used

## What I learned

  • How to create and share folders over the network using UNC paths
  
  • The difference between Share permissions and NTFS permissions
  
  • How inherited permissions work and when to disable them
  
  • How to use security groups to control folder access
  
  • How the most restrictive permission between Share and NTFS always wins
  
  • How to troubleshoot permission issues by checking both Share and NTFS layers
  
  • How to verify access by testing with multiple user accounts
  
  • Why removing inherited permissions is necessary before setting custom NTFS permissions
  
## Real World Applications

Shared folder permissions are fundamental to IT administration. In a real help desk or junior sysadmin role, you will regularly:

  • Create department shares for HR, Finance, IT, and other teams
  
  • Troubleshoot "access denied" errors by checking Share and NTFS permissions
  
  • Add users to security groups to grant access to shared resources

  • Audit folder permissions to ensure sensitive data is properly protected

  This lab simulates a realistic department file server setup where each team can only access their own folder, which is standard practice in corporate environments.
  
## Next Steps
  • Lab 06 - Event Viewer Investigation
