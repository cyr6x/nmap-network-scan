# 🔍 Nmap Network Reconnaissance Lab

![Security](https://img.shields.io/badge/Security-Ethical_Hacking-red?style=flat-square)
![Tool](https://img.shields.io/badge/Tool-Nmap_7.98-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Screenshots_Pending-yellow?style=flat-square)

Hands-on Nmap reconnaissance lab against a Kali Linux localhost and QEMU virtual gateway. Covers port scanning, service version detection, OS fingerprinting, and attack surface analysis.

## Contents

| File | Description |
|---|---|
| [`lab-report-nmap.md`](lab-report-nmap.md) | Full lab report with findings, analysis & recommendations |
| [`screenshots/`](screenshots/) | Terminal captures for all 5 scan steps |

## Scans Performed

| Step | Command | Purpose |
|---|---|---|
| 1 | `nmap 127.0.0.1` | Basic port scan |
| 2 | `sudo nmap -sV 127.0.0.1` | Service version detection |
| 3 | `sudo nmap -O 127.0.0.1` | OS fingerprinting |
| 4 | `sudo nmap -A 127.0.0.1` | Aggressive full scan |
| 5 | `nmap 10.0.2.2` | Gateway (host machine) scan |

## Key Findings

- Port 22 (OpenSSH 10.2p1) identified on localhost once SSH started
- Gateway exposed ports 135, 445, 902, 912, 5357
- Port 445 (SMB) flagged as highest risk — EternalBlue attack vector

## Screenshots

> Add terminal captures to the `screenshots/` folder:

- `screenshots/01-ssh-start-basic-scan.png`
- `screenshots/02-nmap-sV-service-version.png`
- `screenshots/03-nmap-O-os-detection.png`
- `screenshots/04-nmap-A-aggressive.png`
- `screenshots/05-nmap-gateway-10.0.2.2.png`

---

*Lab conducted in an authorized, isolated virtual environment. All targets are self-owned.*
