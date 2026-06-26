# Network Traffic Analysis & Incident Investigation Lab

## Project Overview

This project is a controlled network traffic analysis lab designed to demonstrate packet capture, Wireshark analysis, tcpdump usage, and SOC-style investigation reporting.

The goal is not only to capture packets, but to explain network activity using packet-level evidence. Each investigation case includes a scenario, analyst question, traffic generation method, capture command, Wireshark filters, key findings, screenshots, limitations, and recommended actions.

## Lab Environment

The lab uses a controlled three-VM setup in VirtualBox.

| VM            | Operating System        | Role                                     | Lab IP Address |
| ------------- | ----------------------- | ---------------------------------------- | -------------- |
| attacker-kali | Kali Linux              | Traffic generation workstation           | 192.168.100.10 |
| analyzer-kali | Kali Linux              | Packet capture and analysis sensor       | 192.168.100.20 |
| victim-ubuntu | Ubuntu Server 24.04 LTS | Target server hosting SSH, HTTP, and FTP | 192.168.100.30 |

Each VM uses two network adapters:

| Adapter   | Mode             | Purpose                                              |
| --------- | ---------------- | ---------------------------------------------------- |
| Adapter 1 | NAT              | Internet access for updates and package installation |
| Adapter 2 | Internal Network | Isolated lab traffic between the three VMs           |

The internal lab network is used for all investigation traffic.

## Tools Used

| Tool       | Purpose                                           |
| ---------- | ------------------------------------------------- |
| Wireshark  | Manual packet inspection and evidence screenshots |
| tcpdump    | Packet capture from the analyzer VM               |
| tshark     | Optional command-line packet analysis             |
| Nmap       | Controlled scan traffic generation                |
| curl       | HTTP request generation                           |
| ping       | ICMP traffic generation                           |
| ftp        | Plaintext FTP traffic generation                  |
| ssh        | Encrypted remote access traffic generation        |
| Apache2    | HTTP service on the victim server                 |
| OpenSSH    | SSH service on the victim server                  |
| vsftpd     | FTP service on the victim server                  |
| VirtualBox | Virtual lab environment                           |

## Investigation Cases

| Case | Title                                                   | Status    |
| ---: | ------------------------------------------------------- | --------- |
|    1 | Normal Traffic Baseline                                 | Completed |
|    2 | TCP Handshake and HTTP Connection Analysis              | Planned   |
|    3 | DNS Investigation                                       | Planned   |
|    4 | Nmap Port Scan Investigation                            | Planned   |
|    5 | ICMP Ping Sweep Investigation                           | Planned   |
|    6 | Plaintext FTP Credential Exposure                       | Planned   |
|    7 | SSH vs FTP Comparison                                   | Planned   |
|    8 | Suspicious Repeated Connection / Beaconing-like Traffic | Planned   |

## Completed Work

### Case 1 — Normal Traffic Baseline

The first investigation case captured normal baseline traffic between the attacker workstation and the Ubuntu victim server.

Observed traffic included:

* ARP resolution
* ICMP echo request and reply
* HTTP request and response traffic
* SSH session traffic
* FTP control traffic

The capture confirmed that the lab environment is working and that the analyzer VM can observe traffic between the attacker and victim systems.

Case 1 report:

```text
reports/01-normal-traffic-baseline.md
```

Case 1 PCAP:

```text
pcaps/01-normal-traffic-baseline.pcap
```

Case 1 screenshots:

```text
screenshots/01-normal-traffic-baseline/
```

## Repository Structure

```text
network-traffic-investigation-lab/
│
├── README.md
├── lab-setup.md
├── tools-used.md
├── scope-and-limitations.md
│
├── docs/
│   ├── environment-overview.md
│   ├── vm-network-map.md
│   ├── capture-methodology.md
│   ├── analyst-report-template.md
│   └── troubleshooting-notes.md
│
├── pcaps/
│   ├── README.md
│   └── 01-normal-traffic-baseline.pcap
│
├── reports/
│   └── 01-normal-traffic-baseline.md
│
├── screenshots/
│   ├── 00-lab-setup/
│   └── 01-normal-traffic-baseline/
│
├── filters/
│   └── wireshark-filters.md
│
├── commands/
│   ├── attacker-commands.md
│   ├── analyzer-capture-commands.md
│   └── victim-service-commands.md
│
└── notes/
    ├── lessons-learned.md
    └── limitations.md
```

## Current Status

The lab environment has been configured and validated. Case 1 has been captured, analyzed, and documented.

Next planned case:

```text
Case 2 — TCP Handshake and HTTP Connection Analysis
```

## Notes

This project uses only controlled lab traffic. No real credentials, production systems, or third-party network traffic are used.

FTP credential exposure is demonstrated only with fake lab credentials for educational purposes.
