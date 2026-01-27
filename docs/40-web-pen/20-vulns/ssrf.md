---
title: Comprendiendo el SSRF
description: Qué es SSRF, cómo se explota (incluido pivot a red interna y cloud metadata), variantes, impacto, detección y mitigación práctica.
tags:
  - ssrf
  - web-security
  - api-security
  - owasp
  - cloud
  - aws-metadata
  - pentesting
  - mitigation
sidebar_position: 10
---

# Comprendiendo el SSRF: la vulnerabilidad que convierte tu servidor en un caballo de Troya

El **Server-Side Request Forgery (SSRF)** es una vulnerabilidad crítica que ocurre cuando una aplicación permite que el **servidor** realice peticiones hacia un destino controlado (directa o indirectamente) por un atacante. En lugar de comunicarse solo con servicios previstos, el atacante manipula la entrada para que el servidor actúe como **puente** hacia recursos internos o externos **no autorizados**.

> Idea clave: **no es el atacante quien “llega” a la red interna**, es tu **servidor** quien sale a pedir cosas por él.

---

## ¿Cómo funciona un SSRF?

En un escenario típico, un servidor web actúa como intermediario entre Internet y una red corporativa protegida por firewall. Con SSRF, el atacante usa ese servidor como un **punto de giro (pivot)** para eludir controles perimetrales y acceder a servicios que normalmente **no son accesibles desde fuera**.

### Ejemplo clásico: robo de metadatos en la nube

Muchas aplicaciones consultan APIs externas (por ejemplo, stock, previsualización de URLs, importadores de imágenes, webhooks, etc.). Si el parámetro de URL es controlable, un atacante puede redirigir la petición a endpoints internos.

1. **Petición legítima**  
   `stockApi=http://api.empresa.com/v1/stock`

2. **Petición maliciosa**  
   `stockApi=http://169.254.169.254/latest/meta-data/iam/security-credentials/`

3. **Resultado**  
   El servidor devuelve **metadatos/credenciales temporales** (por ejemplo, en AWS), lo que puede permitir **tomar control** parcial o total de la infraestructura cloud.

**Por qué funciona:** el servidor “confía” en sí mismo y tiene acceso a redes/servicios que el atacante no tiene.

---

## Tipos de SSRF

Las variantes más comunes dependen de **qué se alcanza** y de **si el atacante ve o no la respuesta**.

### 1) SSRF contra el propio servidor
El atacante obliga a la app a pedir recursos a **localhost** o **127.0.0.1**, llegando a:
- paneles internos
- endpoints de administración
- servicios de debug
- APIs internas

**Ejemplo de objetivo:** `http://127.0.0.1:PORT/admin`

---

### 2) SSRF contra otros sistemas back-end
Se apunta a sistemas en la red interna con IPs privadas (por ejemplo `192.168.0.0/16`, `10.0.0.0/8`) que suelen tener menos endurecimiento al no estar expuestos.

**Ejemplo de objetivo:** `http://10.0.0.5:8080/`

---

### 3) Blind SSRF (SSRF ciego)
La aplicación realiza la petición, pero **no devuelve la respuesta** al usuario. Aun así, puede explotarse para:
- **port scanning interno**
- **detección de servicios**
- **exfiltración por canales laterales**
- **RCE indirecta** si el objetivo interno lo permite

**Cómo se valida:** con técnicas **OOB (Out-of-Band)** (colaboradores/webhooks) y análisis de tiempos.

---

### 4) SSRF vía protocolos no HTTP
Cuando el parser/cliente permite otros esquemas, el impacto aumenta. Ejemplos típicos:

- `file://` → lectura de archivos locales (p.ej. `/etc/passwd`)
- `gopher://` → envío de payloads raw para hablar con Redis/MySQL/otros servicios
- `dict://` → accesos a servicios basados en diccionario (menos habitual)

> Nota: que sea posible depende del stack (librería HTTP, framework, validaciones, runtime).

---

## Impacto de un SSRF exitoso

