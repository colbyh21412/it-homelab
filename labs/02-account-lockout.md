# Lab 02 - Account Lockout Policy
Date: June 2026
Platform: VirtualBox
Environment: lab.local domain (built in Lab 01)


## Objective
Configure an account lockout policy via Group Policy, test it by triggering a lockout, and resolve it from the Domain Controller the way a help desk technician would.

## Environment
| Machine | Role | IP Address |
| :--- | :--- | :--- |
| DCO1    | Domain Controller | 192.168.1.10 |
| CLIENT01 | Domain-joined Workstation | 192.168.1.20 |

## What I did
Step 1 - Configured Account Lockout Policy via GPO
  •  Opened Group Policy Management on DC01
  •  Expanded Forest: lab.local -> Domains -> lab.local
  •  Right-clicked Default Domain Policy -> Edit
  •  Navigated to: Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Account Lockout Policy
  •  Configured the following settings:
  | Setting | Value |
  | :--- | :--- |
  | Account lockout threshold | 3 invalid logon attempts |
  | Account lockout duration | Domain-joined Workstation |
  | Reset account lockout counter after | 30 minutes |

Step 2 - Applied the Policy to CLIENT01
  • Ran gpupdate /force on CLIENT01 to apply the new policy immediately

Step 3 - Triggered a Lockout
  • Signed out of CLIENT01
  • Attempted to log in as LAB|ajohnson with the wrong password 3 times in a row
  • After the 3rd failed attempt, I received the following error: "The referenced account is currently locked out and may not be logged on to"
  
Step 4 - Unlocked the Account from DC01
  • Opened Active Directory Users and Computers on DC01
  • Navigated to _Branches -> Houston -> Users
  • First attempted to unlock via the Account tab:
    • Double-clicked 'ajohnson' -> Account tab
    • The unlock checkbox was visible but did not work correctly
  • Used Reset Password as an alternative:
    • Right-clicked ajohnson -> Reset Password
    • Entered a new password
    • Checked "Unlock the user's account" <- this checkbox only appears when the account is currently locked
  • Clicked OK
  
Step 5 - Verified Login
  • Returned to CLIENT01
  • Logged in as LAB\ajohnson with the new password
  • Login succeeded, confirming the account was successfully unlocked

## Issues Encountered and How I Fixed Them
### Issue 1 - Unlock account checkbox not working in ADUC
   • Cause: The Account tab showed the unlock checkbox, but unchecking it did not unlock the account
   • Fix: Used Right-click -> Reset Password instead, which includes an **Unlock the user's account** that only appears when the account is actively locked. This method worked correctly
   • **Note:** The Reset Password unlock checkbox is always visible on the layout, but it will automatically check itself and become active only when the Domain Controller recognizes that the account is currently locked

## key Commands Used
  Group Policy:
  gpupdate /force         # Force immediate Group Policy update on Client
  gpresult /r             # View applied Group Policies on current machine

## What I learned
  • How to configure an account lockout policy via Group Policy
  • How account lockout threshold, duration, and reset count work together
  • How to identify a locked out account in Active Directory Users and Computers
  • How to unlock a user account from the Domain Controller
  • How to use password reset with account unlock as an alternative method
  • Account lockouts are the most common help desk ticket in any organization
  • gpupdate /force is required to push new Group Policy changes to client machines immediately

## Real World Application
Account lockout issues are one of the most frequent help desk tickets in any organization. A user enters their password incorrectly too many times and gets locked out. The help desk tech needs to:

  • Verify the user's identity
  • Locate the account in Active Directory
  • Unlock the account or reset the password
  • Document the ticket resolution
