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

## Case 2 — TCP Handshake and HTTP Connection Analysis

```bash
sudo tcpdump -i eth1 -nn -w ~/pcaps/02-tcp-handshake-http.pcap 'host 192.168.100.30 and tcp port 80'
```

### Verification:

```bash
ls -lh ~/pcaps/02-tcp-handshake-http.pcap
tcpdump -nn -r ~/pcaps/02-tcp-handshake-http.pcap | head -30
tcpdump -nn -r ~/pcaps/02-tcp-handshake-http.pcap 'tcp port 80' | head
```

## Case 3 — DNS Investigation

```bash
sudo tcpdump -i eth0 -nn -w ~/pcaps/03-dns-investigation.pcap 'udp port 53 or tcp port 53'
```

### DNS generation commands:

```bash
dig example.com
dig ubuntu.com
nslookup wireshark.org
```

### Verification:

```bash
ls -lh ~/pcaps/03-dns-investigation.pcap
tcpdump -nn -r ~/pcaps/03-dns-investigation.pcap | head -30
tcpdump -nn -r ~/pcaps/03-dns-investigation.pcap 'udp port 53 or tcp port 53' | head
```

## Case 4 — Nmap Port Scan Investigation

```bash
sudo tcpdump -i eth1 -nn -w ~/pcaps/04-nmap-portscan-investigation.pcap host 192.168.100.30
```

### Verification:

```bash
ls -lh ~/pcaps/04-nmap-portscan-investigation.pcap
tcpdump -nn -r ~/pcaps/04-nmap-portscan-investigation.pcap | head -30
tcpdump -nn -r ~/pcaps/04-nmap-portscan-investigation.pcap 'tcp[tcpflags] & tcp-syn != 0' | head
tcpdump -nn -r ~/pcaps/04-nmap-portscan-investigation.pcap 'tcp port 21 or tcp port 22 or tcp port 80' | head
```

## Case 5 — ICMP Ping Sweep / Local Host Discovery Investigation

```bash
sudo tcpdump -i eth1 -nn -w ~/pcaps/05-icmp-pingsweep-investigation.pcap '(icmp or arp)'
```

### Verification:

```bash
ls -lh ~/pcaps/05-icmp-pingsweep-investigation.pcap
tcpdump -nn -r ~/pcaps/05-icmp-pingsweep-investigation.pcap arp | head -30
tcpdump -nn -r ~/pcaps/05-icmp-pingsweep-investigation.pcap icmp | head -30
tcpdump -nn -r ~/pcaps/05-icmp-pingsweep-investigation.pcap | head -50
```

## Case 6 — Plaintext FTP Credential Exposure

```bash
sudo tcpdump -i eth1 -nn -w ~/pcaps/06-plaintext-ftp-credential-exposure.pcap 'host 192.168.100.30 and tcp port 21'
```

### Verification:

```bash
ls -lh ~/pcaps/06-plaintext-ftp-credential-exposure.pcap
tcpdump -nn -r ~/pcaps/06-plaintext-ftp-credential-exposure.pcap | head -40
tcpdump -nn -r ~/pcaps/06-plaintext-ftp-credential-exposure.pcap 'tcp port 21' | head -40
```