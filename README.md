# Network Protocol Security Lab Walkthrough

## 1. Lab Setup
- Tools used: Hydra, Medusa, Burp Suite, Wireshark, etc.
- Target machine: Metasploitable 2

## 2. Enumeration
### Nmap Scan
```bash
nmap -p 21,23,22 --script=ftp-anon,telnet-encryption,ssh-hostkey <TARGET_IP>
