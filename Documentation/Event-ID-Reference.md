# Event ID Reference — CASE-002

Quick reference for the Windows Security Event IDs used in this investigation.
All entries are from the **Security** log (`Windows Logs > Security` in Event Viewer).

---

| Event ID | Name | What it means | Why it matters in this case |
|---|---|---|---|
| **4625** | Failed Logon | An account failed to log on — wrong password, locked account, or non-existent account | 9 rapid failures against `testuser1` = brute-force pattern |
| **4624** | Successful Logon | An account successfully logged on | The success immediately after 9 failures confirmed initial access was achieved |
| **4720** | User Account Created | A new local user account was created | `backdoor_svc` created — attacker planting a hidden return path |
| **4732** | Member Added to Group | An account was added to a security-enabled local group | `backdoor_svc` added to Administrators — the privilege escalation moment |
| **4697** | Service Installed | A new service was installed on the system | `WinUpdateHelper` installed as LocalSystem — persistence established |
| **4740** | Account Locked Out | An account was locked out due to too many failed attempts | Not triggered in this case — lockout threshold was 10, simulation reached 9 failures |

---

## Additional Context

### Logon Types (inside Event 4624/4625)
| Type | Meaning | Seen in this case |
|---|---|---|
| 2 | Interactive (physical keyboard) | Yes — all logons in this simulation were local |
| 5 | Service logon | Yes — background SYSTEM account logons (normal, expected noise) |
| 10 | RemoteInteractive (RDP) | No — this would indicate a remote attacker |

### Key Fields to Check in Every Entry
- **Account Name** — who the event is about
- **Subject** — who performed the action (the "actor")
- **Logon Type** — how the logon happened
- **Source Network Address** — where the request came from (127.0.0.1 = local)
- **Logon ID** — unique session identifier, used to correlate a logon (4624) with its matching logoff (4634/4647)
