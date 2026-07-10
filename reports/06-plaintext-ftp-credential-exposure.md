# Case 6 — Plaintext FTP Credential Exposure

## Scenario

A user logged in to the Ubuntu FTP server using a lab-only test account. The purpose of this case is to determine whether FTP usernames and passwords can be observed in packet contents.

This case demonstrates the risk of using plaintext protocols for authentication.

## Objective

Identify plaintext FTP credentials and control commands using packet-level evidence in Wireshark.

## Analyst Question

Can FTP usernames and passwords be observed in packet contents?

## PCAP File

```text
pcaps/06-plaintext-ftp-credential-exposure.pcap
```

## Lab Environment

| Asset              | Hostname      | Role                               | IP Address     |
| ------------------ | ------------- | ---------------------------------- | -------------- |
| Source host        | attacker-kali | FTP client                         | 192.168.100.10 |
| Capture sensor     | analyzer-kali | Packet capture and analysis sensor | 192.168.100.20 |
| Destination server | victim-ubuntu | FTP server                         | 192.168.100.30 |

## Tools Used

| Tool      | Purpose                                       |
| --------- | --------------------------------------------- |
| tcpdump   | Captured FTP traffic from the analyzer VM     |
| Wireshark | Inspected FTP commands and Follow TCP Stream  |
| ftp       | Generated FTP login and command traffic       |
| vsftpd    | Provided the FTP service on the victim server |

## Capture Command

The packet capture was performed from `analyzer-kali` on the internal lab interface `eth1`.

```bash
sudo tcpdump -i eth1 -nn -w ~/pcaps/06-plaintext-ftp-credential-exposure.pcap 'host 192.168.100.30 and tcp port 21'
```

## Traffic Generation Commands

The following command was executed from `attacker-kali`.

```bash
ftp 192.168.100.30
```

The FTP session used a lab-only test account:

```text
Username: ftpuser
Password: [REDACTED]
```

FTP commands used during the session:

```text
pwd
ls
cd ftp-test
ls
bye
```

## Capture Summary

| Field                  | Value          |
| ---------------------- | -------------- |
| Packets captured       | 56             |
| Snapshot length        | 262144 bytes   |
| Packets dropped        | 0              |
| Source IP              | 192.168.100.10 |
| Destination IP         | 192.168.100.30 |
| Destination port       | TCP/21         |
| Protocol observed      | FTP            |
| Username visible       | Yes            |
| Password field visible | Yes            |
| Login response visible | Yes            |

## Wireshark Filters Used

```text
ftp
ftp.request.command == "USER"
ftp.request.command == "PASS"
ftp.response.code == 230
ftp.request
tcp.port == 21
ip.addr == 192.168.100.10 && ip.addr == 192.168.100.30
```

## Evidence Collected

| Evidence                      | Screenshot                                                                   |
| ----------------------------- | ---------------------------------------------------------------------------- |
| FTP overview                  | `screenshots/06-plaintext-credentials/01-ftp-overview.png`                   |
| FTP USER command              | `screenshots/06-plaintext-credentials/02-ftp-user-command.png`               |
| FTP PASS command              | `screenshots/06-plaintext-credentials/03-ftp-pass-command-redacted.png`      |
| FTP login successful response | `screenshots/06-plaintext-credentials/04-ftp-login-successful.png`           |
| FTP commands after login      | `screenshots/06-plaintext-credentials/05-ftp-commands-after-login.png`       |
| FTP Follow TCP Stream         | `screenshots/06-plaintext-credentials/06-ftp-follow-tcp-stream-redacted.png` |
| FTP TCP conversation          | `screenshots/06-plaintext-credentials/07-ftp-tcp-conversation.png`           |

## Packet-Level Findings

### Finding 1 — FTP Username Was Visible in Plaintext

The FTP `USER` command was visible in the packet capture.

The observed command showed the username being transmitted over the network in readable text:

```text
USER ftpuser
```

This confirms that FTP exposes the username in the control channel.

### Finding 2 — FTP Password Field Was Visible in Plaintext

The FTP `PASS` command was also visible in the packet capture.

The screenshot was redacted for public documentation, but Wireshark showed that the password field was transmitted in readable text.

This demonstrates that FTP does not protect authentication data by default.

### Finding 3 — FTP Login Response Confirmed Successful Authentication

The FTP server returned a successful login response after the username and password were sent.

The observed FTP response indicated that the login was accepted by the server.

This confirms that the captured credentials were part of a real FTP authentication flow in the controlled lab.

### Finding 4 — FTP Control Commands Were Readable After Login

Post-login FTP commands were visible in plaintext, including directory and navigation commands such as:

```text
PWD
LIST
CWD
QUIT
```

This shows that FTP exposes not only credentials, but also user activity within the FTP control channel.

### Finding 5 — Follow TCP Stream Showed Readable FTP Session Content

Wireshark's Follow TCP Stream feature displayed the FTP session in readable text.

This is strong evidence that an observer with packet visibility could reconstruct the FTP control-channel activity, including the login sequence and commands issued after authentication.

## Timeline

| Activity              | Description                                              |
| --------------------- | -------------------------------------------------------- |
| Capture started       | tcpdump capture started on analyzer-kali `eth1`          |
| FTP session initiated | FTP client connected from attacker-kali to victim-ubuntu |
| Username sent         | FTP `USER` command observed                              |
| Password sent         | FTP `PASS` command observed and redacted in screenshot   |
| Login accepted        | FTP successful login response observed                   |
| FTP commands executed | Directory and navigation commands were issued            |
| Capture stopped       | tcpdump capture stopped after FTP session ended          |
| PCAP verified         | Saved PCAP confirmed to contain FTP traffic on TCP/21    |

## MITRE ATT&CK Mapping

No direct MITRE ATT&CK technique is assigned for this case.

This case demonstrates insecure protocol behavior and credential exposure risk. It does not represent credential theft by itself because the traffic was generated intentionally in a controlled lab with fake credentials.

In a real investigation, captured FTP credentials could become relevant to credential access or lateral movement analysis depending on how the credentials were obtained and used.

## Conclusion

The capture confirms that FTP transmits authentication data and control commands in plaintext.

The username, password field, login response, and post-login FTP commands were visible in Wireshark. This demonstrates why traditional FTP is unsuitable for transmitting credentials or sensitive data across untrusted or monitored networks.

## Recommended Actions

In a real environment, the following actions would be recommended:

* Replace FTP with encrypted alternatives such as SFTP or FTPS.
* Disable plaintext FTP where there is no business requirement.
* Restrict FTP access using firewall rules and network segmentation.
* Monitor for FTP usage, especially from user workstations or untrusted networks.
* Avoid using real credentials over plaintext protocols.
* Review authentication logs if plaintext credential exposure is suspected.
* Rotate credentials if real passwords were transmitted over FTP.

## Limitations

This case used a fake lab-only FTP account. No real user credentials were used.

The capture demonstrates plaintext exposure, but it does not prove malicious credential theft.

Packet capture alone shows that credentials were transmitted in plaintext. Additional evidence such as authentication logs, endpoint telemetry, user context, and access history would be needed in a real incident investigation.
