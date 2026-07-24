# Tools Used — CASE-002

All tools used in this investigation are native to Windows — no third-party software required.

---

| Tool | Command(s) Used | Purpose in This Investigation |
|---|---|---|
| **Windows Event Viewer** | GUI — Filter Current Log, Find | Read and filter Security log entries for Event IDs 4625, 4624, 4720, 4732, 4697 |
| **PowerShell — net user** | `net user`, `net user <name> <pass> /add`, `net user <name> /delete` | Create test accounts (`testuser1`, `backdoor_svc`), verify account list, delete accounts after cleanup |
| **PowerShell — net localgroup** | `net localgroup Administrators <name> /add` | Add `backdoor_svc` to the Administrators group (privilege escalation simulation) |
| **PowerShell — sc.exe** | `sc.exe create`, `sc.exe delete` | Create and delete the test service (`WinUpdateHelper`) for persistence simulation |
| **PowerShell — auditpol** | `auditpol /get /subcategory:"..."`, `auditpol /set /subcategory:"..." /success:enable /failure:enable` | Check and correct audit policy configuration — discovered Security System Extension was No Auditing by default |
| **PowerShell — net accounts** | `net accounts` | Verify account lockout threshold (10 attempts) and lockout duration (10 minutes) before simulation |
| **MITRE ATT&CK Framework** | https://attack.mitre.org | Map observed attacker behaviors to standardized technique IDs (T1110, T1136, T1098, T1543, T1036) |

---

## Why No Third-Party Tools?

This investigation was deliberately scoped to native Windows tooling only, for two reasons:

1. **Realistic L1 scenario** — A SOC L1 analyst investigating a single endpoint often starts with what's already available on the machine before pulling in specialized tools. Knowing how to extract and interpret evidence from built-in Windows tools is a foundational skill.

2. **Portfolio accessibility** — Anyone reading this report or repository can reproduce every step on a standard Windows machine without installing anything extra.

---

## What Would Be Used in a Real SOC Environment

| Real SOC Tool | Replaces / Enhances |
|---|---|
| SIEM (Splunk / Microsoft Sentinel) | Event Viewer — collects logs from all machines company-wide, not just one at a time |
| PowerShell Remoting / Invoke-Command | Allows pulling event logs from remote machines without physical access |
| VirusTotal / AlienVault OTX | Cross-references service names and file paths against known threat intelligence |
| ServiceNow / Jira | Ticketing system for formal case logging and L2 escalation |
