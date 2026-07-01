# Case 4 — Nmap Port Scan Investigation

## Scenario

An internal Ubuntu server received a high number of TCP connection attempts from a single internal host across multiple destination ports.

The purpose of this case is to determine whether the traffic represents normal service access or reconnaissance-like port scanning activity.

## Objective

Identify port scanning behavior using packet-level evidence in Wireshark.

## Analyst Question

Was the traffic normal service access or a port scan?

## PCAP File

```text id="ijcwak"
pcaps/04-nmap-portscan-investigation.pcap
```

## Lab Environment

| Asset              | Hostname      | Role                               | IP Address     |
| ------------------ | ------------- | ---------------------------------- | -------------- |
| Source host        | attacker-kali | Scan source                        | 192.168.100.10 |
| Capture sensor     | analyzer-kali | Packet capture and analysis sensor | 192.168.100.20 |
| Destination server | victim-ubuntu | Scanned server                     | 192.168.100.30 |

## Tools Used

| Tool            | Purpose                                                |
| --------------- | ------------------------------------------------------ |
| tcpdump         | Captured traffic from the analyzer VM                  |
| Wireshark       | Analyzed TCP flags, conversations, and packet behavior |
| Nmap            | Generated controlled scan traffic                      |
| Ubuntu services | Provided open ports for scan results                   |

## Capture Command

The packet capture was performed from `analyzer-kali` on the internal lab interface `eth1`.

```bash id="7nggag"
sudo tcpdump -i eth1 -nn -w ~/pcaps/04-nmap-portscan-investigation.pcap host 192.168.100.30
```

## Traffic Generation Commands

The following commands were executed from `attacker-kali`.

```bash id="9te3kz"
nmap -sV -p 21,22,80 192.168.100.30
sudo nmap -sS -p 1-1000 192.168.100.30
```

The first command checked known service ports. The second command generated a wider SYN scan across TCP ports 1 to 1000.

## Capture Summary

| Field                             | Value                         |
| --------------------------------- | ----------------------------- |
| Packets captured                  | 2116                          |
| Snapshot length                   | 262144 bytes                  |
| Packets dropped                   | 0                             |
| Source IP                         | 192.168.100.10                |
| Destination IP                    | 192.168.100.30                |
| Known open service ports observed | 21, 22, 80                    |
| Scan behavior observed            | SYN packets to multiple ports |
| RST responses observed            | Yes                           |

## Wireshark Filters Used

```text id="9hnjzl"
ip.src == 192.168.100.10 && ip.dst == 192.168.100.30
ip.src == 192.168.100.10 && ip.dst == 192.168.100.30 && tcp.flags.syn == 1 && tcp.flags.ack == 0
ip.src == 192.168.100.30 && ip.dst == 192.168.100.10 && tcp.flags.syn == 1 && tcp.flags.ack == 1
ip.src == 192.168.100.30 && ip.dst == 192.168.100.10 && tcp.flags.reset == 1
tcp.port == 21 || tcp.port == 22 || tcp.port == 80
```

## Evidence Collected

| Evidence                                | Screenshot                                                             |
| --------------------------------------- | ---------------------------------------------------------------------- |
| Scan overview                           | `screenshots/04-nmap-portscan/01-scan-overview.png`                    |
| SYN packets to multiple ports           | `screenshots/04-nmap-portscan/02-syn-packets-to-multiple-ports.png`    |
| SYN/ACK responses from open ports       | `screenshots/04-nmap-portscan/03-open-port-syn-ack-responses.png`      |
| RST responses from closed ports         | `screenshots/04-nmap-portscan/04-closed-port-rst-responses.png`        |
| Known service ports                     | `screenshots/04-nmap-portscan/05-known-open-service-ports.png`         |
| TCP conversations across multiple ports | `screenshots/04-nmap-portscan/06-tcp-conversations-multiple-ports.png` |
| Expert information                      | `screenshots/04-nmap-portscan/07-expert-information.png`               |

## Packet-Level Findings

### Finding 1 — One Source Contacted Multiple Destination Ports

The capture shows traffic from `192.168.100.10` to `192.168.100.30` across many TCP destination ports.

This pattern is different from normal single-service access, where a client usually connects to one expected service such as TCP/80 for HTTP or TCP/22 for SSH.

The number of attempted ports and the short time window make the activity consistent with port scanning.

### Finding 2 — SYN Packets Were Sent to Multiple Ports

The scan generated TCP SYN packets from the source host to many destination ports on the victim server.

A SYN packet with no ACK flag set usually indicates the start of a TCP connection attempt.

The repeated SYN packets to many ports are consistent with network reconnaissance behavior because the source host is attempting to discover which services are reachable.

### Finding 3 — SYN/ACK Responses Indicated Open Ports

The victim server responded with SYN/ACK packets for open ports.

In this case, known service ports such as FTP, SSH, and HTTP were expected to be open:

```text id="ygzkj3"
TCP/21  FTP
TCP/22  SSH
TCP/80  HTTP
```

A SYN/ACK response indicates that the destination port is open and willing to complete a TCP connection.

### Finding 4 — RST Responses Indicated Closed or Rejected Ports

The capture also contains TCP reset responses.

RST packets are commonly observed when a connection attempt is made to a closed port or when a connection is rejected.

This supports the conclusion that the source host attempted to connect to multiple ports, some open and many closed.

### Finding 5 — TCP Conversations Showed Repeated Connection Attempts

The TCP conversations view showed multiple TCP interactions between the same source and destination.

This is useful because it summarizes the scan pattern at a higher level and supports the packet-level evidence seen in the SYN, SYN/ACK, and RST packets.

## Timeline

| Activity                 | Description                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------- |
| Capture started          | tcpdump capture started on analyzer-kali `eth1`                                     |
| Known port scan executed | `nmap -sV -p 21,22,80 192.168.100.30` executed                                      |
| Wider SYN scan executed  | `sudo nmap -sS -p 1-1000 192.168.100.30` executed                                   |
| Capture stopped          | tcpdump capture stopped after scan activity                                         |
| PCAP verified            | Saved PCAP confirmed to contain SYN packets, known service ports, and RST responses |

## MITRE ATT&CK Mapping

| Technique | Name                      | Reason                                                                                                   |
| --------- | ------------------------- | -------------------------------------------------------------------------------------------------------- |
| T1046     | Network Service Discovery | The source host attempted to discover open services on the victim server by scanning multiple TCP ports. |

## Conclusion

The traffic is consistent with port scanning activity.

The key evidence is that one internal source host sent TCP SYN packets to many destination ports on the Ubuntu server within a short time window. The victim server responded with SYN/ACK packets for open ports and RST packets for closed or rejected ports.

This behavior matches network service discovery because the source system attempted to determine which services were available on the destination host.

## Recommended Actions

In a real environment, the following actions would be recommended:

* Confirm whether the scan was authorized.
* Identify the owner and purpose of the source host.
* Review endpoint logs on the source system for the process that initiated the scan.
* Review server logs on the destination host for follow-up activity.
* Check whether the source host scanned other internal systems.
* Apply firewall or network segmentation controls where appropriate.
* Monitor for repeated scanning or escalation attempts.

## Limitations

Port scanning behavior does not automatically prove malicious intent. Authorized vulnerability scans, asset discovery, and administrative checks can produce similar traffic.

Packet capture alone shows network behavior, but additional context is needed before escalation. Useful supporting evidence would include change tickets, vulnerability scan schedules, endpoint process logs, authentication logs, and firewall logs.

This scan was generated intentionally in a controlled lab using Nmap.
