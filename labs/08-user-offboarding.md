# Lab 08 — User Offboarding

**Date:** June 2026

**Platform:** VirtualBox

**Environment:** lab.local domain (built in Lab 01)

---

## Objective

Simulate the offboarding process for an employee leaving the company. Follow a complete checklist to disable the account, remove access, document the change, and move the account to an appropriate OU for record keeping.

---

## Environment

| Machine | Role | IP Address |
|---|---|---|
| DC01 | Domain Controller | 192.168.1.10 (static) |

---

## Scenario

Chris Walker (cwalker), a member of the ITSupport security group, is leaving the company. IT needs to revoke his access while preserving his account for record-keeping and potential audit purposes.

---

## Offboarding Checklist

### Step 1 — Created a Disabled Accounts OU

- Opened Active Directory Users and Computers on DC01
- Right-clicked `lab.local` → New → Organizational Unit
- Named it `_Disabled Accounts`

This OU serves as a holding area for offboarded accounts, separate from active employee OUs.

### Step 2 — Disabled the User Account

- Navigated to `_Branches → _Houston → Users`
- Right-clicked **cwalker** → **Disable Account**

This immediately prevents the account from being used to log in anywhere on the domain.

### Step 3 — Reset the Password

- Right-clicked **cwalker** → **Reset Password**
- Set a new random password unknown to the former employee

This prevents the use of any cached or saved credentials even if the account were somehow re-enabled.

### Step 4 — Removed Security Group Memberships

Checked current group memberships:

```powershell
Get-ADPrincipalGroupMembership cwalker | Select-Object Name
```

**Output before removal:**
```
Domain Users
ITSupport
```

Removed from ITSupport:

```powershell
Remove-ADGroupMember -Identity "ITSupport" -Members "cwalker"
```

**Output after removal:**
```
Domain Users
```

Note: Domain Users is the default group every account belongs to and cannot be removed.

### Step 5 — Moved Account to Disabled Accounts OU

- Right-clicked **cwalker** → **Move**
- Selected `_Disabled Accounts` OU
- Clicked OK

### Step 6 — Documented the Offboarding

- Opened cwalker's properties → General tab
- Added to the Description field:

```
Offboarded 06/2026 - Account disabled, removed from all security groups, moved to Disabled Accounts OU
```

### Step 7 — Verified All Changes

```powershell
Get-ADUser cwalker -Properties Enabled, Description, DistinguishedName | Select-Object Name, Enabled, Description, DistinguishedName
```

**Output:**
```
Name:         : Chris Walker
Enabled       : False
Description   : Offboarded 06/2026 - Account disabled, removed from all security groups, moved to Disabled Accounts OU
DistinguishedName : CN=Chris Walker,OU=_Disabled Accounts,DC=lab,DC=local
```

---

## Offboarding Checklist Summary

| Task | Status |
|---|---|
| Disable account | ✅ Complete |
| Reset password | ✅ Complete |
| Remove security group memberships | ✅ Complete |
| Move to Disabled Accounts OU | ✅ Complete |
| Document the change | ✅ Complete |
| Verify all changes | ✅ Complete |

---

## Key Commands Used

```powershell
# Check group memberships
Get-ADPrincipalGroupMembership cwalker | Select-Object Name

# Remove from a security group
Remove-ADGroupMember -Identity "ITSupport" -Members "cwalker"

# Verify account status and properties
Get-ADUser cwalker -Properties Enabled, Description, DistinguishedName | Select-Object Name, Enabled, Description, DistinguishedName
```

---

## What I Learned

- The complete checklist process for offboarding a departing employee
- How to disable an account without permanently deleting it
- Why resetting the password is a critical security step during offboarding
- How to check and remove AD security group memberships using PowerShell
- The purpose of using a dedicated OU for disabled accounts
- The importance of documenting changes directly on the AD object for future reference
- How to verify offboarding was completed correctly using PowerShell

---

## Real World Application

User offboarding is one of the most commonly asked interview questions for entry-level IT roles: **"Walk me through what you would do when an employee leaves the company."**

A proper offboarding process protects the organization by:

- Preventing a former employee from accessing company resources
- Removing access to sensitive shared folders and systems immediately
- Maintaining an audit trail of who was offboarded, when, and what access was removed
- Allowing accounts to be easily identified and cleaned up later (e.g., deleted after a retention period)

Disabling rather than immediately deleting an account is standard practice — it preserves the account for potential rehires, audits, or legal holds while completely preventing access.

---

## Next Steps

- Lab 09 — RDP Troubleshooting
