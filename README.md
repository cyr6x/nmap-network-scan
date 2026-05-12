# Nmap Network Reconnaissance Lab
**Port Scanning | Service Enumeration | OS Fingerprinting | Attack Surface Analysis**

> Structured network reconnaissance lab using Nmap against a Kali Linux localhost and QEMU virtual gateway — simulating the initial scanning phase of a penetration test or vulnerability assessment.

---

## Lab Environment

| Component | Details |
|---|---|
| **Tool** | Nmap 7.98 |
| **Platform** | Kali Linux 2024 (Debian-based, VirtualBox/QEMU) |
| **Network Mode** | NAT |
| **Target 1** | 127.0.0.1 — Kali localhost |
| **Target 2** | 10.0.2.2 — QEMU virtual gateway (host machine) |
| **Privileges** | Standard user + `sudo` for OS/service detection |

> ⚠️ All scans performed against self-owned targets in an isolated virtual environment. No unauthorized systems were accessed.

---

## Objectives

- Perform progressive Nmap scans from basic discovery to full aggressive fingerprinting
- Identify open ports and enumerate running services with version disclosure
- Fingerprint the operating system of both targets
- Assess the attack surface exposed through NAT mode
- Map findings to real-world vulnerabilities and security controls

---

## Scan Summary

| Step | Command | Target | Purpose |
|---|---|---|---|
| 1 | `nmap 127.0.0.1` | Localhost | Basic port discovery |
| 2 | `sudo nmap -sV 127.0.0.1` | Localhost | Service version detection |
| 3 | `sudo nmap -O 127.0.0.1` | Localhost | OS fingerprinting |
| 4 | `sudo nmap -A 127.0.0.1` | Localhost | Aggressive full scan |
| 5 | `nmap 10.0.2.2` | Gateway | Host machine attack surface |

---

## Step 1 — Basic Port Scan (`nmap 127.0.0.1`)

```bash
nmap 127.0.0.1
```

**Findings:**
```
PORT   STATE SERVICE
22/tcp open  ssh
999 closed TCP ports (reset)
Scan time: 0.08s
```

- SSH service manually started: `sudo service ssh start`
- Port 22 immediately visible once the service was enabled
- All other ports closed — minimal attack surface

**Security Interpretation:**  
A basic scan reveals exactly what an unauthenticated attacker sees first. With only SSH running, the attack surface is limited to one service — but that one open port is enough for brute-force, credential stuffing, or version-specific exploits.

![Basic Nmap Scan — Port 22 Open](screenshots/01-ssh-start-basic-scan.png)

---

## Step 2 — Service Version Detection (`sudo nmap -sV 127.0.0.1`)

```bash
sudo nmap -sV 127.0.0.1
```

**Findings:**
```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 10.2p1 Debian 5 (protocol 2.0)
```

- Exact software version exposed: **OpenSSH 10.2p1**
- OS hint: **Debian Linux**
- Protocol: **SSH v2.0**

**Security Interpretation:**  
Version disclosure is a critical misconfiguration. An attacker cross-references `OpenSSH 10.2p1` against CVE databases to find known vulnerabilities specific to that build. In production, SSH banners should be suppressed via `DebianBanner no` in `sshd_config`.

![Nmap -sV — Service Version Disclosure](screenshots/02-nmap-sV-service-version.png)

---

## Step 3 — OS Detection (`sudo nmap -O 127.0.0.1`)

```bash
sudo nmap -O 127.0.0.1
```

**Findings:**
```
Device type:      general purpose
Running:          Linux 2.6.X|5.X
OS details:       Linux 2.6.32, Linux 5.0 - 6.2
Network Distance: 0 hops
```

**Security Interpretation:**  
OS fingerprinting reveals the kernel version range, which attackers use to select kernel-specific privilege escalation CVEs. The broad match (`2.6.X|5.X`) is typical of virtualized environments where the TCP/IP stack signature is partially obscured by the hypervisor.

![Nmap -O — OS Fingerprinting](screenshots/03-nmap-O-os-detection.png)

---

## Step 4 — Aggressive Scan (`sudo nmap -A 127.0.0.1`)

```bash
sudo nmap -A 127.0.0.1
```

**Findings:**
```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 10.2p1 Debian 5 (protocol 2.0)
OS CPE: cpe:/o:linux:linux_kernel
OS details: Linux 2.6.32, Linux 5.0 - 6.2
Scan time: 1.74s
```

- `-A` combines: `-sV` (versions) + `-O` (OS) + `--traceroute` + NSE script scanning
- Most information-rich single scan — used in CTFs and professional pen tests
- Scan time increased from 0.08s (basic) to 1.74s

**Security Interpretation:**  
Aggressive scanning gives a complete picture in one pass. Any IDS/IPS monitoring for port scans will almost certainly flag `-A` due to its volume and variety of probe types. In a real engagement this would be a deliberate, post-stealth-recon step.

![Nmap -A — Aggressive Full Scan](screenshots/04-nmap-A-aggressive.png)

---

## Step 5 — Gateway Scan (`nmap 10.0.2.2`)

```bash
nmap 10.0.2.2
```

