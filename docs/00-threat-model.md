# 00: Lab threat model and safety controls

**Method:** STRIDE, applied to each data flow in the lab
**Last reviewed:** November 28, 2026 (updated for the home Wi-Fi attack path)

A lab built for attacking machines is itself a risk. Attack traffic can leak onto my home network, a test tool can reach a real device, and screenshots can leak details to the internet. I wrote this threat model before running any attacks, so the safety controls exist first.

## What changed on November 28

The original plan kept every VM on an isolated lab network. That turned out not to be possible: the Mac has no Ethernet port and the two laptops have no wired link. The attack path now crosses my home Wi-Fi, so I re-ran the threat model for that design.

- Kali stays on UTM's Shared Network and reaches the home network through the Mac's NAT.
- The Windows victim got a second network adapter, bridged to the ROG's Wi-Fi. That adapter is the attack target and is connected only during exercises.
- Wazuh stays on the ADR-Lab segment only. It is never bridged.

## Scope

In scope: the three lab VMs, the two hosts they run on, the networks between them, and this public repository.
Out of scope: anything I don't own. Other devices on my home network are never targeted.

## Trust boundaries

```
┌──────────────────── Home Wi-Fi (shared with other devices) ───────────────────┐
│   phones, TVs, other devices · default gateway · internet                     │
│                                                                               │
│   MacBook (host)                         ROG laptop (host)                    │
│   ┌──────────────┐   NAT via the Mac     ┌───────────────────────────┐       │
│   │ Kali attacker │──── attack traffic ──▶│ NIC 2: bridged, DHCP IP   │       │
│   └──────────────┘                       │   Windows 11 victim       │       │
│                                          └────────────┬──────────────┘       │
└───────────────────────────────────────────────────────┼──────────────────────┘
                                                        │ NIC 1
┌──────────────────── ADR-Lab segment (isolated, 10.77.77.0/24) ───────────────┐
│                                          ┌────────────▼──────────────┐       │
│   no gateway, no route out               │ Ubuntu + Wazuh SIEM       │       │
│                                          └────────────┬──────────────┘       │
└───────────────────────────────────────────────────────┼──────────────────────┘
                                                        ▼ dashboard
                                         Analyst (me), from the victim VM
```

## Data flows

| ID | Flow | Crosses a boundary? |
|---|---|---|
| F1 | Kali to the victim's bridged adapter: attack traffic over home Wi-Fi | Yes, between the two physical hosts across the home network |
| F2 | Victim to Wazuh over ADR-Lab: agent logs (TCP 1514) and enrollment (TCP 1515) | No, inside the ROG |
| F3 | Browser to the Wazuh dashboard (HTTPS 443) over ADR-Lab | No, inside the ROG |
| F4 | VMs to the internet for updates | Yes, lab to home network |
| F5 | Screenshots and write-ups to this public GitHub repo | Yes, private to public |
| F6 | Other home devices to the victim's bridged adapter | Yes, home network into the lab |

## Threats

| Flow | STRIDE | Threat | Risk | Mitigation | Status |
|---|---|---|---|---|---|
| F1 | Elevation of privilege | An attack tool hits someone else's device because a range or the wrong IP was scanned | High | Every scan targets the victim's single home IP, checked with `ipconfig` first because DHCP can change it. Never a /24 | In place |
| F1 | Tampering | Malware or exploit code escapes the victim VM to the ROG host | High | VMware shared folders, drag-and-drop and clipboard turned off on the victim. Snapshot taken before every exercise and restored after | Clipboard off, verified. Snapshot before every exercise: to make a habit |
| F6 | Elevation of privilege | The victim, deliberately weakened for exercises, is reachable by every device on the home network | High | Bridged adapter unticked ("Connected" off) except during an exercise. Windows Firewall stays on: a 1,000-port scan showed every port filtered. The only inbound exception is ICMP echo, for testing reachability | In place |
| F6 | Elevation of privilege | A compromised victim attacks other home devices | Medium | Exercises that leave the victim compromised are reverted to a snapshot before the bridged adapter is reconnected. Pre-bridge snapshot kept as a known-good restore point | Snapshot exists. Revert rule starts with Scenario 2 |
| F2 | Spoofing | A rogue machine enrolls as a fake agent and sends false logs | Medium | Wazuh and the agent channel stay on ADR-Lab, which has no gateway and no bridge. Enrollment password (`authd`) planned | Partly in place |
| F2 | Information disclosure | Logs read in transit | Low | Wazuh agent traffic is encrypted by default. Accepted | Accepted |
| F2 | Repudiation | Attacker clears the Windows event log to hide activity | Medium | Logs are forwarded to Wazuh as they happen, so a cleared local log doesn't erase the SIEM copy. Log clearing (Event ID 1102) gets its own alert | Planned (Scenario 7) |
| F3 | Spoofing | Default Wazuh admin password used to log in to the dashboard | High | Default password changed at install. Dashboard lives only on ADR-Lab and is never bridged or port-forwarded | To verify |
| F3 | Denial of service | Wazuh indexer runs out of memory or disk during a noisy exercise | Medium | 8 GB RAM and 80 GB disk allocated. Check disk use before each exercise. The ROG has 16 GB, so Splunk waits for a RAM upgrade | Accepted |
| F4 | Tampering | A compromised update or ISO is installed | Medium | Hypervisor and ISOs checked against published SHA-256 hashes and signatures before install | Done |
| F5 | Information disclosure | Screenshots leak passwords, public IPs, hostnames or work and Army details | High | Check every screenshot before committing. Only private lab and home-LAN addresses appear. Lab uses throwaway passwords only. Nothing work- or Army-related in the lab or the repo | In place |

## Residual risks

- **Attack traffic shares the home Wi-Fi.** Other people's devices see the victim while the bridged adapter is connected, and they could see scan traffic. Accepted for now, because exercises are short and targeted at one IP. A pfSense VM or a USB Ethernet link between the hosts would remove this, and both are on the roadmap.
- **Home-network noise reaches the victim.** Other devices' SSDP and LLMNR broadcasts show up in the victim's logs. Accepted, and useful practice: detections are written to ignore it (for example, the port-scan rule only counts TCP).
- **Evaluation Windows licence.** The victim expires after the evaluation period. Accepted. The VM is rebuilt from the documented steps when needed.
- **Single analyst.** I'm the only reviewer of my own detections. Accepted for a home lab. Write-ups are public so others can check them.
