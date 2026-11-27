# Attack, Detect, Respond: Virtual Home Security Lab

![Status](https://img.shields.io/badge/status-in%20progress-orange)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-005571)
![Attacker](https://img.shields.io/badge/attacker-Kali%20Linux-557C94?logo=kalilinux&logoColor=white)
![Hypervisors](https://img.shields.io/badge/hypervisors-VMware%20%7C%20UTM-607078)
![Framework](https://img.shields.io/badge/mapped%20to-MITRE%20ATT%26CK-C0392B)

I'm building a home security lab from scratch to practise the full defensive cycle: run an attack, catch it in a SIEM, respond to it, and write it up the way a SOC analyst would. Every step is documented here, including what broke and how I fixed it.

> **Status:** The first Windows-to-Wazuh exercise is complete. I verified event collection, tested a custom rule, and triaged the resulting alert. [Read the exercise write-up](docs/02-first-lab-exercise.md). Kali-based exercises are planned.

![Lab architecture](diagrams/lab-architecture.svg)

## How a detection works in this lab

```mermaid
flowchart LR
    A["Kali Linux<br/>attacker"] -->|"attack traffic"| B["Windows 11<br/>victim"]
    B -->|"Sysmon + Windows<br/>event logs"| C["Wazuh agent"]
    C -->|"encrypted"| D["Wazuh manager<br/>rules + decoders"]
    D --> E["Dashboard alert"]
    E --> F["Analyst<br/>triage + respond"]

    classDef red fill:#F8CECC,stroke:#B85450,color:#000
    classDef blue fill:#DAE8FC,stroke:#6C8EBF,color:#000
    classDef green fill:#D5E8D4,stroke:#82B366,color:#000
    classDef yellow fill:#FFF2CC,stroke:#D6B656,color:#000
    class A red
    class B,C blue
    class D,E green
    class F yellow
```

## Progress

| # | Phase | Status | Write-up |
|---|---|---|---|
| 00 | Threat model and lab safety controls | Done | [00-threat-model.md](docs/00-threat-model.md) |
| 01 | Attacker VM: Kali Linux on Apple Silicon | Installed and updated | [01-attacker-setup.md](docs/01-attacker-setup.md) |
| 02 | SIEM server: Ubuntu Server + Wazuh | Operational | [First exercise](docs/02-first-lab-exercise.md) |
| 03 | VMware lab segment for Windows and Wazuh | Used for the first exercise | [First exercise](docs/02-first-lab-exercise.md) |
| 04 | Windows 11 + Sysmon + Wazuh agent | Active during the exercise | [First exercise](docs/02-first-lab-exercise.md) |
| 05 | First telemetry, detection, and triage exercise | Complete | [First exercise](docs/02-first-lab-exercise.md) |
| 06 | Attack exercises mapped to MITRE ATT&CK | Planned | [attack-scenarios.md](docs/attack-scenarios.md) |

## Planned attack coverage

| Technique | What it looks like | Detected? |
|---|---|---|
| T1046 Network Service Discovery | Nmap scan from Kali | Not yet run |
| T1110.001 Password Guessing | Hydra against SMB | Not yet run |
| T1078 Valid Accounts | Login after a successful guess | Not yet run |
| T1059.001 PowerShell | Remote commands over WinRM | Not yet run |
| T1136.001 Create Local Account | Backdoor user added | Not yet run |
| T1053.005 Scheduled Task | Persistence via `schtasks` | Not yet run |
| T1070.001 Clear Event Logs | `wevtutil cl` to hide tracks | Not yet run |

Full detail, including the Windows event IDs each one should produce, is in [attack-scenarios.md](docs/attack-scenarios.md). This table only changes when I've run the attack and have the alert to show for it.

## Repository layout

```
docs/          write-ups, threat model, attack plan, workflows
diagrams/      architecture diagram (SVG, plus the editable draw.io source)
screenshots/   evidence referenced from the write-ups
```

## Skills this lab covers

Virtualization (VMware, UTM/QEMU) · Linux administration · SIEM deployment (Wazuh) · Endpoint telemetry (Sysmon) · Threat modeling (STRIDE) · MITRE ATT&CK mapping · Incident response (NIST SP 800-61) · Network isolation · Technical documentation

## Ethics and scope

All testing runs only on machines I own, inside an isolated lab network. Nothing in this repository targets systems I don't own or have permission to test. The safety controls are in the [threat model](docs/00-threat-model.md).

---

Built and documented by [Stewart Ayim](https://github.com/ayimstewart) · CompTIA Security+ · AWS Solutions Architect – Associate
