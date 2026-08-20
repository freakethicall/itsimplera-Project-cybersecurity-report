# itsimplera-Project-cybersecurity-report
# 🔐 Nimbus Retail Co. — Vulnerability Assessment

A complete **Cybersecurity Vulnerability Assessment & Penetration Testing (VAPT) project** conducted as part of the IT-SIMPLERA Institute Week 7 & 8 assignment.

The project simulates a real-world security assessment for **Nimbus Retail Co.**, a fictional e-commerce company preparing to launch its platform. The assessment covers the complete security-review lifecycle, from network architecture and reconnaissance to vulnerability identification, risk analysis, remediation, and incident response.

> **⚠️ Educational / Authorized Security Testing**
>
> This project was performed only against instructor-approved targets in an isolated lab environment. No unauthorized systems were scanned or exploited. `hackthissite.org` was used for reconnaissance methodology, while vulnerability assessment and exploitation-related testing were performed against a local **Metasploitable 2** virtual machine.

---

## 📌 Project Overview

**Nimbus Retail Co.** represents an e-commerce organization whose infrastructure requires security validation before production deployment.

The assessment was divided into multiple phases:

| Phase       | Focus                                       |
| ----------- | ------------------------------------------- |
| **Phase 1** | Security foundations & network architecture |
| **Phase 2** | OSINT & reconnaissance                      |
| **Phase 3** | Enumeration & network/service scanning      |
| **Phase 4** | Vulnerability assessment & risk analysis    |
| **Phase 5** | Remediation & incident response             |

The assessment identified multiple critical security weaknesses in the lab environment, including vulnerable services capable of remote code execution.

---

## 🎯 Objectives

The primary objectives of this project were to:

* Design a secure e-commerce network architecture
* Apply the **CIA Triad** to real-world security incidents
* Analyze network communication using the **OSI 7-layer model**
* Develop an access-control policy based on **least privilege**
* Perform passive OSINT and reconnaissance
* Enumerate subdomains and discover live hosts
* Identify exposed services and technologies
* Perform network and service enumeration using Nmap
* Perform banner grabbing and service-version identification
* Map discovered services to known CVEs
* Calculate and analyze CVSS risk
* Build a likelihood × impact risk matrix
* Review TLS/HTTPS security
* Develop remediation recommendations
* Create an incident response plan

---

# 🏗️ Network Architecture

The proposed Nimbus Retail Co. architecture uses network segmentation to separate public-facing services from sensitive internal systems.

### DMZ

`192.168.10.0/24`

Contains internet-facing services such as:

* Reverse Proxy / Load Balancer — `192.168.10.5`
* Web Server — `192.168.10.10`

### Internal Network

`192.168.20.0/24`

Contains sensitive internal resources:

* Database Server — `192.168.20.10`
* Internal File Server — `192.168.20.20`
* Employee Workstations — `192.168.20.50-100`

The architecture follows a **default-deny and least-privilege approach**. Internet traffic is restricted to HTTP/HTTPS toward the DMZ, while communication from the DMZ to the internal network is limited to the database service.

---

# 🔎 Reconnaissance & OSINT

The reconnaissance phase focused on understanding the external attack surface associated with `hackthissite.org`.

### Techniques Used

* WHOIS enumeration
* Certificate Transparency investigation
* Google Dorking
* Subdomain enumeration
* Live host verification
* Visual reconnaissance
* theHarvester
* Shodan

Example Google dork categories included:

```text
filetype:pdf
filetype:xml
inurl:admin
intitle:"index of"
inurl:login
ext:log
inurl:backup
filetype:sql
```

Certificate Transparency lookup through `crt.sh` was attempted but encountered persistent `502 Bad Gateway` errors during the assessment, so subdomain discovery was supplemented using Subfinder and Amass.

---

# 🌐 Subdomain Enumeration

Two passive enumeration tools were used:

### Subfinder

Subfinder identified:

**67 subdomains**

### OWASP Amass

Amass was used to expand the passive discovery process.

After combining and deduplicating results:

> **334 unique subdomains were identified.**

This significantly exceeded the project's required minimum and provided a broader view of the potential external attack surface.

---

# 🗺️ Attack Surface Mapping

The discovered attack surface was categorized into several groups:

### 🟢 Live Web Assets

Examples:

```text
www.hackthissite.org
forums.hackthissite.org
h5ai.hackthissite.org
ctf.hackthissite.org
status.hackthissite.org
```

### 🔵 IRC Infrastructure

```text
irc.hackthissite.org
irc-hub.hackthissite.org
wolf.irc.hackthissite.org
lille.irc.hackthissite.org
```

### 🟠 CDN / Staging

```text
cdn.hackthissite.org
v3stage-cdn.hackthissite.org
v3dev-cdn.hackthissite.org
```

### 🔴 Infrastructure / Firewall

```text
vm-099.outbound.firewall.hackthissite.org
vm-150.outbound.firewall.hackthissite.org
ns1.hackthissite.org
ns2.hackthissite.org
```

### 🚨 Admin / Sensitive Exposure

