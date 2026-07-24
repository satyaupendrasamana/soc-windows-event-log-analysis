# Windows Event Log Analysis — SOC L1 Investigation Project

**Case Study: CASE-002**  
**Analyst:** Satya Upendra Samana  
**Date:** July 2026  
**Environment:** Personal Windows Laptop (ANNAYA_THAMMUDU, Workgroup, non-domain)

---

## What This Project Is

This is a hands-on SOC L1 investigation project built from scratch — not a simulation using pre-made log files, and not a theory exercise. Every log entry in this repository was generated on a real Windows machine by deliberately simulating a complete attack chain, and then investigated the same way a SOC L1 analyst would investigate a live incident.

The attack chain simulated:

- Brute-force logon attempts against a test account
- Successful initial access after repeated failures
- Backdoor account creation using a disguised name
- Privilege escalation by adding the backdoor account to the Administrators group
- Persistence via a disguised Windows service installation

All five stages produced real Windows Security event log entries. Those entries were then correlated into a timeline, classified by attack pattern, and documented in a full investigation report.

---

## What This Project Demonstrates

**Log analysis skills**  
Reading and interpreting raw Windows Security event log entries at the field level — not just knowing what an Event ID number means, but understanding what each field inside the entry tells you and how to use it.

**Core Event ID knowledge**  
Working fluency with the seven Event IDs that cover the majority of real SOC L1 investigations: 4625, 4624, 4634, 4647, 4720, 4732, 4688, and 4697.

**Correlation and timeline building**  
Connecting multiple log entries by account name, Logon ID, source address, and time window to reconstruct a complete attack sequence from scattered evidence.

**Attack pattern recognition**  
Identifying brute force, privilege escalation, and persistence patterns directly from log evidence — not from a pre-labelled dataset.

**Audit policy diagnosis**  
Two default Windows audit gaps were discovered during the investigation. The Security System Extension subcategory was set to No Auditing by default, which initially suppressed Event ID 4697 entirely. The account lockout threshold was 10 attempts, not 5 as assumed, which affected simulation planning. Both were identified, corrected, and documented as genuine findings.

**Professional investigation documentation**  
The full report follows the same structure as a real SOC L1 incident report — executive summary, scope and methodology, detailed evidence per event, attack classification, critical finding, recommendations, and conclusion.

---

## Repository Structure

```
windows-event-log-analysis/
    README.md
    docs/
        Case-002-SOC-Windows-Event-Log.docx
    evidence/
        01_event_4625_failed_logons.txt
        02_event_4624_successful_logon.txt
        03_event_4720_account_created.txt
        04_event_4732_privilege_escalation.txt
        05_event_4697_service_installed.txt
    screenshots/
        01_failed_logons_4625.png
        02_successful_logons_4624.png
    Documentation/
        Investigation-Methodology.md
        Event-ID-Reference.md
        Timeline.md
        Tools-Used.md
```

**docs/** contains the full formal investigation report in Word format.  
**evidence/** contains one plain-text file per event, with the raw log entry and analyst annotation.  
**screenshots/** contains real Event Viewer captures taken during the investigation.  
**Documentation/** contains the methodology, event ID reference, chronological timeline, and tools reference.

---

## Attack Timeline

| Step | Time | Event ID | What Happened |
|------|------|----------|---------------|
| 1 | 14:39:36 to 14:42:11 | 4625 x9 | Nine failed logon attempts against testuser1 in approximately 2.5 minutes |
| 2 | 14:42:32 | 4624 | Successful logon to testuser1 — initial access achieved 21 seconds after the final failure |
| 3 | 15:30:45 | 4720 | New local account backdoor_svc created with a name designed to resemble a legitimate service account |
| 4 | 15:34:18 | 4732 | backdoor_svc added to the built-in Administrators group — privilege escalation |
| 5 | 15:49:21 | 4697 | Service WinUpdateHelper installed running as LocalSystem — persistence established |

---

## Critical Finding

The single most important event in this chain is Step 4, Event 4732. At the point the backdoor account was created (Step 3), it was a standard unprivileged account with no meaningful access. It was only once it was added to the Administrators group that it gained full control of the host — the ability to install services, disable security tools, access all files, and create further accounts.

In a real investigation, catching and reverting this one event would have neutralized the backdoor account before the persistence mechanism could be installed. The account would still exist, but it would be powerless.

---

## Audit Policy Gaps Found

Two default audit configuration gaps were discovered during this investigation that would have hidden key evidence in a real incident.

**Security System Extension — No Auditing by default**  
Event ID 4697 was completely suppressed on this machine out of the box. The WinUpdateHelper service was created successfully, confirmed by PowerShell output, but left no entry in the Security log until the subcategory was manually enabled via auditpol. This gap is not obvious — there is no error, no warning, just silence.

**Account lockout threshold — 10 attempts, not 5**  
The simulation was initially designed around a 5-attempt lockout threshold based on a common assumption. The actual configured threshold was 10, confirmed via net accounts. This meant the Account Lockout event (4740) was never triggered during the simulation, and the brute-force succeeded before any lockout policy activated.

Both findings are documented in the evidence files and the investigation report.

---

## Tools Used

All tools used in this investigation are native to Windows. No third-party software was required.

| Tool | Purpose |
|------|---------|
| Windows Event Viewer | Read and filter Security log entries |
| PowerShell — net user | Create and delete test accounts |
| PowerShell — net localgroup | Manage group membership for privilege escalation simulation |
| PowerShell — sc.exe | Create and delete the test Windows service |
| PowerShell — auditpol | Check and correct audit policy configuration |
| PowerShell — net accounts | Verify lockout threshold before simulation |
| MITRE ATT&CK Framework | Map observed behaviors to standardized technique IDs |

---

## MITRE ATT&CK Mapping

| Technique ID | Description |
|-------------|-------------|
| T1110 — Brute Force | Repeated failed logon attempts against a single account until one succeeded |
| T1136.001 — Local Account Creation | New local account backdoor_svc created to maintain access beyond initial compromise |
| T1098 — Account Manipulation | Backdoor account added to the Administrators group to gain elevated privileges |
| T1543.003 — Windows Service | Service WinUpdateHelper installed to maintain persistence across reboots |
| T1036 — Masquerading | Service and account names chosen to resemble legitimate system components |

---

## Context and Disclosure

This investigation was performed entirely on a personal, standalone Windows laptop in a Workgroup configuration. All activity originated locally — Source Network Address 127.0.0.1 throughout. There was no real attacker, no external network involvement, and no actual compromise. The test accounts (testuser1 and backdoor_svc) and the dummy service (WinUpdateHelper) were deleted after the investigation was complete and the evidence was captured.

The report and all supporting files are structured to match the format of a real SOC L1 incident report. The intent is to demonstrate the investigation workflow, log analysis skill, and documentation standard expected at L1 level — not to claim this was a live production incident.

---

## Background

This project is the final phase of a self-directed SOC L1 learning path covering Windows OS fundamentals, Windows security concepts (Active Directory, domain vs workgroup, SIDs, authentication vs authorization), Windows event logging, core Event IDs, logon types and authentication details, log correlation and timeline building, and common attack pattern recognition. The learning path began from absolute zero and progressed through nine structured phases before reaching this investigation project.

CASE-002 is part of the same portfolio series as CASE-001 (SOC Phishing Email Investigation), following the same investigation structure and documentation standard.

---

## Author

Satya Upendra Samana  
SOC Analyst Fresher  
Google Cybersecurity Certified  
July 2026  
GitHub: https://github.com/satyaupendrasamana
