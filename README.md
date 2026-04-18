# Windows-11-Firewall-and-Network-protection
This project documents a complete hands-on lab exercise covering Windows 11 firewall configuration and network protection. The lab was designed to build practical experience with the exact scenarios encountered by SOC analysts and system administrators when investigating reports of blocked or inaccessible websites.
# Windows 11 Firewall & Network Protection — Hands-On Lab

**SOC Analyst Training Series | March 2026**

## Overview

A complete practical exercise covering Windows Defender Firewall configuration and Network Protection (SmartScreen). The lab builds real diagnostic skills for investigating "website won't load" scenarios from a SOC analyst or sysadmin perspective.
https://arturskaufmanis.github.io/Windows-11-Firewall-and-Network-protection/ 
[View my HTML] (https://arturskaufmanis.github.io/Windows-11-Firewall-and-Network-protection/myfile.html)
## Lab Environment

| Component | Details |
|---|---|
| Platform | Windows 11 Pro |
| Browser | Microsoft Edge |
| Shell | PowerShell (Administrator) |
| Test Target | https://example.com |
| NP Test URL | https://smartscreentestratings2.net |

## Objectives

- Block a website using firewall outbound rules (PowerShell & GUI)
- Observe and document browser error messages
- Read and interpret firewall drop logs (pfirewall.log)
- Read Event Viewer firewall audit trail (Event ID 2082)
- Remediate the block and restore connectivity
- Enable and test Windows Defender Network Protection (SmartScreen)

## Project Files

| File | Description |
|---|---|
| `README.md` | This file |
| `commands.md` | All commands in execution sequence |
| `Windows11-Firewall-Lab-Report.docx` | Full illustrated report with all screenshots |

## Phases Completed

1. Baseline — confirm example.com loads
2. DNS resolution — `Resolve-DnsName example.com` (4 IPs returned)
3. Block via PowerShell — `New-NetFirewallRule`
4. Observe browser error — `ERR_NETWORK_ACCESS_DENIED`
5. Enable firewall drop logging — `netsh advfirewall`
6. Read drop log — `pfirewall.log` showing DROP entries
7. Read Event Viewer audit trail — Event ID 2082
8. Remediate — `Remove-NetFirewallRule`, verify `TcpTestSucceeded: True`
9. Repeat block and remediation via **wf.msc GUI wizard**
10. Network Protection — AuditMode → Enabled → test → Disabled
11. SmartScreen GUI — Windows Security App & browser control

## Key Diagnostic Reference

| Browser Error | Cause | Speed |
|---|---|---|
| ERR_NETWORK_ACCESS_DENIED | WFP firewall block | Instant |
| ERR_CONNECTION_TIMED_OUT | Firewall silent drop | ~30s timeout |
| Red SmartScreen page | Network Protection | Instant |
| ERR_NAME_NOT_RESOLVED | DNS block | Instant |

## Skills Demonstrated

`Resolve-DnsName` · `New-NetFirewallRule` · `Remove-NetFirewallRule` · `Test-NetConnection` · `netsh advfirewall` · `Get-Content pfirewall.log` · `Get-WinEvent` · `Set-MpPreference` · `wf.msc` · Windows Security GUI