Higher-priority assets included hostnames referencing:

```text
admin.hackthissite.org
git.hackthissite.org
mirror.hackthissite.org
```

These were flagged for additional investigation because administrative interfaces, Git repositories, mirrors, staging environments, and infrastructure-related hosts can represent particularly sensitive exposure.

---

# 🔬 Enumeration & Scanning

The vulnerability assessment was performed against:

```text
Metasploitable 2
IP: 192.168.177.133
```

Multiple Nmap scan techniques were compared.

### TCP SYN Scan

```bash
nmap -sS -p- -T4 <TARGET>
```

Used for fast full-port TCP discovery.

### TCP Connect Scan

```bash
nmap -sT -p- <TARGET>
```

Used for full TCP scanning using complete connections.

### UDP Scan

```bash
nmap -sU --top-ports 100 <TARGET>
```

Used to identify commonly exposed UDP services.

### Version & Script Scan

```bash
nmap -sV -sC -p- -T4 <TARGET>
```

Used to identify:

* Service versions
* Default NSE script results
* Additional service information

The version/script scan provided the greatest level of service detail, while the SYN scan was the fastest full-range TCP discovery method.

---

# 🧪 Banner Grabbing

Banner grabbing was performed using **Netcat** to verify service versions.

Example findings included:

| Port | Service | Version       |
| ---: | ------- | ------------- |
|   21 | FTP     | vsftpd 2.3.4  |
|   22 | SSH     | OpenSSH 4.7p1 |
|   25 | SMTP    | Postfix smtpd |

The banner information was then used to support service identification and vulnerability mapping.

---

# 🌐 Web Technology Analysis

**WhatWeb** and direct browser inspection were used to fingerprint the web environment.

The assessment identified an Apache HTTP server hosting deliberately vulnerable applications including:

* DVWA
* Mutillidae
* TWiki
* Apache Tomcat

Based on the observed technologies and applications, the following OWASP Top 10 categories were considered particularly relevant:

* **A01 — Broken Access Control**
* **A03 — Injection**
* **A05 — Security Misconfiguration**
* **A06 — Vulnerable and Outdated Components**

---

# 🚨 Vulnerability Findings

Four major vulnerabilities were mapped from the discovered services.

| Service            | Vulnerability         | CVE           |              CVSS |
| ------------------ | --------------------- | ------------- | ----------------: |
| vsftpd 2.3.4       | Backdoor              | CVE-2011-2523 | **10.0 Critical** |
| Samba 3.X          | usermap_script RCE    | CVE-2007-2447 | **10.0 Critical** |
| UnrealIRCd 3.2.8.1 | Backdoor              | CVE-2010-2075 | **10.0 Critical** |
| distccd            | Remote Code Execution | CVE-2004-2687 |  **9.8 Critical** |

The assessment determined that these services could allow unauthorized remote command execution or shell access under the vulnerable configurations.

---

# 📊 Risk Assessment

The vulnerabilities were evaluated using **Likelihood × Impact**.

| Vulnerability            | Likelihood | Impact   | Risk            |
| ------------------------ | ---------- | -------- | --------------- |
| vsftpd 2.3.4 Backdoor    | High       | Critical | 🔴 **Critical** |
| Samba usermap_script RCE | High       | Critical | 🔴 **Critical** |
| UnrealIRCd Backdoor      | Medium     | Critical | 🟠 **High**     |
| distccd RCE              | Medium     | High     | 🟠 **High**     |

Likelihood was based on how easily a vulnerable service could be discovered and reached, while impact considered the potential consequences of successful exploitation.

---

# 🔐 TLS / Cryptography Review

The Metasploitable 2 environment did not provide HTTPS on port `443`.

Consequently, the hosted web applications were accessible through unencrypted HTTP, potentially exposing:

* Credentials
* Session cookies
* Submitted information
* Application traffic

The report recommends that the production Nimbus Retail platform enforce:

```text
TLS 1.2 / TLS 1.3
Valid certificates
HTTP → HTTPS redirection
HSTS
```

before production deployment.

---

# 🛠️ Remediation

The assessment provided the following remediation priorities:

### vsftpd

* Remove the compromised/backdoored binary
* Upgrade to a current verified release
* Validate package integrity

### Samba

* Upgrade Samba
* Disable `username map script` unless required
* Validate configuration

### UnrealIRCd

* Remove or replace the vulnerable version
* Use verified current builds
* Obtain software from trusted official sources

### distccd

* Disable the service where unnecessary
* Restrict it to trusted internal build infrastructure

### HTTPS

* Deploy a valid TLS certificate
* Redirect HTTP to HTTPS
* Enable HSTS

### Excessive Subdomains

* Review the **334 discovered subdomains**
* Remove unnecessary public exposure
* Restrict staging/development infrastructure
* Decommission unused services

These recommendations were classified primarily as immediate or pre-go-live priorities.

---

# 🚑 Incident Response

An incident response scenario was developed around exploitation of the **vsftpd 2.3.4 backdoor** followed by potential customer-data theft.

The response process follows four stages:

