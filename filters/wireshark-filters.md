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