# Wireshark Filters

## Case 1 — Normal Traffic Baseline

```text
arp
icmp
http
tcp.port == 80
tcp.port == 22
ssh
ftp
tcp.port == 21
ip.addr == 192.168.100.30
ip.addr == 192.168.100.10
ip.addr == 192.168.100.10 && ip.addr == 192.168.100.30
```

## Case 2 — TCP Handshake and HTTP Connection Analysis

```text
tcp.port == 80
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.flags.syn == 1 && tcp.flags.ack == 1
http.request
http.response
ip.addr == 192.168.100.10 && ip.addr == 192.168.100.30
```

## Case 3 — DNS Investigation

```text
dns
udp.port == 53
tcp.port == 53
dns.flags.response == 0
dns.flags.response == 1
dns.qry.name contains "example"
dns.qry.name contains "ubuntu"
dns.qry.name contains "wireshark"
```

## Case 4 — Nmap Port Scan Investigation

```text
ip.src == 192.168.100.10 && ip.dst == 192.168.100.30
ip.src == 192.168.100.10 && ip.dst == 192.168.100.30 && tcp.flags.syn == 1 && tcp.flags.ack == 0
ip.src == 192.168.100.30 && ip.dst == 192.168.100.10 && tcp.flags.syn == 1 && tcp.flags.ack == 1
ip.src == 192.168.100.30 && ip.dst == 192.168.100.10 && tcp.flags.reset == 1
tcp.port == 21 || tcp.port == 22 || tcp.port == 80
```

## Case 5 — ICMP Ping Sweep / Local Host Discovery Investigation

```text
arp || icmp
arp
ip.src == 192.168.100.10 && icmp.type == 8
ip.dst == 192.168.100.10 && icmp.type == 0
icmp.type == 8
icmp.type == 0
```