```text
1. Detect
      ↓
2. Contain
      ↓
3. Eradicate
      ↓
4. Recover
```

### Detect

* Monitor suspicious connections
* Identify abnormal shell activity
* Detect unusual outbound data transfers
* Confirm and declare the incident

### Contain

* Isolate affected systems
* Revoke exposed credentials
* Block attacker infrastructure
* Disable vulnerable services

### Eradicate

* Remove malicious/backdoored software
* Remove persistence mechanisms
* Patch affected systems
* Search for additional compromise

### Recover

* Restore from known-clean backups
* Verify systems before reconnecting
* Monitor for reinfection
* Notify affected parties where required
* Conduct a post-incident review

---

# 🧰 Tools Used

| Tool                   | Purpose                                  |
| ---------------------- | ---------------------------------------- |
| **Kali Linux**         | Security testing environment             |
| **Nmap**               | Network & service enumeration            |
| **Subfinder**          | Passive subdomain enumeration            |
| **OWASP Amass**        | Passive attack-surface discovery         |
| **httpx**              | Live host verification                   |
| **Gowitness**          | Visual reconnaissance                    |
| **theHarvester**       | OSINT collection                         |
| **Shodan**             | Internet-facing infrastructure discovery |
| **WhatWeb**            | Web technology fingerprinting            |
| **Netcat**             | Banner grabbing                          |
| **NVD**                | CVE research & vulnerability mapping     |
| **VMware Workstation** | Isolated lab virtualization              |

---

# 🖥️ Lab Environment

```text
Attacker
└── Kali Linux
    ├── Native Dual-Boot
    └── zsh

Target 1 — Reconnaissance
└── hackthissite.org

Target 2 — Vulnerability Assessment
└── Metasploitable 2
    └── 192.168.177.133

Virtualization
└── VMware Workstation
    ├── Host-Only Networking
    └── NAT Networking
```

The report identifies Kali as the attacker environment and Metasploitable 2 as the isolated vulnerability-assessment target.

---

# 📁 Project Structure

A recommended repository structure for this project is:

```text
Nimbus-Retail-Vulnerability-Assessment/
│
├── README.md
├── report/
│   └── Nimbus-Retail-Vulnerability-Assessment.pdf
│
├── diagrams/
│   ├── network-architecture.png
│   └── attack-surface-map.png
│
├── reconnaissance/
│   ├── subfinder/
│   ├── amass/
│   ├── httpx/
│   ├── gowitness/
│   ├── theharvester/
│   └── shodan/
│
├── scanning/
│   ├── nmap/
│   └── banner-grabbing/
│
├── vulnerability-assessment/
│   ├── cve-mapping/
│   ├── cvss/
│   └── risk-matrix/
│
└── incident-response/
    └── incident-response-plan.md
```

---

# 📈 Key Results

### Attack Surface

**334 unique subdomains** discovered through combined passive enumeration.

### Critical Vulnerabilities

**4 major vulnerabilities** identified:

* CVE-2011-2523
* CVE-2007-2447
* CVE-2010-2075
* CVE-2004-2687

### Highest CVSS

**10.0 / Critical**

### Security Posture

The assessment concluded that the simulated Nimbus Retail environment would have a **High-to-Critical security risk level** if the identified weaknesses were present in production. The two Critical findings were recommended as **blocking conditions before production launch**.

---

# 🎓 Learning Outcomes

This project provided practical experience with:

* Network security architecture
* DMZ and network segmentation
* Firewall policy design
* CIA Triad
* OSI model analysis
* Passive reconnaissance
* OSINT
* Subdomain enumeration
* Attack-surface mapping
* Network scanning
* Service enumeration
* Banner grabbing
* Web technology fingerprinting
* CVE research
* CVSS scoring
* Risk assessment
* TLS security
* Security remediation
* Incident response

---

# ⚖️ Disclaimer

This repository is intended **strictly for educational and authorized security-testing purposes**.

All vulnerability-assessment activities were performed against the instructor-approved **Metasploitable 2** laboratory environment. Reconnaissance activities were performed against the project-approved `hackthissite.org` target for demonstrating OSINT and reconnaissance methodology.

**Do not scan, enumerate, exploit, or otherwise test systems without explicit authorization.**

---

# 👨‍💻 Author

**Abdul Moiz Zahoor**

🎓 Cybersecurity Student
🏫 IT-SIMPLERA Institute
🐙 GitHub: [github.com/freakethicall](https://github.com/freakethicall)
💼 LinkedIn: [linkedin.com/in/abdul-moiz-zahoor-419933341](https://linkedin.com/in/abdul-moiz-zahoor-419933341)

**Project:** Nimbus Retail Co. — Vulnerability Assessment
**Academic Session:** 2026
**Submission:** August 2026

---

## ⭐ Project Summary

> **Nimbus Retail Co. — Vulnerability Assessment** demonstrates a complete security assessment workflow, combining secure network architecture, OSINT, reconnaissance, enumeration, vulnerability identification, CVSS-based risk analysis, TLS review, remediation planning, and incident response within an authorized cybersecurity laboratory environment.
