# Attacker Commands

## Case 1 — Normal Traffic Baseline

```bash
ping -c 4 192.168.100.30
curl -I http://192.168.100.30
curl http://192.168.100.30
ssh lab@192.168.100.30
ftp 192.168.100.30
```

### FTP commands used:

```bash
ftpuser
LabFTP123!
pwd
ls
cd ftp-test
ls
bye
```

## Case 2 — TCP Handshake and HTTP Connection Analysis

```bash
curl -I http://192.168.100.30
curl http://192.168.100.30
```

## Case 3 — DNS Investigation

No attacker-kali commands were used for this case.

DNS traffic was generated from analyzer-kali because external DNS lookups were routed through the NAT interface.

## Case 4 — Nmap Port Scan Investigation

```bash
nmap -sV -p 21,22,80 192.168.100.30
sudo nmap -sS -p 1-1000 192.168.100.30
```

## Case 5 — ICMP Ping Sweep / Local Host Discovery Investigation

```bash
for i in $(seq 1 30); do ping -c 1 -W 1 192.168.100.$i; done
```