El impacto depende del **alcance de red del servidor** y de **qué servicios internos existan**, pero suele incluir:

- **Exfiltración de información sensible**: credenciales, archivos locales, tokens, datos de clientes.
- **Acceso a interfaces administrativas**: paneles internos accesibles solo desde “dentro”.
- **Escalada de privilegios**: pivot hacia servicios con más permisos.
- **Ejecución remota de código (RCE)**: cuando el SSRF alcanza un servicio interno vulnerable o con funcionalidades peligrosas.
- **Denegación de servicio (DoS)**: forzando descargas enormes o rutas que consumen CPU/IO (p.ej. `file:///dev/urandom` en algunos contextos).

---

## Detección y herramientas

En SSRF, muchas veces la clave es confirmar **si el servidor está haciendo la petición**. Para eso se usan técnicas de **interacción fuera de banda (OOB)** que registran conexiones salientes.

### Herramientas recomendadas

**Interacción y captura (OOB):**
- **Burp Collaborator**
- **webhook.site**
- **canarytokens**
- **pingb**

**Análisis automatizado / escaneo:**
- **SSRFmap**
- **OWASP ZAP**
- Escáner de **Burp Suite**

**Plugins y utilidades:**
- **Collaborator Everywhere** (amplía superficies al inyectar y observar callbacks)
- **Gopherus** (payloads para `gopher://` y protocolos complejos)
- **remote-method-guesser** (entornos Java/RMI)

> Realidad práctica: SSRF rara vez es “click → pwn”. Requiere entender el flujo, el parser, DNS, redirects, egress y la red interna.

---

## Cómo mitigar SSRF

La defensa efectiva es una combinación de **validación**, **control de red** y **diseño seguro**.

### 1) Listas de permitidos (Allowlist)
Permitir solo destinos explícitos y conocidos:
- dominios concretos
- rutas específicas
- esquemas permitidos

**Mejor**: allowlist por **host + puerto + path** cuando aplique.

---

### 2) Validación estricta de URL y esquema
- restringir a `http`/`https`
- bloquear esquemas no deseados (`file`, `gopher`, etc.)
- evitar parseos ambiguos (doble encoding, userinfo `user@host`, etc.)

---

### 3) Resolución DNS y protección contra DNS rebinding
Antes de conectar:
- resolver DNS
- validar que el destino **no** cae en rangos privados/loopback/link-local
- repetir validación si hay redirects

---

### 4) Aislamiento de red (segmentación / DMZ)
- no mezclar servidores expuestos a Internet con redes internas sensibles
- aplicar **egress filtering**: el servidor no debería poder hablar con “todo”
- separar servicios internos críticos (metadata, admin panels, DBs)

---

### 5) Deshabilitar o controlar redirecciones
Evitar `follow redirects` automático. Un atacante puede:
- pasar un filtro inicial con un dominio permitido
- redirigir luego a un destino interno

---

## Checklist rápida

**Si auditas:**
- ¿hay parámetros URL / fetch / webhook / import?
- ¿hay redirects?
- ¿hay egress libre?
- ¿respuestas reflejadas o SSRF ciego?
- ¿se alcanza `localhost`, IPs privadas o `169.254.169.254`?

**Si defiendes:**
- allowlist real + validación robusta
- bloquear IPs internas/loopback/link-local
- egress filtering estricto
- controlar redirects
- logs de conexiones salientes (muy útil para detectar abuso)

---
## Referencias

- [PortSwigger Web Security Academy – SSRF](https://portswigger.net/web-security/ssrf)
- [HackTricks (ES) – SSRF](https://book.hacktricks.wiki/es/pentesting-web/ssrf-server-side-request-forgery/index.html)
- [Akamai Glossary – What is SSRF](https://www.akamai.com/es/glossary/what-is-server-side-request-forgery)
- [Hackmetrix – SSRF Server-Side Request Forgery](https://blog.hackmetrix.com/ssrf-server-side-request-forgery/)
- [Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)

