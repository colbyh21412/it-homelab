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
### Step 1 - Installed the DHCP Server Role
  •  Opened Server Manager on DC01
  
  •  Clicked Manage -> Add Roles and Features
  
  •  Selected DHCP Server Role
  
  •  Completed installation
  
### Step 2 - Created a DHCP Scope
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

### Why This range was chosen:
  •  DC01 uses static IP 192.168.1.10
 
  •  CLIENT01 was previously using static IP 192.168.1.20
 
  •  Range starts at .50 to leave room for future static devices (servers, printers, network equipment)
 
  •  Range ends at .100 providing 51 available IP addresses for DHCP clients

### Step 3 - Configured DHCP Options
| Option | Value | Purpose |
| :--- | :--- | :--- |
| Router (Default Gateway) | 192.168.1.1 | Tells clients where to send external traffic |
| DNS Server | 192.168.1.10 | Points clients to DC01 for DNS resolution
| Parent Domain | lab.local | Tells clients the domain name |

### Step 4 - Activated the Scope
  •  Right-clicked Lab Network scope -> Activate
  
  •  Scope status confirmed as active in DHCP console
  
### Step 5 - Configured CLIENT01 to Use DHCP
  •  Logged into CLIENT01 as LAB\Administrator
  
  •  Opened Network Connections (ncpa.cpl)
  
  •  Changed IPv4 settings from static to:
    •  Obtain an IP address automatically
    •  Obtain DNS server address automatically
    
  •  Disabled and re-enabled the network adapter to force a fresh DHCP request
  
### Step 6 - Verified DHCP Assignment
  • Ran ipconfig /all on CLIENT01 and confirmed:
  | Setting | Value |
  | :--- | :--- | 
  | IPv4 Address | 192.168.1.50 |
  | DHCP Enabled | Yes |
  | DHCP Server | 192.168.1.10 |
  | DNS Servers | 192.168.1.10 |
  
  • Verified CLIENT01 lease appears in DHCP console under IPv4 -> Lab Network -> Address Leases
  
## How DHCP Works
When a device connects to the network and requests an IP address, the following process occurs:
  1.  Client broadcasts "I need an IP address" (DHCP Discover)
  2.  DHCP server responds with an offer (DHCP offer)
  3.  Client accepts the offer (DHCP Request)
  4.  DHCP server confirms the assignment (DHCP Acknowledge)

This is known as the DORA process (Discover, Offer, Request, Acknowledge)

### Lease Duration
IP addresses assigned by DHCP are temporary, or leased. My lab scope uses an 8-day lease duration, meaning:
  •  At 50% of the lease (day 4) the client automatically tries to renew
  
  •  At 87.5% (day 7) the client tries again if renewal failed
  
  •  At 100% (day 8) the lease expires and the IP returns to the pool

### IP exhaustion 
If all IPs in the scope are in use and a new device tries to connect, DHCP cannot assign an address. This is called IP exhaustion and is a real help desk troubleshooting scenario. Fixes include:
  •  Expanding the DHCP scope range
  
  •  Shortening the lease duration
  
  •  Adding additional subnets
  
### APIPA
If a device cannot reach a DHCP server, it assigns itself an Automatic Private IP Address (APIPA) in the 169.254.x.x range. Seeing a 169.254.x.x address on a machine is a clear sign the device cannot communicate with the DHCP server.

## Issues Encountered and How I Fixed Them
### Issue 1 - CLIENT01 received a 169.254.x.x APIPA address instead of a DHCP address
  • Cause:
  
  • Fix:
  
### Issue 2 - ipconfig /renew returned NCB error
  • Cause: 
  
  • Fix: 
  
## Key Commands Used

## What I Learned

## Real World Applications

## Next Steps
