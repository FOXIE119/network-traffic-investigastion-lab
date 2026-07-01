# Case 3 — DNS Investigation

## Scenario

DNS lookups were performed from the analyzer VM to observe how domain names are resolved into IP addresses.

Unlike the internal victim-server cases, this DNS capture was collected from the analyzer VM's NAT interface because external DNS lookups were routed through the NAT interface rather than the isolated internal lab interface.

The purpose of this case is to identify DNS query and response packets, inspect queried domain names, and understand the limitations of using DNS traffic as investigation evidence.

## Objective

Analyze DNS query and response traffic at packet level.

## Analyst Question

What domain names were queried, and what DNS responses were returned?

## PCAP File

```text id="vlhgyd"
pcaps/03-dns-investigation.pcap
```

## Lab Environment

| Asset             | Hostname                           | Role                          | IP Address              |
| ----------------- | ---------------------------------- | ----------------------------- | ----------------------- |
| Source host       | analyzer-kali                      | DNS client and capture system | 10.0.2.x                |
| DNS resolver      | VirtualBox NAT / upstream resolver | DNS resolution path           | External / NAT-provided |
| Capture interface | analyzer-kali `eth0`               | NAT interface                 | 10.0.2.x                |

## Tools Used

| Tool      | Purpose                                   |
| --------- | ----------------------------------------- |
| tcpdump   | Captured DNS traffic from the analyzer VM |
| Wireshark | Inspected DNS packet details              |
| dig       | Generated DNS queries                     |
| nslookup  | Generated DNS queries                     |

## Capture Command

The packet capture was performed from `analyzer-kali` on the NAT interface `eth0`.

```bash id="eeo63u"
sudo tcpdump -i eth0 -nn -w ~/pcaps/03-dns-investigation.pcap 'udp port 53 or tcp port 53'
```

## Traffic Generation Commands

The following commands were executed from `analyzer-kali` to generate DNS lookup traffic.

```bash id="o4pjcq"
dig example.com
dig ubuntu.com
nslookup wireshark.org
```

## Capture Summary

| Field             | Value                |
| ----------------- | -------------------- |
| Packets captured  | 8                    |
| Packets dropped   | 0                    |
| Capture interface | analyzer-kali `eth0` |
| Traffic type      | DNS                  |
| Protocol/port     | UDP/53 or TCP/53     |

## Wireshark Filters Used

```text id="px4bvg"
dns
udp.port == 53
tcp.port == 53
dns.flags.response == 0
dns.flags.response == 1
dns.qry.name contains "example"
dns.qry.name contains "ubuntu"
dns.qry.name contains "wireshark"
```

## Evidence Collected

| Evidence              | Screenshot                                                      |
| --------------------- | --------------------------------------------------------------- |
| DNS packet overview   | `screenshots/03-dns-investigation/01-dns-overview.png`          |
| DNS query packet      | `screenshots/03-dns-investigation/02-dns-query.png`             |
| DNS response packet   | `screenshots/03-dns-investigation/03-dns-response.png`          |
| DNS query name filter | `screenshots/03-dns-investigation/04-dns-query-name-filter.png` |
| DNS conversations     | `screenshots/03-dns-investigation/05-dns-conversations.png`     |

## Packet-Level Findings

### Finding 1 — DNS Query Packets Were Observed

DNS query packets were observed in the capture. These packets show the client requesting resolution for domain names such as `example.com`, `ubuntu.com`, or `wireshark.org`, depending on the query selected in the packet capture.

In Wireshark, the DNS query packet shows fields such as:

```text id="6dft3s"
Query Name
Query Type
Transaction ID
```

This confirms that DNS can reveal domain names being resolved by a system.

### Finding 2 — DNS Response Packets Were Observed

DNS response packets were observed in the capture. The response packets contained answer records for the queried domain names.

Depending on the query type and resolver response, the answers may include:

```text id="30x763"
A records
AAAA records
CNAME records
Returned IP addresses
```

This shows how DNS maps human-readable domain names to network destinations.

### Finding 3 — DNS Query Name Filtering Helped Locate Specific Domains

The DNS query name filter was used to locate specific queried domains inside the capture.

Example filters:

```text id="qr8oi7"
dns.qry.name contains "example"
dns.qry.name contains "ubuntu"
dns.qry.name contains "wireshark"
```

This is useful during investigations because analysts may need to search for suspicious domains, recently observed domains, or domains linked to alerts.

### Finding 4 — DNS Was Captured on the NAT Interface

The DNS traffic was captured on `analyzer-kali` interface `eth0`, not the internal lab interface `eth1`.

This happened because external DNS queries were routed through the NAT interface. This demonstrates the importance of choosing the correct capture interface based on the expected traffic path.

## Timeline

| Activity               | Description                                          |
| ---------------------- | ---------------------------------------------------- |
| Capture started        | tcpdump capture started on analyzer-kali `eth0`      |
| DNS queries generated  | `dig` and `nslookup` commands executed               |
| DNS responses received | Resolver returned DNS answers for queried domains    |
| Capture stopped        | tcpdump capture stopped after DNS traffic generation |
| PCAP verified          | Saved PCAP confirmed to contain DNS traffic          |

## MITRE ATT&CK Mapping

No direct MITRE ATT&CK technique is assigned for this case.

This case represents normal DNS resolution and protocol analysis. DNS may become relevant to MITRE ATT&CK mapping in later or more advanced scenarios, such as command-and-control, domain generation algorithm activity, tunneling, or malicious infrastructure lookup.

## Conclusion

The packet capture successfully recorded DNS query and response traffic.

The analysis showed that DNS packets can reveal queried domain names, query types, response records, and resolver communication. It also demonstrated that capture interface selection matters because DNS traffic may follow a different path than internal lab traffic.

## Recommended Actions

* Review DNS query names and response records during investigations.
* Correlate suspicious DNS queries with proxy logs, endpoint telemetry, and threat intelligence.
* Do not rely on DNS alone to prove user intent or malicious activity.
* Capture from the correct interface based on the network path being investigated.
* Consider encrypted DNS protocols such as DoH or DoT in real environments, as they may reduce DNS visibility in packet captures.

## Limitations

DNS queries alone do not prove that a user intentionally visited a website. DNS requests may be generated by browsers, operating systems, background applications, updates, prefetching, or cached services.

This capture was performed on the analyzer VM's NAT interface instead of the internal lab interface because the DNS traffic was routed externally.

This case used normal domain lookups and does not represent malicious DNS activity.
