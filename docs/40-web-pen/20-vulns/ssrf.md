---
title: SSRF – Detección y explotación
description: Cheatsheet corto para validar SSRF (OOB), pivotar a internos y aplicar bypasses comunes (allowlist/redirects).
tags:
  - web
  - ssrf
  - oob
  - bypass
  - pivot
  - open-redirect
  - cloud
  - aws
  - metadata
  - mitigation
sidebar_position: 30
---

# SSRF – Detección y explotación

SSRF ocurre cuando el servidor realiza peticiones a destinos controlados por el usuario. Sirve para **pivotar** a servicios internos, acceder a **metadata cloud** o encadenar con fallos internos.

---

## 1. Checklist (rápido)

1) Localiza input tipo URL/destino (fetch, webhook, import, preview).  
2) Valida SSRF:
   - **In-band** (si ves respuesta/errores)
   - **OOB** (si es ciego)  
3) Identifica restricciones (allowlist, bloqueo RFC1918/localhost, esquemas).  
4) Pivot a internos (localhost/RFC1918/metadata).  
5) Aplica bypass (redirect, `@`, encoding, DNS).  
6) Identifica el servicio interno y encadena (auth bypass/RCE/LFI/panel).

---

## 2. Validación (OOB recomendado)

Herramientas:
- Burp Collaborator / interactsh / webhook.site / canarytokens

Evidencias:
- callback DNS/HTTP con timestamp
- host solicitado
- headers/IP origen (si aplica)

---

## 3. Targets de pivot (tabla)

| Tipo | Ejemplos |
|---|---|
| Loopback | `http://127.0.0.1:8080/` · `http://localhost:8080/` · `http://[::1]:8080/` |
| RFC1918 | `http://10.0.0.5:8080/` · `http://172.16.0.10/` · `http://192.168.1.1/` |
| Metadata | `http://169.254.169.254/` |

---

## 4. Bypass comunes (mínimo viable)

### 4.1 Open Redirect (allowlist → internal)
Si te dejan `https://allowed.com`, busca redirect abierto:
- `https://allowed.com/redirect?url=http://127.0.0.1:8080/`
- `https://allowed.com/go?next=http://169.254.169.254/`

### 4.2 Userinfo `@` (parsing)
- `http://allowed.com@127.0.0.1:8080/`

### 4.3 Encoding / doble encoding (según stack)
- `%2f`, `%3a`, `%40`, `%252f`…

### 4.4 DNS tricks (si filtra por hostname)
- host que resuelve a IP interna / rebinding (situacional)

---

## 5. SSRF ciego (blind)

- Confirma con **OOB**.
- Usa **timing** como señal:
  - `refused` suele ser rápido, `timeout` suele ser lento (depende del backend).

Guarda una tabla breve: `target:port -> refused/timeout/status`.

---

## 6. Mitigación (esencial)

- Allowlist **real** (host exacto, parsing robusto).
- Resolver DNS y bloquear IP final: loopback/RFC1918/link-local.
- No seguir redirects (o validar cada salto).
- Egress filtering (salidas limitadas por red/puerto).
- Logs/alertas de conexiones salientes.

---

## 7. Notas

- Añade aquí casos específicos de writeups (CVE, producto, comportamiento raro) sin inflar el cheatsheet.

---

## 8. Referencias

- PortSwigger Web Security Academy – SSRF
- HackTricks – SSRF
- OWASP (WSTG / mitigación SSRF)
