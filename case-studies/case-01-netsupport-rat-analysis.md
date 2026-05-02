# network-traffic-analysis

> Real-world PCAP investigations focused on malware analysis, network forensics, IOC extraction, incident response, and threat hunting.

---

## Project Overview

This repository contains hands-on packet capture (PCAP) investigations performed on real-world network traffic samples.

The purpose of this project is to simulate practical **Security Operations Center (SOC)** workflows by analyzing suspicious network traffic, identifying malicious behavior, extracting indicators of compromise (IOCs), and documenting findings in structured case reports.

Each case study follows an investigation methodology similar to blue team incident response and malware traffic analysis workflows.

---

## Repository Structure

```bash
network-traffic-analysis/
│
├── README.md
├── case-studies/
│   ├── case-01-netsupport-rat-analysis.md
│
├── pcap-files/
│   ├── README.md
│   └── sample-file.pcap
│
├── screenshots/
│   ├── protocol-hierarchy.png
│   ├── dns-analysis.png
│   ├── http-analysis.png
│
├── iocs/
│   ├── malicious_ips.txt
│   ├── malicious_domains.txt
│   ├── file_hashes.txt
│
└── notes/
    ├── useful-filters.md
```

---

## Core Objectives

- Analyze suspicious PCAP traffic
- Detect malware communications
- Investigate phishing and credential theft traffic
- Identify command-and-control (C2) infrastructure
- Extract and validate IOCs
- Document incident findings

---

## Technical Skills Demonstrated

### Network Analysis
- Packet Analysis
- Protocol Analysis
- Endpoint Analysis
- Conversation Analysis
- TCP Stream Analysis

### Security Investigation
- Malware Traffic Analysis
- Network Forensics
- Incident Investigation
- IOC Extraction
- Threat Hunting

### Protocols Analyzed
- DNS
- HTTP / HTTPS
- TCP
- UDP
- TLS
- ICMP

---

## Tools & Technologies

- Wireshark
- Tshark
- NetworkMiner
- VirusTotal
- CyberChef
- WHOIS
- URLHaus
- AbuseIPDB

---

## Investigation Workflow

1. Load PCAP into Wireshark
2. Review Protocol Hierarchy
3. Analyze Endpoints & Conversations
4. Inspect DNS Activity
5. Review HTTP/HTTPS Requests
6. Follow TCP Streams
7. Export Files/Objects
8. Extract Indicators of Compromise
9. Validate Findings with Threat Intelligence
10. Document Final Incident Report

---

## Case Studies

| Case ID | Investigation | Focus Area | Status |
|---------|--------------|-----------|--------|
| Case 01 | NetSupport RAT Traffic Analysis | Malware Investigation | In Progress |

---

## Current Case Study

### Case 01: NetSupport RAT Traffic Analysis
Investigation of suspicious traffic associated with NetSupport Manager RAT activity identified through SIEM alerts.

Key focus:
- infected host identification
- malicious external IP analysis
- outbound RAT communication analysis
- IOC extraction

Case report:
- `case-studies/case-01-netsupport-rat-analysis.md`

---

## IOC Categories

This repository collects:
- Malicious IP addresses
- Domains
- URLs
- File hashes
- User agents
- Payload indicators

---

## Screenshots Preview

Add investigation screenshots inside `/screenshots`.

Examples:
- Protocol Hierarchy
- DNS Queries
- HTTP Requests
- TCP Stream Analysis

---

## PCAP Files

PCAP files are stored inside `/pcap-files`.

Note:
- Some traffic samples are sourced from external training platforms.
- Original traffic samples belong to their respective owners.

---

## Traffic Sample Source

Traffic analysis exercises referenced from:

https://www.malware-traffic-analysis.net/

---

## Framework Alignment

Analysis findings may be mapped to:
- MITRE ATT&CK
- Incident Response Lifecycle
- Threat Intelligence Enrichment

---

## Learning Focus

This repository is maintained to strengthen practical skills in:

- SOC Operations
- Blue Team Analysis
- Malware Detection
- Threat Intelligence
- Network Defense

---

## Connect With Me

- GitHub: YOUR_GITHUB_LINK
- LinkedIn: YOUR_LINKEDIN_LINK
