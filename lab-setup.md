# Lab Setup

## Environment Design

This lab uses three virtual machines connected through VirtualBox.

| VM | OS | Role |
|---|---|---|
| attacker-kali | Kali Linux | Generates controlled traffic |
| analyzer-kali | Kali Linux | Captures traffic using tcpdump and analyzes PCAPs |
| victim-ubuntu | Ubuntu Server 24.04 LTS | Hosts SSH, HTTP, and FTP services |

## Network Configuration

Each VM uses two network adapters:

| Adapter | Mode | Purpose |
|---|---|---|
| Adapter 1 | NAT | Internet access for updates and tools |
| Adapter 2 | Internal Network: lab-net | Isolated lab traffic |

## IP Addressing

| VM | Lab Interface | Lab IP |
|---|---|---|
| attacker-kali | eth1 | 192.168.100.10/24 |
| analyzer-kali | eth1 | 192.168.100.20/24 |
| victim-ubuntu | enp0s8 | 192.168.100.30/24 |

## Victim Services

The Ubuntu victim server provides:

| Service | Port |
|---|---:|
| SSH | 22 |
| HTTP / Apache2 | 80 |
| FTP / vsftpd | 21 |

## Capture Point

Traffic is captured from the analyzer VM on:

```bash
eth1