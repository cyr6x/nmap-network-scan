# 🔍 Lab Report: Nmap Network Reconnaissance

> **Lab Type:** Network Scanning & Enumeration  
> **Environment:** Kali Linux VM (NAT Mode) — VirtualBox/QEMU  
> **Date:** May 7, 2026  
> **Author:** Cyril Baaya | ID: 205132  
> **Qualification:** L5 Diploma in Computing with Cybersecurity (NCC Education)  
> **Mapped to:** CompTIA Security+ SY0-701 — Domain 4.1 (Network Security), Domain 3.3 (Vulnerability Scanning)

---

## 🎯 Objective

Conduct structured network reconnaissance against a Kali Linux localhost and a QEMU virtual gateway using Nmap. Demonstrate the ability to identify open ports, enumerate service versions, fingerprint operating systems, and interpret findings in a security context.

---

## 🛠️ Tools & Environment

| Component | Detail |
|---|---|
| **OS** | Kali Linux 2024 (Debian-based) |
| **Tool** | Nmap 7.98 |
| **Network Mode** | NAT (VirtualBox/QEMU) |
| **Target 1** | 127.0.0.1 (localhost) |
| **Target 2** | 10.0.2.2 (QEMU virtual gateway) |
| **Privileges** | Standard + sudo for OS/service detection |

---

## 📋 Methodology

Scans were performed in progressive depth, from basic port discovery to full aggressive fingerprinting:

```
Step 1 → Basic port scan (localhost)         nmap 127.0.0.1
Step 2 → Service version detection           sudo nmap -sV 127.0.0.1  
Step 3 → OS detection                        sudo nmap -O 127.0.0.1
Step 4 → Aggressive full scan               sudo nmap -A 127.0.0.1
Step 5 → Gateway scan (external-facing)     nmap 10.0.2.2
```

---

## 📸 Scan Results & Analysis

> ⏳ **Screenshots in progress** — terminal captures for all 5 steps will be added to `screenshots/` shortly.

### Step 1 — SSH Service Started + Basic Scan (`nmap 127.0.0.1`)

![Basic Nmap Scan](screenshots/01-ssh-start-basic-scan.png)

**Findings:**
- SSH service was manually started: `sudo service ssh start`
- Port `22/tcp` → **open** → `ssh`
- 999 closed TCP ports (reset)
- Scan completed in **0.08 seconds**

**Security Interpretation:**  
With no services running, the attack surface is minimal. Once SSH was enabled, port 22 became visible — exactly how attackers identify entry points.

---

### Step 2 — Service Version Detection (`sudo nmap -sV 127.0.0.1`)

![Nmap -sV Service Scan](screenshots/02-nmap-sV-service-version.png)

**Findings:**
```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 10.2p1 Debian 5 (protocol 2.0)
```
- **Service:** OpenSSH 10.2p1
- **OS Hint:** Debian Linux
- **Protocol:** SSH v2.0

**Security Interpretation:**  
Version disclosure reveals the exact software build. An attacker can cross-reference this with CVE databases to find known vulnerabilities specific to OpenSSH 10.2p1.

---

### Step 3 — OS Detection (`sudo nmap -O 127.0.0.1`)

![Nmap -O OS Detection](screenshots/03-nmap-O-os-detection.png)

**Findings:**
```
Device type:  general purpose
Running:      Linux 2.6.X|5.X
OS details:   Linux 2.6.32, Linux 5.0 - 6.2
Network Distance: 0 hops
```

**Security Interpretation:**  
OS fingerprinting reveals the kernel version range. Attackers use this to select kernel-specific exploits (e.g., privilege escalation CVEs). The broad range `2.6.X|5.X` indicates Nmap matched multiple signatures — common with virtualized environments.

---

### Step 4 — Aggressive Scan (`sudo nmap -A 127.0.0.1`)

![Nmap -A Aggressive Scan](screenshots/04-nmap-A-aggressive.png)

