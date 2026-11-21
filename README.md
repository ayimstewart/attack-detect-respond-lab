# Attack, Detect, Respond: Virtual Home Security Lab

A virtual security lab I'm building from scratch to practise the full defensive cycle: **simulate an attack, detect it in a SIEM, and respond to it**, with every step documented.

> **Status:** 🔨 In progress. Infrastructure is being built. No attack simulations yet.

---

## 🎯 Goals

- Build an isolated lab with a dedicated attacker, victim and SIEM
- Collect Windows endpoint telemetry with Sysmon and send it to Wazuh
- Run controlled attacks from Kali Linux and write detections for them
- Document each incident like a real SOC analyst: what happened, how it was detected, and how to respond

## 🗺️ Architecture

| Host | Role | Platform | Software |
|---|---|---|---|
| MacBook Air (Apple M5, 16 GB) | **Attacker** | UTM (QEMU, ARM64) | Kali Linux 2026.3 |
| ASUS ROG laptop | **Defender** | VMware Workstation Pro | Ubuntu Server 24.04.5 LTS + Wazuh SIEM |
| ASUS ROG laptop | **Victim** | VMware Workstation Pro | Windows 11 Enterprise Evaluation + Sysmon *(planned)* |

The lab is designed to run on an isolated network, separate from my home network. Kali uses a temporary internet connection only for installation and updates.

```
 [ Kali Linux ]  ──── attacks ────►  [ Windows 11 + Sysmon ]
   (Mac / UTM)                              │
                                            │ logs (Wazuh agent)
                                            ▼
                                  [ Ubuntu Server + Wazuh ]
                                     alerts and detections
```

## ✅ Progress

| # | Phase | Status | Write-up |
|---|---|---|---|
| 01 | Attacker VM: Kali Linux on Apple Silicon | ✅ Installed | [docs/01-attacker-setup.md](docs/01-attacker-setup.md) |
| 02 | SIEM server: Ubuntu Server + Wazuh | ✅ Dashboard verified | *Write-up coming soon* |
| 03 | Isolated lab network | 🔨 In progress | |
| 04 | Victim VM: Windows 11 + Sysmon + Wazuh agent | ⏳ Planned | |
| 05 | First telemetry test (Sysmon → Wazuh) | ⏳ Planned | |
| 06 | First attack, detect and respond exercise | ⏳ Planned | |

## 🛠️ Skills demonstrated

Virtualization (VMware, UTM/QEMU) · Linux administration · SIEM deployment (Wazuh) · Endpoint telemetry (Sysmon) · Network isolation · Troubleshooting · Technical documentation

## ⚠️ Ethics and scope

All testing is performed only on machines I own, inside an isolated lab network. Nothing in this repository targets systems I don't own or have permission to test.

---

*Built and documented by [Stewart Ayim](https://github.com/ayimstewart): CompTIA Security+ · AWS Solutions Architect – Associate*
