---
id: msrpc
title: MSRPC - 135,593
---

### Rpcclient enumeration

#### Null session
```bash
rpcclient -U "" <IP> -N 
rpcclient $> enumdomusers
rpcclient $> enumdomgroups
rpcclient $> querygroupmem <rid>
rpcclient $> queryuser <rid>
rpcclient $> querydispinfo
```

```bash
rpcclient -U "" <IP> -N -c 'enumdomusers'
rpcclient -U "" <IP> -N -c 'enumdomgroups'
```

#### Usuario y contraseña

```bash
rpcclient -U "user%password" <IP> -c 'enumdomusers'
rpcclient -U "user%password" <IP> -c 'querydispinfo'
```

#### Usuario y hash

```bash
rpcclient -U "user%hash" --pw-nt-hash <IP> 
```


https://book.hacktricks.xyz/network-services-pentesting/135-pentesting-msrpc