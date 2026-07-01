# Case 2 — TCP Handshake and HTTP Connection Analysis

## Scenario

A web request was made from an internal workstation to the Ubuntu web server in the controlled lab environment.

The purpose of this case is to inspect the TCP connection establishment process and confirm that HTTP request and response data can be identified at packet level.

## Objective

Analyze a focused HTTP connection and identify the TCP three-way handshake, HTTP request, HTTP response, and plaintext web content.

## Analyst Question

Can the TCP three-way handshake and HTTP request/response be identified in the packet capture?

## PCAP File

```text
pcaps/02-tcp-handshake-http.pcap
```

## Lab Environment

| Asset              | Hostname      | Role                               | IP Address     |
| ------------------ | ------------- | ---------------------------------- | -------------- |
| Source host        | attacker-kali | HTTP client / traffic generator    | 192.168.100.10 |
| Capture sensor     | analyzer-kali | Packet capture and analysis sensor | 192.168.100.20 |
| Destination server | victim-ubuntu | Apache web server                  | 192.168.100.30 |

## Tools Used

| Tool      | Purpose                                    |
| --------- | ------------------------------------------ |
| tcpdump   | Captured HTTP traffic from the analyzer VM |
| Wireshark | Inspected TCP and HTTP packet details      |
| curl      | Generated HTTP HEAD and GET requests       |
| Apache2   | Served the web page from the victim server |

## Capture Command

The packet capture was performed from `analyzer-kali` on the internal lab interface `eth1`.

```bash
sudo tcpdump -i eth1 -nn -w ~/pcaps/02-tcp-handshake-http.pcap 'host 192.168.100.30 and tcp port 80'
```

## Traffic Generation Commands

The following commands were executed from `attacker-kali` to generate HTTP traffic toward the Ubuntu Apache web server.

```bash
curl -I http://192.168.100.30
curl http://192.168.100.30
```

## Capture Summary

| Field              | Value          |
| ------------------ | -------------- |
| Packets captured   | 20             |
| Snapshot length    | 262144 bytes   |
| Packets dropped    | 0              |
| Source IP          | 192.168.100.10 |
| Destination IP     | 192.168.100.30 |
| Destination port   | TCP/80         |
| Protocols observed | TCP, HTTP      |

## Wireshark Filters Used

```text
tcp.port == 80
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.flags.syn == 1 && tcp.flags.ack == 1
http.request
http.response
ip.addr == 192.168.100.10 && ip.addr == 192.168.100.30
```

## Evidence Collected

| Evidence                        | Screenshot                                                                 |
| ------------------------------- | -------------------------------------------------------------------------- |
| TCP/80 overview                 | `screenshots/02-tcp-handshake-http/01-tcp-port-80-overview.png`            |
| TCP SYN packet                  | `screenshots/02-tcp-handshake-http/02-tcp-syn.png`                         |
| TCP SYN/ACK packet              | `screenshots/02-tcp-handshake-http/03-tcp-syn-ack.png`                     |
| TCP handshake and HTTP sequence | `screenshots/02-tcp-handshake-http/04-tcp-handshake-and-http-sequence.png` |
| HTTP request                    | `screenshots/02-tcp-handshake-http/05-http-request.png`                    |
| HTTP response                   | `screenshots/02-tcp-handshake-http/06-http-response-200-ok.png`            |
| HTTP Follow TCP Stream          | `screenshots/02-tcp-handshake-http/07-http-follow-tcp-stream.png`          |

## Packet-Level Findings

### Finding 1 — TCP Three-Way Handshake Was Observed

The capture shows a normal TCP three-way handshake between the client and the web server.

The observed sequence was:

```text
192.168.100.10 → 192.168.100.30  SYN
192.168.100.30 → 192.168.100.10  SYN, ACK
192.168.100.10 → 192.168.100.30  ACK
```

This confirms that the client successfully established a TCP connection to the Apache web server on TCP port 80 before HTTP data was exchanged.

### Finding 2 — HTTP Request Was Sent in Plaintext

The packet capture shows HTTP request traffic from the client to the server.

The request was generated using `curl` and included either a `HEAD /` request, a `GET /` request, or both depending on the observed packet. This demonstrates that HTTP application-layer request data can be inspected directly in Wireshark.

### Finding 3 — HTTP 200 OK Response Was Returned

The victim server responded successfully to the HTTP request.

The HTTP response included a successful status code:

```text
HTTP/1.1 200 OK
```

This indicates that the Apache web server was reachable and served the requested content successfully.

### Finding 4 — HTTP Content Was Readable in Follow TCP Stream

Using Wireshark's Follow TCP Stream feature, the HTTP request and response content could be viewed in readable text.

This confirms that HTTP traffic is not encrypted and can expose request headers, server headers, and page content to anyone with packet visibility on the network path.

## Timeline

| Activity                    | Description                                                 |
| --------------------------- | ----------------------------------------------------------- |
| Capture started             | tcpdump capture started on analyzer-kali `eth1`             |
| HTTP HEAD request generated | `curl -I http://192.168.100.30` executed from attacker-kali |
| HTTP GET request generated  | `curl http://192.168.100.30` executed from attacker-kali    |
| Capture stopped             | tcpdump capture stopped after HTTP traffic generation       |
| PCAP verified               | Saved PCAP confirmed to contain TCP/80 traffic              |

## MITRE ATT&CK Mapping

No direct MITRE ATT&CK technique is assigned for this case.

This activity represents normal web client-to-server communication and protocol analysis. MITRE ATT&CK mapping is not necessary unless the HTTP traffic is part of suspicious behavior, such as command-and-control, data exfiltration, or malicious web access.

## Conclusion

The packet capture successfully shows a complete TCP connection to the Ubuntu Apache web server followed by HTTP request and response traffic.

The analysis confirmed the TCP three-way handshake process and demonstrated that HTTP traffic can be inspected in plaintext using Wireshark.

This case supports the project goal by showing packet-level understanding of TCP connection establishment and web protocol analysis.

## Recommended Actions

* Use HTTPS/TLS instead of plaintext HTTP for sensitive web applications.
* Treat unencrypted HTTP as unsuitable for transmitting credentials, session tokens, or confidential data.
* In real investigations, review HTTP headers, requested URIs, user agents, response codes, and transferred objects for suspicious behavior.
* Use this case as a foundation before analyzing suspicious HTTP patterns in later scenarios.

## Limitations

This capture was intentionally small and focused on HTTP traffic only.

The activity was generated in a controlled lab and does not represent a full production web browsing session.

The capture confirms plaintext HTTP visibility, but it does not indicate malicious activity by itself.

Additional logs such as web server access logs, proxy logs, DNS logs, and endpoint telemetry would provide more context in a real investigation.
