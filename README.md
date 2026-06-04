# IT Home Lab

About This Lab:
This repository documents my hand-on IT home lab. This is being worked on and built to build skills towards an entry-level IT support role. This lab is built using VirtualBox on a local machine using Windows Server 2025 ISO - no cloud services required.

Active Directory Domain Setup:
Date: June 2026
Platform: VirtualBox
OS: Windows Server 2025 (Evaluation)

Objective:
Build a functional Active Directory domain from scratch, including a Domain Controller, Organizational Units (OUs), user accounts, security groups, and a domain-joined client machine.

Environment:

| Machine | Role | IP Address |
| :--- | :--- | :--- |
| DCO1    | Domain Controller/DNS Server | 192.168.1.10 |
| CLIENT01 | Domain-joined Workstation | 192.168.1.20 |

• Domain: lab.local
• Network: VirtualBox Bridged Adapter

## What I built

Step 1 - Created DC01 VM in VirtualBox
  •  Installed Windows Server 2025 (Desktop Experience)
  •  Set static IP: 192.168.1.10
  •  Set DNS to 127.0.0.1(Loopback Address) in preparation for AD DS

Step 2 - Installed Active Directory Domain Services (AD DS)
  •  Used Server Manager --> Add Roles and Features
  •  Added AD DS role with all required features as part of the install

Step 3 - Promoted DC01  to Domain Controller 
  •  Created a new forest: lab.local
  •  Configured DSRM password
  •  DNS role is installed automatically during promotion

Step 4 - Created Organizational Units
```text
lab.local
└── _Branches
    └── Houston
        ├── Users
        ├── Workstations
        └── Laptops
└── _Groups
```
Step 5 - Created Domain User Accounts

| Name | Username | OU |
| :--- | :--- | :--- |
| Alice Johnson    | ajohnson | _Groups |
| Bob Martinez | bmartinez | _Groups |
| Chris Walker | cwalker | _Groups |

Step 6 - Created Security Groups and Added Members

| Group | Member | OU |
| :--- | :--- | :--- |
| Helpdesk    | ajohnson | _Groups |
| Accounting | bmartinez | _Groups |
| ITSupport | cwalker | _Groups |

Step 7 - Created CLIENT01 and Joined Domain
  •  Installed Windows Server 2025 on CLIENT01 VM
  •  Set static IP: 192.168.1.20
  •  Set DNS to point to DC01: 19.168.1.10
  •  Joined domain lab.local via sysdm.cpl
  •  Authenticated with LAB\Administrator credentials

Step 8 - Tested Domain Authentication
  •  Logged into CLIENT01 as LAB\ajohnson
  •  Verified group membership with whoami \groups
  •  Confirmed Helpdesk group appeared in token



## Issues Encountered and How I Fixed Them

---

  ### Issue 1 - VM not booting from ISO
•  **Cause:** Boot order had Hard Disk above optical

•  **Fix:** Changed the boot order in VirtualBox Settings -> System -> Boot Order to put Optical Drive first

  ### Issue 2 - Partition not available during Windows Server install
•  **Cause:** Disk showed as unallocated with no New option visible

•  **Fix:** Used the Create Partition option to initialize the disk before installing
      
  ### Issue 3 - DC01 network adapter was set to NAT after domain promotion
•  **Cause:** Network Adapter was changed from NAT to Bridged Adapter after AD DS was already promoted, causing DNS records to register against the wrong network
     
•  **Fix:** Demoted DC01 using Uninstall-ADDSDomainController, corrected network settings, then repromoted

 ### Issue 4 - IP6 DNS servers interfering with domain join
•  **Cause:** Bridge Adapter inherited IPV6 DNS servers from the home router, which took priority over DC01's DNS

•  **Symptoms:** nslookup lab.local timed out, nltest /dsgetdc:lab.local returned ERROR_NO_SUCH_DOMAIN

•  **Fix:** Disabled IPv6 on both DC01 and CLIENT01 via network adapter properties (ncpa.cpl)
      
  ### Issue 5 - CLIENT01 rebooting into Windows Setup instead of installed OS
•  **Cause:** Boot order reverted to Optical above Hard Disk after VM reset
     
•  **Fix:** Changed boot order in VirtualBox to Hard Disk first

 ### Issue 6 - Sign-in method not allowed after password policy was set
