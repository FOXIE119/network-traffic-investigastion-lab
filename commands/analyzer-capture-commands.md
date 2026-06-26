# Analyzer Capture Commands

## Case 1 — Normal Traffic Baseline

```bash
sudo tcpdump -i eth1 -nn -w ~/pcaps/01-normal-traffic-baseline.pcap host 192.168.100.30
```

### Verification:

```bash
ls -lh ~/pcaps/01-normal-traffic-baseline.pcap

tcpdump -nn -r ~/pcaps/01-normal-traffic-baseline.pcap | head -30

tcpdump -nn -r ~/pcaps/01-normal-traffic-baseline.pcap 'icmp' | head

tcpdump -nn -r ~/pcaps/01-normal-traffic-baseline.pcap 'tcp port 80' | head

tcpdump -nn -r ~/pcaps/01-normal-traffic-baseline.pcap 'tcp port 22' | head

tcpdump -nn -r ~/pcaps/01-normal-traffic-baseline.pcap 'tcp port 21' | head
```