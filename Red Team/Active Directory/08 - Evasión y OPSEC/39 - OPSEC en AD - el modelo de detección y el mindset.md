---
tags:
  - Active-Directory
  - Windows
  - Pentesting/Post-Explotacion
  - Tipo/Introduccion
Descripción: "El marco de OPSEC para operar en AD: cómo funciona la detección moderna (MDI/EDR/SIEM), la pirámide del dolor aplicada al atacante y por qué no existe sigilo absoluto"
Fecha de actualización: 2026-11-20
Nota previa: "[[38 - Arsenal de herramientas AD]]"
Nota siguiente: "[[40 - Recon y enumeración sigilosa]]"
Area: "[[Evasión y OPSEC.base|Evasión y OPSEC]]"
---
---

La nota [[37 - Detección y evasión en AD]] resume **qué telemetría deja cada ataque**; este sub-tema es la otra mitad, la que de verdad pesa en un Red Team y en CRTO: **cómo operar para dejar lo menos posible y parecer legítimo**. No es una lista de *bypasses*; es un modelo mental. Esta nota lo fija, y las siguientes lo aplican a recon, Kerberos, forjado de tickets, credential access y movimiento lateral.

# Contra qué juegas de verdad

El AD de 2026 no se defiende con un antivirus: se defiende con **tres capas que se correlan**, y cada técnica hay que medirla contra las tres.

| Capa | Qué ve | Dónde vive |
| --- | --- | --- |
| **EDR** (endpoint) | Procesos, inyección, llamadas a API, carga de drivers, acceso a LSASS | En cada host |
| **Identity/network** (MDI) | Tráfico y comportamiento del **propio DC**: recon LDAP, Kerberos anómalo, DCSync, relay | Sensor en los DC |
| **SIEM** | Correlación de los eventos de seguridad de Windows de todos los hosts | Centralizado |

<mark style="background: #ADCCFFA6;">La clave es que se **correlan**</mark>: una acción que es invisible para una capa puede ser obvia para otra, y el SIEM une puntos que por separado no dicen nada. `Microsoft Defender for Identity` es el cambio de juego de los últimos años porque observa el DC —donde acaba casi todo ataque de AD— y modela **comportamiento**, no firmas: no busca "la herramienta X", busca "este principal nunca había hecho reconocimiento LDAP masivo y ahora sí".

> [!important]+ La consecuencia operativa que define todo el sub-tema
> Como MDI modela comportamiento con **líneas base por principal y por host**, el enemigo del atacante no es solo *qué* hace, sino *quién* lo hace, *desde dónde* y *cuándo*. <mark style="background: #8000E1A6;">La misma acción es ruidosa desde una cuenta que nunca la hace y silenciosa desde una que la hace a diario</mark>. Por eso el *blending* (mimetizarse con la actividad legítima: cuentas de administración, horarios, hosts de salto esperados) vence a cualquier *bypass* técnico aislado.

# La pirámide del dolor, del lado del atacante

[David Bianco](https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html) ordenó los indicadores por lo que le **cuesta al atacante** cambiarlos cuando lo queman. Leída al revés, es el mapa de qué evasión envejece bien:

```text
TTPs ───────────── lo más caro de cambiar para ti → si el defensor detecta AQUÍ, estás muerto
Herramientas ────  recompilar/cambiar de tool
Artefactos de red/host ─ nombres de pipe, rutas
Valores de dominio
Hashes de ficheros
IPs
Hashes/dominios ── trivial de cambiar
```

Aplicada a AD: ocultar el **hash** de tu binario (recompilar mimikatz) es barato pero el EDR te caza igual por el **comportamiento** (abrir un handle a LSASS con `PROCESS_VM_READ`). <mark style="background: #FFB86CA6;">La evasión que de verdad dura ataca la detección por **comportamiento y TTP**</mark>: no abrir LSASS como lo hace la herramienta estándar, no pedir 500 TGS de golpe, no lanzar `cmd.exe` desde un proceso de servicio. Por eso este sub-tema insiste en *cómo* se hace cada cosa, no en *con qué*.

# Las tres preguntas antes de cada acción

El mindset que CRTO evalúa y que conviene interiorizar. Antes de ejecutar **cualquier** técnica, respóndete:

1. **¿Qué indicador genera?** — evento de Windows (`4769`, `4662`, `4624`…), telemetría de EDR (handle a LSASS, proceso hijo anómalo), petición de red anómala (replicación desde un no-DC). Las notas siguientes lo detallan por técnica.
2. **¿Es anómalo para este principal/host?** — un `DCSync` desde un DC es normal; desde una workstation es una alerta de alta fidelidad. Un `Get-ADUser` desde la cuenta de un sysadmin es ruido de fondo; desde la de un becario, no.
3. **¿Puedo conseguir lo mismo con menos indicador?** — AES en vez de RC4, DCOnly en vez de barrer, un Diamond Ticket en vez de un Golden, leer solo el atributo que necesito en vez de `-Properties *`.

<mark style="background: #FF5582A6;">No existe "AD sigiloso" absoluto</mark>: DCSync y Golden Ticket, por ejemplo, se detectan sí o sí en un entorno con MDI bien puesto. Lo que decides no es *si* generas indicador, sino *cuándo*, *desde dónde* y *asumiendo qué riesgo* — y en un engagement con ventana acordada se documenta y se sigue; en un Red Team con objetivo de sigilo, cambia el plan.

# El mapa del sub-tema

Cada nota aplica este modelo a una fase, de la más temprana (y evitable) a la más ruidosa (e inevitable):

- [[40 - Recon y enumeración sigilosa]] — la fase más barata de arruinar y la más fácil de hacer bien.
- [[41 - OPSEC de Kerberos]] — roasting, encryption types, el downgrade RC4 que te delata.
- [[42 - Forjado de tickets y su detección]] — golden/silver/diamond/sapphire y cómo los caza el blue.
- [[43 - OPSEC de credential access]] — LSASS, PPL, Credential Guard; el frente donde el EDR es más duro.
- [[44 - Movimiento lateral, ejecución y deception]] — métodos de ejecución, su telemetría y los honeytokens que te esperan.

> [!info]+ Alcance: esto es AD-céntrico
> La evasión de **endpoint pura** (AMSI/ETW, inyección de procesos, *in-memory*, cargadores) y el **C2** (Cobalt Strike, perfiles malleable, redirectores) son el otro gran bloque de CRTO, pero pertenecen al área [[Evasión de defensas.base|Evasión de defensas]] (endpoint/EDR), hoy en pausa. Aquí se asume que ya ejecutas con un *beacon* razonablemente sigiloso y el foco es lo que pasa **sobre AD**.

> [!info]+ Fuentes
> [The Pyramid of Pain (David Bianco)](https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html), [MITRE ATT&CK — Credential Access / Lateral Movement / Discovery](https://attack.mitre.org/), [Microsoft Defender for Identity — alerts](https://learn.microsoft.com/en-us/defender-for-identity/alerts-overview). Sub-tema net-new (ADR 028), foco CRTO.
