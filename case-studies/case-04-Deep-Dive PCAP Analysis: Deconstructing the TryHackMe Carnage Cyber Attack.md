# Threat Investigation & Wireshark Analysis Report – TryHackMe Carnage

## Overview

This repository documents the investigation of the **TryHackMe: Carnage** challenge using **Wireshark**. The objective was to analyze a malicious packet capture (`carnage.pcap`) and reconstruct the complete attack chain, from the initial malware delivery through post-compromise communication.

### Scenario

Eric Fischer from the Purchasing Department at **Bartell Ltd.** received a Microsoft Word document from a trusted contact. After opening the document, he clicked **"Enable Content"**, allowing a malicious macro to execute. Shortly afterward, the Security Operations Center (SOC) received an alert from the endpoint security agent, prompting a network traffic investigation.

The analysis focuses on identifying:

- Initial infection vector
- Malicious file downloads
- Multi-stage payload delivery
- Command and Control (C2) infrastructure
- Post-infection communication
- Malware reconnaissance
- Malicious email activity

---

# Investigation Summary

| Category | Finding |
|----------|----------|
| Initial Access | Malicious HTTP download |
| Malware Delivery | ZIP archive containing malicious Excel file |
| Initial Domain | attirenepal.com |
| C2 Framework | Cobalt Strike |
| Post-Infection Traffic | HTTP POST beaconing |
| Victim IP Check | api.ipify.org |
| Email Activity | SMTP Malspam Campaign |

---

# Step 1 – Initial HTTP Infection

### Objective
Identify the first HTTP request made by the infected host.

### Wireshark Filter

```text
http
```

### Investigation

Locate the earliest HTTP GET request from the victim.

### Findings

| Question | Answer |
|----------|--------|
| First HTTP connection | **2021-09-24 16:44:38** |
| Downloaded ZIP file | **documents.zip** |
| Hosting Domain | **attirenepal.com** |

---

# Step 2 – ZIP Payload Analysis

### Objective

Determine the file contained inside the downloaded ZIP archive without extracting it.

### Method

Follow the TCP stream containing the HTTP download.

```
Right Click Packet
→ Follow
→ TCP Stream
```

### Findings

| Question | Answer |
|----------|--------|
| File inside ZIP | **chart-1530076591.xls** |
| Web Server | **LiteSpeed** |
| Server Version | **PHP/7.2.34** |

---

# Step 3 – Multi-Stage Malware Download

### Objective

Identify additional domains contacted after the initial infection.

### Wireshark Filter

```text
dns and frame.time >= "Sep 24, 2021 16:45:11" &&
frame.time <= "Sep 24, 2021 16:45:30"
```

### Findings

Malicious domains:

- finejewels.com.au
- thietbiagt.com
- new.americold.com

### SSL Certificate

| Question | Answer |
|----------|--------|
| Certificate Authority | **GoDaddy** |

---

# Step 4 – Cobalt Strike Command & Control

### Objective

Identify the Cobalt Strike infrastructure.

### Method

```
Statistics
→ Conversations
→ TCP
```

Pivot suspicious IP addresses using VirusTotal.

### Findings

| Question | Answer |
|----------|--------|
| C2 Server 1 | **185.106.96.158** |
| C2 Server 2 | **185.125.204.174** |
| Host Header | **ocsp.verisign.com** |
| Domain (IP 1) | **survmeter.live** |
| Domain (IP 2) | **securitybusinpuff.com** |

---

# Step 5 – Post-Infection Traffic

### Objective

Analyze beaconing traffic after compromise.

### Wireshark Filter

```text
http.request.method == "POST"
```

### Findings

| Question | Answer |
|----------|--------|
| Domain | **maldivehost.net** |
| First 11 Characters Sent | **zLIisQRWZI9** |
| First Packet Length | **281 Bytes** |
| Server Header | **Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4** |

---

# Step 6 – Malware Reconnaissance

### Objective

Determine whether the malware checked the victim's public IP.

### Wireshark Filter

```text
dns.qry.name contains "api"
```

### Findings

| Question | Answer |
|----------|--------|
| DNS Query Time | **2021-09-24 17:00:04 UTC** |
| Domain | **api.ipify.org** |

---

# Step 7 – SMTP Malspam Activity

### Objective

Investigate email traffic associated with the attack.

### Wireshark Filter

```text
smtp
```

### Findings

| Question | Answer |
|----------|--------|
| First MAIL FROM | **farshin@mailfa.com** |
| SMTP Packets | **1439** |

---

# Key Wireshark Filters Used

```text
http

http.request.method == "GET"

http.request.method == "POST"

dns

dns.qry.name contains "api"

smtp

ip.addr == 185.106.96.158 and http

tls.handshake.type == 11

frame.time >= "Sep 24, 2021 16:45:11" &&
frame.time <= "Sep 24, 2021 16:45:30"
```

---

# Attack Flow

```
Malspam Email
      │
      ▼
Word Document
      │
      ▼
Enable Content (Macro)
      │
      ▼
HTTP GET
      │
      ▼
documents.zip
      │
      ▼
chart-1530076591.xls
      │
      ▼
Additional Malware Downloads
      │
      ▼
Cobalt Strike C2
      │
      ▼
HTTP POST Beaconing
      │
      ▼
Public IP Check
      │
      ▼
Persistent Communication
```

---

# Skills Demonstrated

- Network Traffic Analysis
- Wireshark Packet Investigation
- HTTP Analysis
- DNS Analysis
- SMTP Traffic Analysis
- TCP Stream Analysis
- TLS Certificate Inspection
- Malware Traffic Analysis
- Command & Control Detection
- Threat Hunting
- IOC Identification
- Digital Forensics
- Incident Response

---

# Tools Used

- Wireshark
- VirusTotal
- TryHackMe
- TCP Stream Analysis

---

# Learning Outcomes

This investigation demonstrates how Wireshark can be used to reconstruct an entire malware infection chain by correlating HTTP, DNS, SMTP, TLS, and TCP traffic. Through packet analysis, it was possible to identify the initial compromise, trace multi-stage payload delivery, uncover Cobalt Strike infrastructure, observe post-infection beaconing, detect reconnaissance activity, and validate malicious email traffic, providing a complete view of the incident from infection to command-and-control communication.
