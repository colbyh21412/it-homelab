# Lab 09 — RDP Troubleshooting

**Date:** June 2026

**Platform:** VirtualBox

**Environment:** lab.local domain (built in Lab 01)

---

## Objective

Configure Remote Desktop Protocol (RDP) between DC01 and CLIENT01, then intentionally break it to practice troubleshooting a common help desk ticket: "I can't remote into my computer."

---

## Environment

| Machine | Role | IP Address |
|---|---|---|
| DC01 | Domain Controller / Remote support technician machine | 192.168.1.10 (static) |
| CLIENT01 | Domain-joined workstation | 192.168.1.20 (static) |

---

## What I Did

### Step 1 — Enabled Remote Desktop on CLIENT01

- Pressed Windows key + R → `sysdm.cpl` → Remote tab
- Selected **Allow remote connections to this computer**
- Unchecked **Allow connections only from computers running Remote Desktop with Network Level Authentication** (for lab simplicity)
- Clicked Apply → OK

### Step 2 — Connected from DC01 via RDP

- On DC01 ran `mstsc`
- Entered `192.168.1.20` as the computer to connect to
- Authenticated with `LAB\ajohnson`
- Accepted the certificate warning (expected in a lab environment without a trusted CA)
- Successfully connected to CLIENT01's desktop remotely

### Step 3 — Intentionally Broke RDP

- On CLIENT01: `sysdm.cpl` → Remote tab
- Selected **Don't allow remote connections to this computer**
- Clicked Apply → OK

### Step 4 — Attempted RDP Connection (Simulated Ticket)

On DC01 ran `mstsc` → `192.168.1.20`

**Error received:**
```
Remote Desktop can't connect to the remote computer for one of these reasons:
1) Remote access to the server is not enabled
2) The remote computer is turned off
3) The remote computer is not available on the network
```

This is the exact error message a user would report in a help desk ticket.

### Step 5 — Troubleshooting Process

**Check 1 — Is the computer online?**

```
ping 192.168.1.20
```

Verified CLIENT01 responds to ping — confirms the machine is powered on and network connectivity is working.

**Check 2 — Is Remote Desktop enabled?**

On CLIENT01: `sysdm.cpl` → Remote tab

Found: **Don't allow remote connections to this computer** was selected — this was the root cause.

### Step 6 — Resolved the Issue

- Changed setting to **Allow remote connections to this computer**
- Clicked Apply → OK
- Returned to DC01 and ran `mstsc` → `192.168.1.20`
- Connection succeeded

---

## RDP Troubleshooting Checklist

When a user reports they cannot be remoted into, check in this order:

```
1. Is the target computer powered on and connected to the network?
   → ping the computer's IP address

2. Is Remote Desktop enabled on the target computer?
   → sysdm.cpl → Remote tab → "Allow remote connections"

3. Is the Windows Firewall blocking RDP (port 3389)?
   → Check Windows Defender Firewall rules for "Remote Desktop"

4. Is the user account allowed to use RDP?
   → Check Remote Desktop Users group membership for non-admin accounts

5. Is the Remote Desktop Services service running?
   → services.msc → Remote Desktop Services
```

---

## Key Facts About RDP

- RDP uses **port 3389** by default
- A computer that is powered off cannot accept RDP connections
- By default, only members of the local **Administrators** group and the **Remote Desktop Users** group can connect via RDP
- Multiple users typically cannot have simultaneous RDP sessions on non-server editions of Windows — connecting kicks the existing session
- **Ctrl+Alt+Delete** does not work inside an RDP session — use **Ctrl+Alt+End** instead to bring up the security screen
- Certificate warnings during RDP connection are normal in environments without an internal Certificate Authority

---

## Issues Encountered and How I Fixed Them

### Issue 1 — Could not stop Remote Desktop Services via services.msc

- **Cause:** Remote Desktop Services has dependent services, and Windows prevents stopping it directly with an unclear error
- **Fix:** Used the simpler and more realistic method of disabling RDP via System Properties (`sysdm.cpl` → Remote tab) instead, which is also the most common way RDP gets accidentally disabled in real environments

---

## Key Commands Used

```
mstsc                    # Open Remote Desktop Connection client
ping 192.168.1.20        # Test connectivity to target computer
sysdm.cpl                # Open System Properties (Remote tab for RDP settings)
services.msc             # Open Services management console
```

---

## What I Learned

- How to enable and configure Remote Desktop on a Windows machine
- How to initiate an RDP connection using mstsc
- The exact error message users see when RDP is disabled on the target machine
- A systematic troubleshooting checklist for RDP connectivity issues
- The difference between a network connectivity issue (ping fails) and an RDP configuration issue (ping succeeds, RDP fails)
- Key RDP facts including default port, session limits, and the Ctrl+Alt+End shortcut
- Why certificate warnings appear during RDP in lab/internal environments

---

## Real World Application

Remote support is one of the most frequent tasks in help desk roles. Common RDP-related tickets include:

- "I can't connect to my work computer from home"
- "IT needs to remote into my machine to fix an issue, but it won't connect"
- "My RDP session keeps disconnecting"

Being able to methodically work through the troubleshooting checklist above — starting with connectivity, then checking RDP settings, then firewall, then user permissions — demonstrates a structured approach that interviewers look for in entry level candidates.

---

## Next Steps

- Lab 10 — Printer Deployment via Logon Script
