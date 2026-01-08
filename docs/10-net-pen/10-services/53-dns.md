---
id: dns
title: DNS - 53
---

# DNS - 53

### Local Host Resolution Override
```bash
nano /etc/hosts
<IP> <dominio>
```

### Direct Record Query
#### Direct A/AAAA Record Query
```bash
dig @<IP> dominio.dominio
```

#### Mail Exchange Records Enumeration (MX)
```bash
dig @<IP> dominio.dominio mx
```

#### Authoritative Name Servers Enumeration (NS)
```bash
dig @<IP> dominio.dominio ns
```

#### Full DNS Records Enumeration (ANY)
```bash
dig @<IP> dominio.dominio any
```

### Domain Zone Transfer Attack (AXFR)
```bash
dig @<IP> dominio.dominio axfr
```
