# Investigation Timeline — CASE-002

Chronological timeline of the simulated attack chain, from first observed event to final cleanup.
All times are local machine time (ANNAYA_THAMMUDU, IST).

---

## Pre-Simulation Setup

| Time | Action | Finding |
|---|---|---|
| Before simulation | Ran `auditpol /get /subcategory:"Security System Extension"` | **No Auditing** — gap found, corrected |
| Before simulation | Ran `net accounts` | Lockout threshold = 10 (not 5 as assumed) — plan adjusted |
| Before simulation | Created `testuser1` via `net user` | Confirmed in account list |

---

## Attack Chain Events

| Time | Event ID | Description | Severity |
|---|---|---|---|
| 14:39:36 | 4625 | Failed logon attempt #1 against `testuser1` | Medium |
| 14:39:37 | 4625 | Failed logon attempt #2 | Medium |
| 14:39:39 | 4625 | Failed logon attempt #3 | Medium |
| 14:39:41 | 4625 | Failed logon attempt #4 | Medium |
| 14:39:43 | 4625 | Failed logon attempt #5 | Medium |
| 14:39:45 | 4625 | Failed logon attempt #6 — Windows throttling activates | Medium |
| 14:42:08 | 4625 | Failed logon attempt #7 (after throttle delay) | Medium |
| 14:42:10 | 4625 | Failed logon attempt #8 | Medium |
| 14:42:11 | 4625 | Failed logon attempt #9 — final failure | Medium |
| **14:42:32** | **4624** | **Successful logon — `testuser1` — initial access achieved** | **High** |
| 15:30:45 | 4720 | New account `backdoor_svc` created | Medium |
| **15:34:18** | **4732** | **`backdoor_svc` added to Administrators — privilege escalation** | **Critical** |
| 15:49:21 | 4697 | Service `WinUpdateHelper` installed as LocalSystem — persistence | Critical |

---

## Post-Investigation Cleanup

| Time | Action |
|---|---|
| After simulation | Deleted `testuser1` via `net user testuser1 /delete` |
| After simulation | Deleted `backdoor_svc` via `net user backdoor_svc /delete` |
| After simulation | Deleted `WinUpdateHelper` service via `sc.exe delete WinUpdateHelper` |
| After simulation | Confirmed clean account list via `net user` |

---

## Attack Chain Summary

```
14:39:36 ──► Brute force begins (9x Event 4625 over ~2.5 minutes)
14:42:32 ──► Initial access achieved (Event 4624 — testuser1)
15:30:45 ──► Backdoor account created (Event 4720 — backdoor_svc)
15:34:18 ──► Privilege escalation (Event 4732 — added to Administrators) ◄── CRITICAL
15:49:21 ──► Persistence installed (Event 4697 — WinUpdateHelper service)
```

**Total time from first brute-force attempt to persistence established: ~70 minutes**
(Includes deliberate pauses between stages in the simulation.)
