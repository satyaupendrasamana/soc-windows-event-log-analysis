# Investigation Methodology — CASE-002

This document outlines the step-by-step process followed to simulate, detect, and investigate the Windows Event Log attack chain documented in CASE-002.

---

## Pre-Investigation Checks (Before Any Simulation)

Before generating any evidence, I verified two things the machine was actually configured to capture:

1. **Audit policy check**
   ```powershell
   auditpol /get /subcategory:"Security System Extension"
   ```
   Result: `No Auditing` — this means Event ID 4697 (service installed) would be silently dropped. Corrected before simulation.

2. **Lockout threshold check**
   ```powershell
   net accounts
   ```
   Result: Lockout threshold was 10 attempts (not 5 as assumed). Corrected simulation plan accordingly.

---

## Investigation Workflow

### Step 1 — Environment Setup
- Created a controlled test account (`testuser1`) isolated from real user accounts.
- Confirmed account creation via `net user`.
- Corrected audit policy gaps before beginning.

### Step 2 — Brute-Force Simulation
- Triggered 9 consecutive failed logon attempts against `testuser1` at the Windows lock screen.
- Followed immediately with 1 correct logon.
- Filtered Security log for **Event ID 4625** and **4624** to capture the evidence.

### Step 3 — Backdoor Account Creation
- Created a new local account (`backdoor_svc`) via PowerShell.
- Filtered Security log for **Event ID 4720** to capture the account creation record.

### Step 4 — Privilege Escalation
- Added `backdoor_svc` to the built-in Administrators group via PowerShell.
- Filtered Security log for **Event ID 4732** to capture the group membership change.

### Step 5 — Persistence (Service Installation)
- Registered a new service (`WinUpdateHelper`) designed to mimic a legitimate Windows component.
- Confirmed Event ID 4697 was not appearing (audit gap discovered and corrected mid-investigation).
- Deleted and recreated the service after enabling the correct audit subcategory.
- Filtered Security log for **Event ID 4697** to capture the service installation record.

### Step 6 — Cleanup
- Deleted both test accounts (`testuser1`, `backdoor_svc`) using `net user /delete`.
- Deleted the test service using `sc.exe delete WinUpdateHelper`.
- Verified clean account list via `net user`.

---

## Key Methodology Principles

- **Verify before simulate** — check what the machine is actually logging before assuming.
- **One step at a time** — each stage of the attack chain was isolated, logged, and captured before moving to the next.
- **Follow the evidence** — when Event 4697 didn't appear, the investigation paused to find the root cause rather than skipping forward.
- **Cleanup after exercise** — no test accounts or dummy services left on the machine after the investigation.