•  **Cause:** Allow log on locally policy did not include Domain Users or Administrators 
     
•  **Symptoms:** "The sign-in method you're trying to use isn't allowed" error on CLIENT01
     
•  **Fix:** Edited Default Domain Policy -> Local Policies -> User Rights Assignment -> Allow log on locally -> added Domain Users and Administrators

 ### Issue 7 - User in wrong security group
•  **Cause:** ajohnson was accidentally added to Accounting instead of Helpdesk
     
•  **Symptoms:** whoami /groups showed LAB\Accounting instead of LAB\Helpdesk
     
•  **Fix:** Removed ajohnson from Accounting, added to Helpdesk in ADUC. Logged out and back in to rebuild token



# Key Commands Used

 ## Network diagnostics
 ```
 ipconfig /all               # View full network configuration          
 ipconfig /flushdns          # Clear DNS cache
 ipconfig /registerdns       # Re-register DNS records
 ping 192.168.1.10           # Test connectivity to DC01
 nslookup lab.local          # Test DNS resolution
 tracert 192.168.1.10        # Trace route to DC01

 ```
## Domain Diagnostics
 ```
 nltest /dsgetdc:lab.local   # Locate the domain controller
 dcdiag /test:dns            # Run full DNS diagnostic on DC
 whoami /groups              # view current user's group memberships
 ```
## DNS and Services
 ```
 net stop dns                # Stop the DNS service
 net start dns               # Start the DNS service
 Restart-Service DNS         # Restart DNS via PowerShell
 Resolve-DnsName lab.local   # Test DNS resolution via power shell

 ```
 ## AD demotion
 ```
 # demote DC01 - used when repromoting was needed to fix DNS issues
 Uninstall-ADDSDomainController -DemoteOperationMasterRole -RemoveApplicationPartitions -Force -LastDomainControllerInDomain
 ```

What I learned
 •  How to install and configure Active Directory Domain Services on Windows Server
 •  How to promote a server to a Domain Controller and create a new forest
 •  How to structure an AD environment using Organizational Units
 •  How to create and manage domain user accounts and security groups
 •  How to join a client machine to a domain
 •  How DNS is critical to Active Directory — domain joins fail if DNS is misconfigured
 •  How IPv6 DNS entries from a home router can interfere with lab environments
 •  How to use dcdiag and nltest to diagnose AD and DNS issues
 •  How VirtualBox networking modes (NAT vs Bridged vs Internal) affect VM communication
 •  How to configure password policies via Group Policy
 •  How to create and assign Logon Scripts to domain users
 •  How to delegate specific AD permissions to security groups without granting full admin rights
 •  How to organize computer accounts into OUs for Group Policy targeting
 •  How user tokens work and why logging out and back in is required after group membership changes
 •  How the Allow log on locally policy controls which users can sign into a machine

  

Next Steps
 •  Configure Group Policy Objects (GPOs)
 •  Set up account lockout policies
 •  Practice password resets and account management
 •  Delegate help desk permissions to the Helpdesk security group
 •   Set up a logon script

### Bonus Tasks Completed

**Bonus Task 1 - Set Password Policy via Group Policy**
- Opened Group Policy Management on DC01
- Edited Default Domain Policy
- Navigated to Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Password Policy
- Configured the following settings:
  - Minimum password length: 10 characters
  - Maximum password age: 90 days
  - Minimum password age: 1 day
  - Password must meet complexity requirements: Enabled

**Bonus Task 2 - Created a Logon Scripts**
- Created logon.bat in C:\Windows\SYSVOL\sysvol\lab.local\scripts
- Script displays a welcome message with the username on login
- Assigned script to ajohnson via ADUC profile tab
- Verified script runs on CLIENT01 login

**Bonus Task 3 - Delegated Password Reset to Helpdesk**
- Right-clicked Houston -> Users OU in ADUC
- User Delegate Control Wizard
- Granted Helpdesk group permission to reset user passwords and force password change at next logon
- Helpdesk can now reset passwords without Domain Admin rights

**Bonus task 4 - Moved CLIENT01 into correct OU**
- Located CLIENT01 computer account in the default Computers container
- Moved it to _Branches -> Houston -> Workstations
- Ran gpupdate /force on CLIENT01 to apply OU-level Group Policy
 
