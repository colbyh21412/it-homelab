# Lab 04 - DHCP setup

Date: june 2026

Platform: VirtualBox

Environment: lab.local domain (built in Lab 01)

## Objective
Install and configure a DHCP server on DC01 to automatically assign IP addresses, DNS servers, and gateway information to client machines on the network instead of relying on manually configured static IPs.

## Environment
| Machine | Role | IP Address |
| :--- | :--- | :--- |
| DCO1    | Domain Controller | 192.168.1.10 (static) |
| CLIENT01 | Domain-joined Workstation | 192.168.1.50 (DHCP assigned) |

## What I did
Step 1 - Installed the DHCP Server Role
  •  Opened Server Manager on DC01
  
  •  Clicked Manage -> Add Roles and Features
  
  •  Selected DHCP Server Role
  
  •  Completed installation
  
Step 2 - Created a DHCP Scope
  •  Opened DHCP console via Server Manager -> Tools -> DHCP
  
  •  Right-clicked IPv4 -> New Scope
  
  •  Configured the following:
  | Setting | Value |
| :--- | :--- |
| Scope Name | Lab Network |
| Start IP Address | 192.168.1.50 |
| End IP Address | 192.168.1.100 |
| Subnet Mask | 255.255.255.0 |
| Lease Duration | 8 days |

## How DHCP Works  

## Issues Encountered and How I Fixed Them

## Key Commands Used

## What I Learned

## Real World Applications

## Next Steps
