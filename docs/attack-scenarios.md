# Attack scenarios and detection plan

Each exercise in this lab maps to a technique in [MITRE ATT&CK](https://attack.mitre.org/). For each one I plan the attack, decide which log should catch it, and then check whether Wazuh actually alerts. A scenario only moves to **Detected** after I've run it and the alert shows up with evidence.

| # | Scenario | ATT&CK technique | Tool (from Kali) | Expected evidence on the victim | Status |
|---|---|---|---|---|---|
| 1 | Network service scan | [T1046](https://attack.mitre.org/techniques/T1046/) Network Service Discovery | Nmap | Windows Filtering Platform events (Security 5156/5157) once firewall auditing is turned on | Planned |
| 2 | Password guessing against SMB | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) Brute Force: Password Guessing | Hydra | Repeated failed logons, Security 4625 (logon type 3) | Planned |
| 3 | Login with a guessed password | [T1078](https://attack.mitre.org/techniques/T1078/) Valid Accounts | NetExec | A successful logon (4624) right after a run of 4625s from the same source IP | Planned |
| 4 | Remote PowerShell command | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) Command and Scripting Interpreter: PowerShell | Evil-WinRM | Sysmon Event ID 1 (process creation) and PowerShell Script Block Logging, Event ID 4104 | Planned |
| 5 | Backdoor local account | [T1136.001](https://attack.mitre.org/techniques/T1136/001/) Create Account: Local Account | `net user` over the remote session | Security 4720 (account created) and 4732 (added to a local group) | Planned |
| 6 | Scheduled task persistence | [T1053.005](https://attack.mitre.org/techniques/T1053/005/) Scheduled Task | `schtasks` over the remote session | Security 4698 (task created) and Sysmon Event ID 1 for `schtasks.exe` | Planned |
| 7 | Covering tracks | [T1070.001](https://attack.mitre.org/techniques/T1070/001/) Clear Windows Event Logs | `wevtutil cl` | Security 1102 (audit log cleared). The SIEM copy survives because logs were already forwarded | Planned |

## How each scenario gets written up

Each one gets its own page in `docs/` once it has been run:

1. **Setup:** snapshot taken, VMs on the lab network, gateway ping fails.
2. **Attack:** the exact commands run from Kali, with timestamps.
3. **Detection:** the Wazuh alert, its rule, and the raw Windows event behind it.
4. **Response:** following the [incident response workflow](workflows.md): contain, recover from the snapshot, and note what I'd change.
5. **Tuning:** if Wazuh missed it, the custom rule I wrote and proof that it fires.

## Before any of this runs

- [ ] Windows 11 victim built, with Sysmon and the Wazuh agent installed
- [ ] Advanced audit policy turned on for logon events, account management, scheduled tasks and Filtering Platform connections
- [ ] PowerShell Script Block Logging turned on
- [ ] Attacker and victim moved to the isolated lab network (see the [threat model](00-threat-model.md))
