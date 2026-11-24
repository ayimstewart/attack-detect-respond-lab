# 00: Lab threat model and safety controls

**Method:** STRIDE, applied to each data flow in the lab
**Last reviewed:** November 21, 2026

A lab built for attacking machines is itself a risk. Attack traffic can leak onto my home network, a test tool can reach a real device, and screenshots can leak details to the internet. I wrote this threat model before running any attacks, so the safety controls exist first.

## Scope

In scope: the three lab VMs, the two hosts they run on, the network between them, and this public repository.
Out of scope: anything I don't own. No testing ever leaves the lab network.

## Trust boundaries

```
┌──────────────────── Home network (untrusted to the lab) ─────────────────────┐
│   phones, TVs, other devices · default gateway · internet                    │
└───────────────▲──────────────────────────────────────────────────────────────┘
                │ temporary internet access for updates only (NAT / Shared Network)
┌───────────────┴──────────── Lab network (isolated) ──────────────────────────┐
│                                                                              │
│  MacBook (host)              ROG laptop (host)                               │
│  ┌──────────────┐            ┌──────────────────┐   ┌─────────────────────┐  │
│  │ Kali attacker │──attacks──▶ Windows 11 victim │──▶│ Ubuntu + Wazuh SIEM │  │
│  └──────────────┘            └──────────────────┘   └──────────┬──────────┘  │
│                                                                │ dashboard   │
└────────────────────────────────────────────────────────────────┼─────────────┘
                                                                 ▼
                                                  Analyst (me) on the ROG host
```

## Data flows

| ID | Flow | Crosses a boundary? |
|---|---|---|
| F1 | Kali to Windows victim: attack traffic | Yes, between the two physical hosts |
| F2 | Windows victim to Wazuh: agent logs (TCP 1514) and enrollment (TCP 1515) | No, inside the ROG |
| F3 | ROG host browser to Wazuh dashboard (HTTPS 443) | Yes, host to VM |
| F4 | VMs to the internet for updates | Yes, lab to home network |
| F5 | Screenshots and write-ups to this public GitHub repo | Yes, private to public |

## Threats

| Flow | STRIDE | Threat | Risk | Mitigation | Status |
|---|---|---|---|---|---|
| F1 | Elevation of privilege | An attack tool hits a real device because the target IP or subnet was mistyped | High | Lab subnet has no default gateway and no route to the home network. Scan targets are written as single IPs from the lab IP plan, never whole ranges | Planned |
| F1 | Tampering | Malware or exploit code escapes the victim VM to the ROG host | High | VMware shared folders, drag-and-drop and clipboard turned off on the victim. Snapshot taken before every exercise and restored after | Planned |
| F2 | Spoofing | A rogue machine enrolls as a fake agent and sends false logs | Medium | Agent enrollment protected with a password (Wazuh `authd`). Only the victim's IP is allowed to enroll | Planned |
| F2 | Information disclosure | Logs read in transit | Low | Wazuh agent traffic is encrypted by default. Accepted | Accepted |
| F2 | Repudiation | Attacker clears the Windows event log to hide activity | Medium | Logs are forwarded to Wazuh as they happen, so a cleared local log doesn't erase the SIEM copy. Log clearing (Event ID 1102) gets its own alert | Planned |
| F3 | Spoofing | Default Wazuh admin password used to log in to the dashboard | High | Default password changed at install. Dashboard reachable only from the ROG host, with no port forwarding | To verify |
| F3 | Denial of service | Wazuh indexer runs out of memory or disk during a noisy exercise | Medium | 8 GB RAM and 80 GB disk allocated. Check disk use before each exercise | Accepted |
| F4 | Tampering | A compromised update or ISO is installed | Medium | Hypervisor and ISOs checked against published SHA-256 hashes and signatures before install | Done |
| F4 | Elevation of privilege | A VM left on NAT after updating can reach the home network during an attack | High | Attack exercises start only after the attacker and victim are moved to the lab network. Checked with a ping to the home gateway, which must fail | Planned |
| F5 | Information disclosure | Screenshots leak passwords, real IPs, hostnames or work and Army details | High | Check every screenshot before committing. Lab uses throwaway passwords only. Nothing work- or Army-related in the lab or the repo | In place |

## Residual risks

- **Physical link between the hosts.** The Mac and the ROG still have Wi-Fi radios. Accepted for now. Before attack exercises I'll confirm the VMs' traffic goes only over the lab link. Adding a pfSense firewall VM is on the roadmap to enforce this.
- **Evaluation Windows licence.** The victim expires after the evaluation period. Accepted. The VM is rebuilt from the documented steps when needed.
- **Single analyst.** I'm the only reviewer of my own detections. Accepted for a home lab. Write-ups are public so others can check them.