**Findings:**
```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 10.2p1 Debian 5 (protocol 2.0)
OS CPE: cpe:/o:linux:linux_kernel
OS details: Linux 2.6.32, Linux 5.0 - 6.2
```
- Full fingerprint: service version + OS + traceroute combined
- Scan time: **1.74 seconds**

**Security Interpretation:**  
The `-A` flag combines `-sV`, `-O`, `--traceroute`, and script scanning. This is the most information-rich scan — it gives an attacker (or pen tester) a complete picture of the target in one pass.

---

### Step 5 — Gateway Scan (`nmap 10.0.2.2`)

![Nmap Gateway Scan](screenshots/05-nmap-gateway-10.0.2.2.png)

**Findings:**
```
PORT     STATE  SERVICE
135/tcp  open   msrpc
445/tcp  open   microsoft-ds
902/tcp  open   iss-realsecure
912/tcp  open   apex-mesh
5357/tcp open   wsdapi
MAC Address: 52:54:00:12:35:00 (QEMU virtual NIC)
```
- 995 filtered TCP ports (no-response)
- Host identified as **QEMU virtual NIC** (Windows host machine)
- Scan time: **6.41 seconds**

**Security Interpretation:**

| Port | Service | Risk |
|---|---|---|
| 135 | MSRPC | Medium — RPC endpoint mapper, common attack vector |
| 445 | Microsoft-DS (SMB) | **High** — SMB vulnerabilities (EternalBlue, WannaCry) |
| 902 | ISS RealSecure | Medium — VMware/security software port |
| 5357 | WSDAPI | Low-Medium — Web Services Discovery |

⚠️ **Port 445 (SMB) is the most significant finding.** This is the attack vector used in the EternalBlue exploit (MS17-010). On a production network, this port should be blocked at the perimeter firewall.

---

## 📊 Summary of Findings

| Target | Open Ports | Key Services | Risk Level |
|---|---|---|---|
| 127.0.0.1 (localhost) | 22 | OpenSSH 10.2p1 | 🟡 Low-Medium |
| 10.0.2.2 (gateway) | 135, 445, 902, 912, 5357 | SMB, MSRPC | 🔴 Medium-High |

---

## 🛡️ Recommendations

1. **Disable SSH when not in use** — `sudo service ssh stop` to minimize exposure
2. **Restrict SMB (445)** — Block at firewall unless explicitly required; patch MS17-010
3. **Disable MSRPC (135) externally** — Should never be internet-facing
4. **Enable host-based firewall** — `ufw enable` on Kali to control inbound connections
5. **Monitor with IDS** — Deploy Snort/Suricata rules to alert on Nmap-style scans

---

## 📚 Security+ SY0-701 Alignment

| Concept Demonstrated | Domain |
|---|---|
| Port scanning & enumeration | 4.1 — Apply network security controls |
| Service version fingerprinting | 4.3 — Implement network-based solutions |
| OS fingerprinting | 4.1 — Network reconnaissance |
| SMB vulnerability identification | 1.3 — Vulnerability types |
| Attack surface analysis | 2.5 — Vulnerability assessment techniques |

---

## 🔑 Key Takeaways

- **Progressive scanning** (basic → version → OS → aggressive) reveals increasingly sensitive information
- **NAT mode** exposes the host machine's services through the gateway IP (10.0.2.2)
- **Service version disclosure** is a critical misconfiguration — always configure SSH banners to hide version info in production
- **Nmap `-A` flag** is the go-to for comprehensive reconnaissance in CTFs and professional pen tests

---

## Status

- [x] Lab environment configured on Kali
- [x] All 5 scan steps completed
- [x] Findings documented and interpreted
- [x] Security+ SY0-701 alignment mapped
- [ ] Screenshots captured and pushed

---

*Lab conducted in an authorized, isolated virtual environment. All targets are self-owned. No unauthorized systems were accessed.*
