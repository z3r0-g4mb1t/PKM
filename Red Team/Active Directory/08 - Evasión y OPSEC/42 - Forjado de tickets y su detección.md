---
tags:
  - Active-Directory
  - Windows
  - Pentesting/Post-Explotacion
  - Tipo/Deteccion
Descripción: "Golden, Silver, Diamond y Sapphire tickets: qué forja cada uno, qué los delata (encryption type, PAC, 'TGT sin AS-REQ') y cómo el blue valida el PAC"
Fecha de actualización: 2026-11-20
Nota previa: "[[41 - OPSEC de Kerberos]]"
Nota siguiente: "[[43 - OPSEC de credential access]]"
Area: "[[Evasión y OPSEC.base|Evasión y OPSEC]]"
---
---

Forjar tickets es el arte de fabricar credenciales Kerberos que el dominio acepta como legítimas. Es persistencia y suplantación de máximo nivel, y donde más ha evolucionado la carrera entre rojo y azul: cada técnica nueva (golden → diamond → sapphire) nace para evadir la detección de la anterior. Entender **qué firma el defensor** en cada una es lo que separa forjar un ticket que dura de forjar uno que dispara una alerta en segundos. La base (TGT, PAC, claves) está en [[03 - Kerberos por dentro]].

# Los cuatro tickets, por lo que forjan

| Ticket | Clave que necesitas | Qué forja | Alcance |
| --- | --- | --- | --- |
| **Golden** | hash/clave de **`krbtgt`** | Un **TGT** de cero, con el PAC que quieras | Todo el dominio, persistente |
| **Silver** | hash/clave de **una cuenta de servicio** | Un **TGS** de cero para **ese** servicio | Solo ese servicio, pero sin tocar el DC |
| **Diamond** | hash/clave de **`krbtgt`** | Pide un TGT **legítimo** y le **modifica** el PAC | Como el golden, pero más sigiloso |
| **Sapphire** | hash/clave de **`krbtgt`** | Como diamond, pero el PAC se copia de un **usuario privilegiado real** (vía S4U) | Como el diamond, PAC impecable |

<mark style="background: #ADCCFFA6;">La progresión no es de potencia sino de **sigilo**</mark>: los cuatro dan control equivalente, pero cada uno deja menos rastro que el anterior.

# Qué delata a cada uno

**Golden ticket** — forjado **de cero**. Dos problemas de OPSEC:
- <mark style="background: #FF5582A6;">No hay `AS-REQ` previo</mark>: el TGT aparece en uso (`4769` al pedir un TGS con él) sin que el DC haya registrado el `4768` de su emisión. La heurística "TGT en uso sin AS-REQ que lo emitiera" es una de las detecciones clásicas.
- Si se forja con el NT hash → **RC4**, el encryption type anómalo de [[41 - OPSEC de Kerberos]]. Y herramientas antiguas ponían *lifetimes* irreales (10 años) o PACs con campos por defecto que no cuadran con los de un ticket real.

**Silver ticket** — forjado de cero **para un servicio**, presentado directamente al servicio (`AP-REQ`) **sin pasar por el DC**. Eso es su ventaja (el DC no ve nada) y su límite (solo ese servicio). Riesgo: si el servicio hace **validación del PAC contra el DC** (`PAC validation`), el PAC forjado no cuadra y falla. Forjar el PAC con grupos coherentes y el encryption type correcto es lo que lo mantiene vivo.

**Diamond ticket** — <mark style="background: #8000E1A6;">pide un TGT **legítimo** (hay `AS-REQ` real, `4768` normal) y solo descifra/modifica su PAC</mark> con la clave de krbtgt antes de reusarlo. Mata la heurística "TGT sin AS-REQ" del golden: para el DC, hubo una emisión legítima. Es la opción por defecto hoy cuando tienes la clave de krbtgt y quieres sigilo (`Rubeus diamond`).

**Sapphire ticket** — el diamond llevado al extremo: en vez de inventar el PAC, **copia el PAC de un usuario privilegiado real** obtenido vía S4U2Self. El resultado es un ticket cuyo PAC es byte a byte el de un DA legítimo, indistinguible incluso para un análisis del PAC. El estado del arte en forjado sigiloso.

# Cómo lo caza (y debe cazar) el blue — la contraparte CRTO

Entender la detección es parte del temario, y revela por qué unas técnicas envejecen y otras no:

- **Encryption type anómalo** (`4769`/`4768` con RC4 en dominio AES): caza golden/overpass hechos con NT hash. Evasión: AES.
- **"TGT sin AS-REQ"**: correlación en el SIEM/MDI de un TGS que usa un TGT que el DC nunca emitió. Caza el golden clásico. Evasión: diamond/sapphire (hay AS-REQ real).
- **PAC inconsistente**: los parches de Microsoft de 2021-2022 endurecieron la **validación del PAC** (incluida la firma extendida del PAC) precisamente para cazar PACs forjados o manipulados. Caza silver/golden con PAC mal construido. Evasión: PAC coherente (diamond) o copiado de uno real (sapphire).
- **`krbtgt` con dos contraseñas recientes / anomalías de replicación**: indicios de que alguien obtuvo el hash de krbtgt (DCSync). MDI tiene detección dedicada.
- **Tickets de duración anómala, usuarios que no existen, SIDs incoherentes**: restos de herramientas mal configuradas.

> [!important]+ La única defensa real contra el Golden Ticket
> Un Golden Ticket vale hasta que se **rota la contraseña de `krbtgt` dos veces** (dos veces porque Kerberos mantiene la clave anterior para no invalidar tickets en vuelo). Si sospechas compromiso de krbtgt, una sola rotación no basta. Para el atacante, esto define la ventana de persistencia; para el defensor, la rotación doble de krbtgt es el *reset* tras un compromiso de dominio. El ataque de trust hijo→padre ([[33 - Ataque a trust hijo a padre]]) es precisamente un golden/diamond con el SID de Enterprise Admins inyectado en el PAC.

# El checklist

1. **¿Tengo la clave AES o solo el NT hash?** Con AES, el ticket no delata por encryption type. Prefiere robar/derivar AES.
2. **¿Golden o diamond?** Si tienes krbtgt y buscas sigilo, **diamond** (hay AS-REQ real). Golden solo si no puedes pedir un TGT legítimo.
3. **¿Silver evita el DC?** Si solo necesitas un servicio concreto, el silver no toca el DC — pero cuida el PAC por si hay validación.
4. **PAC coherente**: grupos, SIDs y lifetimes realistas. Un PAC "demasiado perfecto o imposible" es una firma.

> [!info]+ Fuentes
> [Rubeus — golden/silver/diamond](https://github.com/GhostPack/Rubeus), [MS-PAC](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/166d8064-c863-41e1-9c23-edaaa5f36962), [Microsoft — KB5008380 / endurecimiento de la validación del PAC](https://support.microsoft.com/en-us/topic/kb5008380-authentication-updates-cve-2021-42287-9dafac11-e0d0-4cb8-959a-143bd0201041), [MDI — forged PAC / golden ticket alerts](https://learn.microsoft.com/en-us/defender-for-identity/alerts-xdr). Base criptográfica en [[03 - Kerberos por dentro]].