**Findings:**
```
PORT     STATE     SERVICE
135/tcp  open      msrpc
445/tcp  open      microsoft-ds
902/tcp  open      iss-realsecure
912/tcp  open      apex-mesh
5357/tcp open      wsdapi
995 filtered ports (no-response)
MAC Address: 52:54:00:12:35:00 (QEMU virtual NIC)
Scan time: 6.41s
```

**Port Risk Assessment:**

| Port | Service | Risk | Notes |
|---|---|---|---|
| 135 | MSRPC | 🟡 Medium | RPC endpoint mapper — DCE/RPC attack vector |
| 445 | Microsoft-DS (SMB) | 🔴 High | EternalBlue (MS17-010), WannaCry, NotPetya |
| 902 | ISS RealSecure | 🟡 Medium | VMware authentication port |
| 912 | Apex-Mesh | 🟡 Medium | VMware remote console |
| 5357 | WSDAPI | 🟠 Low-Medium | Web Services Discovery — info leakage risk |

> ⚠️ **Port 445 (SMB) is the most critical finding.** This is the exact attack vector exploited by EternalBlue (MS17-010) — the vulnerability behind WannaCry and NotPetya. On any production network, SMB must be blocked at the perimeter firewall and patched.

![Nmap Gateway Scan — Ports 135, 445, 902, 912, 5357](screenshots/05-nmap-gateway-10.0.2.2.png)

---

## Findings Summary

| Target | Open Ports | Key Services | Risk Level |
|---|---|---|---|
| 127.0.0.1 (localhost) | 22 | OpenSSH 10.2p1 | 🟡 Low-Medium |
| 10.0.2.2 (gateway) | 135, 445, 902, 912, 5357 | SMB, MSRPC, VMware | 🔴 Medium-High |

---

## Attack Timeline (Simulated)

| Phase | Action | Finding |
|---|---|---|
| Discovery | `nmap 127.0.0.1` | Port 22 open — SSH running |
| Enumeration | `nmap -sV` | OpenSSH 10.2p1 Debian confirmed |
| Fingerprinting | `nmap -O` | Linux kernel 5.x confirmed |
| Full recon | `nmap -A` | Complete service + OS profile assembled |
| Lateral scope | `nmap 10.0.2.2` | Host machine exposed via NAT gateway |
| Risk assessment | Manual analysis | Port 445 SMB flagged as highest priority target |

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence |
|---|---|---|---|
| Reconnaissance | Active Scanning: Port Scan | T1595.001 | All 5 Nmap scans |
| Reconnaissance | Gather Victim Host Info: OS | T1592.001 | `nmap -O` OS fingerprint |
| Reconnaissance | Gather Victim Host Info: Software | T1592.002 | `nmap -sV` version disclosure |
| Discovery | Network Service Discovery | T1046 | Gateway scan (10.0.2.2) |
| Discovery | System Network Connections | T1049 | Open ports enumerated |
| Initial Access | Exploit Public-Facing Application | T1190 | Port 445 SMB — EternalBlue vector |

---

## Defender Takeaways

### Detection Signals

| Signal | Detection Logic |
|---|---|
| **Port scan burst** | >50 SYN packets to sequential ports from single source in <5s — IDS rule |
| **Nmap fingerprint probes** | NSE scripts generate unusual TCP flag combinations (FIN, NULL, XMAS) — SIEM alert |
| **SMB exposure (445)** | Firewall rule: block 445 inbound from all non-management IPs |
| **MSRPC (135) exposure** | Should never be internet-facing — alert on any external SYN to 135 |
| **Version banner disclosure** | OpenSSH returning full version string — suppress with `DebianBanner no` |

### Preventive Controls

| Control | How It Helps |
|---|---|
| **Host-based firewall (`ufw`)** | `ufw enable` on Kali — blocks unsolicited inbound probes |
| **Suppress SSH banner** | `DebianBanner no` in `/etc/ssh/sshd_config` — hides version string |
| **Block SMB at perimeter** | Firewall drop rule on port 445 — eliminates EternalBlue attack vector |
| **Patch MS17-010** | Apply KB4012212 — closes EternalBlue on any exposed Windows host |
| **IDS/IPS deployment** | Snort/Suricata rules: `alert tcp any any -> any any (flags:S; threshold: type threshold, track by_src, count 50, seconds 5;)` |
| **Disable unused services** | Only run SSH when actively needed — `sudo service ssh stop` when idle |

---

## Academic & Professional Context

| Context | Mapping |
|---|---|
| **CompTIA Security+ SY0-701** | Domain 4.1 — Network security controls; Domain 3.3 — Vulnerability scanning |
| **NIST SP 800-115** | Technical Guide to Information Security Testing — network scanning methodology |
| **Pen Test Phases** | This lab covers Phase 2 (Scanning & Enumeration) of the standard pentest lifecycle |
| **SOC Relevance** | Baseline port scan output is used to validate firewall rules and detect rogue services |

---

## Status

- [x] Lab environment configured on Kali (NAT mode)
- [x] All 5 scan steps completed
- [x] Service version enumeration documented
- [x] OS fingerprinting completed
- [x] Gateway attack surface assessed
- [x] Port risk assessment completed
- [x] MITRE ATT&CK mapping finalised
- [x] Defender takeaways documented
- [ ] Screenshots captured and pushed to `screenshots/`

---

*Lab conducted in an authorized, isolated virtual environment. All targets are self-owned. No unauthorized systems were accessed.*
