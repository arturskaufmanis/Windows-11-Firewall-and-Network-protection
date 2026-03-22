# Command Reference — Windows 11 Firewall Lab

All commands run in **Windows PowerShell (Administrator)** unless noted.

---

## Phase 1 — Baseline & DNS Resolution

```powershell
# Confirm target loads in browser first (manual step)
# https://example.com

# Resolve all IPs for the target domain
Resolve-DnsName example.com
```

**Expected output:**
```
Name         Type  TTL  IPAddress
----         ----  ---  ---------
example.com  AAAA  100  2606:4700::6812:1a78
example.com  AAAA  100  2606:4700::6812:1b78
example.com  A     259  104.18.27.120
example.com  A     259  104.18.26.120
```

> NOTE: Block ALL returned IPs. Missing one allows the browser to bypass the rule.

---

## Phase 2 — Create Block Rule (PowerShell)

```powershell
New-NetFirewallRule -DisplayName "LAB-Block-ExampleCom" `
    -Direction Outbound `
    -Action Block `
    -RemoteAddress 104.18.26.120,104.18.27.120,2606:4700::6812:1a78,2606:4700::6812:1b78 `
    -Profile Any `
    -Enabled True
```

**Verify rule exists:**
```powershell
Get-NetFirewallRule -DisplayName "LAB-Block-ExampleCom"
```

**Test connectivity (expect False):**
```powershell
Test-NetConnection -ComputerName 104.18.26.120 -Port 443
# TcpTestSucceeded : False
```

---

## Phase 3 — Enable Firewall Drop Logging

```powershell
# Enable via netsh (works on all Windows 11 builds)
netsh advfirewall set allprofiles logging droppedconnections enable

# Verify logging is active
netsh advfirewall show allprofiles | findstr -i "log"
# LogDroppedConnections: Enable (x3 for Domain/Private/Public)
```

**Read the drop log:**
```powershell
Get-Content "$env:SystemRoot\system32\LogFiles\Firewall\pfirewall.log" | Select-Object -Last 20
```

**Log format:**
```
date  time  DROP  TCP  src-ip  dst-ip  src-port  dst-port ... SEND
2026-03-21 18:11:54 DROP TCP 192.168.1.65 104.18.26.120 58036 443 0 - 0 0 0 - - - SEND 904
```

---

## Phase 4 — Event Viewer Audit Trail

```powershell
# Get recent firewall events (last 10)
Get-WinEvent -LogName "Microsoft-Windows-Windows Firewall With Advanced Security/Firewall" `
    -MaxEvents 10 | Format-List TimeCreated, Id, Message
```

**Key Event IDs:**

| ID   | Meaning |
|------|---------|
| 2004 | Firewall rule added |
| 2006 | Firewall rule deleted |
| 2082 | Firewall setting changed |
| 5157 | WFP blocked a connection |

---

## Phase 5 — Remediation (PowerShell)

```powershell
# Remove the block rule
Remove-NetFirewallRule -DisplayName "LAB-Block-ExampleCom"

# Verify gone (returns nothing if deleted)
Get-NetFirewallRule -DisplayName "LAB-Block-ExampleCom" -ErrorAction SilentlyContinue

# Confirm connectivity restored
Test-NetConnection -ComputerName 104.18.26.120 -Port 443
# TcpTestSucceeded : True
```

---

## Phase 6 — Block & Remediate via GUI (wf.msc)

```
Win+R → wf.msc → Outbound Rules → New Rule...

Wizard:
  Rule Type    → Custom
  Program      → All programs
  Protocol     → Any
  Scope        → Remote: These IP addresses → add all 4 IPs
  Action       → Block the connection
  Profile      → Domain ✓  Private ✓  Public ✓
  Name         → LAB-Block-ExampleCom-GUI
  → Finish

To delete:
  Right-click rule → Delete → Yes
```

---

## Phase 7 — Network Protection

```powershell
# Check current state (0=Disabled, 1=Enabled, 2=AuditMode)
Get-MpPreference | Select-Object EnableNetworkProtection

# Enable Audit Mode (logs without blocking — safe for testing)
Set-MpPreference -EnableNetworkProtection AuditMode
Get-MpPreference | Select-Object EnableNetworkProtection
# Returns: 2

# Enable full enforcement (actively blocks)
Set-MpPreference -EnableNetworkProtection Enabled
Get-MpPreference | Select-Object EnableNetworkProtection
# Returns: 1

# Test: navigate to Microsoft's official NP test URL
# https://smartscreentestratings2.net
# Expected: full red block page — "This site has been reported as unsafe"

# Restore to original state
Set-MpPreference -EnableNetworkProtection Disabled
Get-MpPreference | Select-Object EnableNetworkProtection
# Returns: 0
```

---

## Phase 8 — SmartScreen GUI

```
Windows Security → App & browser control → Reputation-based protection settings

Toggles available:
  - Check apps and files (SmartScreen for downloads)
  - SmartScreen for Microsoft Edge
  - Phishing protection
  - Potentially unwanted app blocking
  - SmartScreen for Microsoft Store apps

Note: Master EnableNetworkProtection toggle only available via
PowerShell or Group Policy on standalone Windows 11 builds.
```

---

## Bonus — Enable WFP Connection Audit Logging

```powershell
# Enables Event ID 5157 (connection blocked) in Security log
auditpol /set /subcategory:"Filtering Platform Connection" /failure:enable
```

---

## Quick Triage Workflow

```
User: "website won't load"
  ↓
Resolve-DnsName <domain>          → does it resolve?
  ↓ Yes
Test-NetConnection -Port 443      → TcpTestSucceeded?
  ↓ False
Get-Content pfirewall.log         → DROP entries to that IP?
  ↓ Yes
Get-NetFirewallRule | Where Action -eq Block  → find the rule
  ↓
Check Event ID 2082               → who created it and when?
  ↓
Remove-NetFirewallRule            → remediate
  ↓
Test-NetConnection again          → TcpTestSucceeded: True ✓
```
