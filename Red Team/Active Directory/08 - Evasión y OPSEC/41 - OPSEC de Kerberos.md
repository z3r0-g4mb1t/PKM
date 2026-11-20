---
tags:
  - Active-Directory
  - Windows
  - Pentesting/Explotacion
  - Tipo/Deteccion
Descripción: "Operar con Kerberos sin delatarte: el downgrade a RC4 que te quema, roasting dirigido, encryption types y los eventos 4769/4768 que modela el defensor"
Fecha de actualización: 2026-11-20
Nota previa: "[[40 - Recon y enumeración sigilosa]]"
Nota siguiente: "[[42 - Forjado de tickets y su detección]]"
Area: "[[Evasión y OPSEC.base|Evasión y OPSEC]]"
---
---

Kerberos es el terreno donde más se juega la OPSEC en AD, porque casi todo pasa por el DC y queda en sus eventos `4768`/`4769`. La mecánica del protocolo está en [[03 - Kerberos por dentro]]; esta nota es cómo usarlo sin que cada acción sea una bandera roja. El hilo conductor es uno: <mark style="background: #ADCCFFA6;">el **tipo de cifrado** del ticket es tu mayor indicador, y casi siempre está en tu mano elegirlo</mark>.

# El downgrade a RC4: el error que más delata

Un ticket Kerberos va cifrado con la clave del principal, y esa clave puede ser el NT hash (cifrado **RC4**, tipo `0x17`) o una clave **AES** (`0x12` para AES-256). Históricamente los atacantes **forzaban RC4** porque se crackea mucho más rápido (Hashcat `13100` vs `19700`). El problema:

> [!warning]+ Pedir RC4 en un dominio AES es una alarma de alta fidelidad
> Los dominios modernos operan "AES-only" o casi: los tickets legítimos son AES. <mark style="background: #FF5582A6;">Una petición `4769` (TGS) con `Ticket Encryption Type 0x17` (RC4) en ese entorno es anómala por sí sola</mark>, y MDI y cualquier SIEM decente la marcan — es la firma clásica del Kerberoasting con downgrade. El atacante que fuerza RC4 para crackear más rápido está cambiando horas de GPU por una alerta garantizada. En 2026 ese trato casi nunca compensa.

La regla: **pide en el tipo de cifrado que el dominio usa por defecto**. Si es AES, roastea en AES (más lento de crackear, pero solo si la contraseña es débil — y si lo es, caerá igual) y no generes la anomalía. Herramientas como Rubeus lo contemplan: `/rc4opsec` pide RC4 **solo** a las cuentas que no soportan AES (donde RC4 no es anómalo), y `/aes` fuerza AES.

# Kerberoasting dirigido, no masivo

El Kerberoasting ([[23 - Kerberoasting]]) es barato de detectar si se hace mal: pedir TGS de **todas** las cuentas con SPN de golpe es una ráfaga de `4769` que ningún comportamiento legítimo produce.

- **Enumera primero por LDAP** (qué cuentas tienen SPN, con los filtros de [[12 - Filtros de alto valor para el atacante]]) y pide **solo** las que valen la pena (cuentas con privilegio, contraseñas probablemente viejas).
- **Reparte en el tiempo**: unas pocas peticiones espaciadas se pierden en el tráfico Kerberos normal; cincuenta en diez segundos no.
- **Evita las cuentas que huelen a honeytoken** (SPN jugoso, `lastLogon` nulo, contraseña antiquísima) — [[44 - Movimiento lateral, ejecución y deception]].

# AS-REP roasting y pre-autenticación

El AS-REP roasting ataca cuentas con la **pre-autenticación deshabilitada** ([[03 - Kerberos por dentro]]): pides un `AS-REP` y obtienes material cifrado con el hash de la cuenta para crackear offline, **sin credenciales**. OPSEC:

- Genera eventos `4768` (AS-REQ). Pedir AS-REP de muchas cuentas es, de nuevo, una ráfaga detectable; enumera por LDAP qué cuentas tienen `DONT_REQ_PREAUTH` y ve a por esas, no a por todas.
- El mismo cuidado con el encryption type aplica: el AS-REP roasteable también se pide y crackea, y pedirlo en RC4 cuando no toca es igual de anómalo.

# Overpass-the-hash: preferir la clave AES

*Overpass-the-hash* (pass-the-key) convierte un hash/clave robado en un TGT legítimo: en vez de reutilizar el NT hash por NTLM (PtH), pides un TGT y operas por Kerberos, que es lo normal en el dominio. <mark style="background: #8000E1A6;">Hacerlo con la **clave AES** (`/aes256`) en vez de con el NT hash (`/rc4`) produce un TGT AES indistinguible de uno legítimo</mark>; hacerlo con el NT hash genera un TGT RC4 anómalo (misma alarma que arriba). Si robaste solo el NT hash y el dominio es AES, esa es una razón para preferir robar/derivar la clave AES cuando puedas.

# El checklist de OPSEC Kerberos

Antes de cualquier operación Kerberos, las tres preguntas de [[39 - OPSEC en AD - el modelo de detección y el mindset]] se concretan así:

1. **¿Encryption type correcto?** AES si el dominio es AES. Nunca forzar RC4 "por costumbre".
2. **¿Dirigido, no masivo?** Enumera por LDAP y pide solo lo que vale; reparte en el tiempo.
3. **¿Desde quién?** Las peticiones Kerberos desde una cuenta que ya usa esos servicios son ruido; desde una cuenta que nunca los toca, señal.

El forjado de tickets (golden/silver/diamond) es el siguiente nivel y tiene su propia OPSEC —sobre todo el encryption type del ticket forjado y la heurística "TGT sin AS-REQ previo"—, en [[42 - Forjado de tickets y su detección]].

> [!info]+ Fuentes
> [Rubeus — kerberoast / opsec flags](https://github.com/GhostPack/Rubeus), eventos [`4769`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4769) y [`4768`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4768), [MDI — Kerberoasting / suspicious Kerberos](https://learn.microsoft.com/en-us/defender-for-identity/alerts-xdr). Mecánica en [[03 - Kerberos por dentro]].
