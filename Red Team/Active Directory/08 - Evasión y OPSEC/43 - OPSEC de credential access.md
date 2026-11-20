---
tags:
  - Active-Directory
  - Windows
  - Pentesting/Post-Explotacion
  - Tipo/Deteccion
Descripción: "El frente donde el EDR es más duro: volcar LSASS sin que salte, y qué cambian PPL, RunAsPPL y Credential Guard sobre lo que puedes robar"
Fecha de actualización: 2026-11-20
Nota previa: "[[42 - Forjado de tickets y su detección]]"
Nota siguiente: "[[44 - Movimiento lateral, ejecución y deception]]"
Area: "[[Evasión y OPSEC.base|Evasión y OPSEC]]"
---
---

Robar credenciales de la memoria es el movimiento más rentable del movimiento lateral —una sesión de DA en un servidor cualquiera expone al DA entero ([[01 - Dónde viven las credenciales en Windows]])— y, a la vez, donde el EDR vigila con más intensidad. Esta nota es la OPSEC de ese frente: cómo se detecta el acceso a LSASS y qué cambian las protecciones modernas sobre lo que de verdad puedes extraer.

# Por qué tocar LSASS es tan ruidoso

El EDR no detecta "mimikatz" por su hash (recompilarlo es trivial — [[39 - OPSEC en AD - el modelo de detección y el mindset|pirámide del dolor]]); detecta el **comportamiento**: <mark style="background: #FF5582A6;">un proceso que abre un *handle* a `lsass.exe` con derechos de lectura de memoria (`PROCESS_VM_READ` | `PROCESS_QUERY_INFORMATION`)</mark>. Ese patrón lo vigilan los *callbacks* de kernel del EDR (`ObRegisterCallbacks`) y lo delata el evento `4656`/`4663` (acceso a objeto) sobre LSASS si la auditoría está activa, además de `Sysmon Event ID 10` (ProcessAccess a lsass). Es una de las telemetrías de más alta fidelidad de todo Windows.

De ahí que la evasión no sea "qué herramienta" sino "cómo toco LSASS":

| Técnica | Idea | Estado de detección 2026 |
| --- | --- | --- |
| `sekurlsa::logonpasswords` clásico | Abrir handle + leer memoria en vivo | Muy detectado (handle + firma de comportamiento) |
| Volcado a fichero (`comsvcs.dll MiniDump`, Task Manager) | Crear un `.dmp` y parsearlo **offline** | El dump se detecta igual (handle a LSASS); además crea un fichero sospechoso |
| **Handle duplicado / robado** | Reusar un handle a LSASS que ya tiene otro proceso, sin abrir uno nuevo | Evade la detección del *open*; más sigiloso |
| **Acceso por clonado de proceso** (`PssCaptureSnapshot`, fork) | Volcar un **clon** de LSASS, no LSASS directo | Evade parte de la telemetría de acceso directo |
| **Solo lo que necesitas** | No volcar todo: extraer un único ticket/hash | Menos tiempo con el handle abierto, menos ruido |

<mark style="background: #8000E1A6;">El principio: minimizar el *handle* a LSASS</mark> —no abrirlo (robar uno), abrirlo el menor tiempo posible, o trabajar sobre un clon— es lo que de verdad reduce la detección, no cambiar de binario. El parseo del volcado se hace **offline** en tu máquina (p. ej. `pypykatz` sobre el `.dmp`), nunca en el host.

> [!warning]+ El dump a fichero no es más sigiloso por ser "legítimo"
> Volcar LSASS con `comsvcs.dll` o el Administrador de tareas se vende como *living off the land*, pero **el indicador de alta fidelidad es el acceso a LSASS, que ocurre igual**, más un `.dmp` en disco que un FIM o el EDR marcan. "Usé una herramienta de Microsoft" no te salva aquí: lo vigilado es el *handle*, no el nombre del proceso.

# Qué cambian las protecciones modernas

Tres defensas han cambiado el panorama, y CRTO espera que sepas qué bloquea cada una:

- **PPL / RunAsPPL** (`Protected Process Light`): marca `lsass.exe` como proceso protegido, de modo que **ni siquiera un proceso con `SeDebugPrivilege` puede abrir un handle de lectura** a su memoria por la vía normal. <mark style="background: #FFB86CA6;">Sortearlo exige subir a kernel</mark> (cargar un driver vulnerable — *BYOVD*, muy ruidoso y cada vez más bloqueado por la *driver blocklist* de Microsoft) o usar técnicas que no pasen por el handle clásico. Es, hoy, el mayor freno al volcado de LSASS, y está activado por defecto en ediciones recientes.
- **Credential Guard**: aísla los secretos (hashes NT, tickets Kerberos, claves) en un proceso protegido por **virtualización** (VBS/VSM), fuera del espacio de LSASS. <mark style="background: #FF5582A6;">Con Credential Guard activo, volcar LSASS **no devuelve los NT hashes ni los TGT** de las sesiones</mark> — simplemente no están ahí. Cambia la estrategia por completo: no hay credenciales en memoria que robar, así que el foco se desplaza a capturar credenciales **en el momento del login** (keylogging, interceptar la autenticación) o a atacar lo que CG no protege (no cubre cuentas locales de la SAM, ni credenciales de aplicaciones/DPAPI, ni evita el abuso de tickets que tú mismo solicitas).
- **WDigest**: el responsable de que en sistemas viejos LSASS guardara la contraseña **en claro**. Deshabilitado por defecto desde Windows 8.1/2012 R2; encontrarlo activado (o poder reactivarlo) sigue siendo un *quick win*, pero es cada vez más raro.

> [!important]+ Comprueba las protecciones antes de tocar LSASS
> Antes de intentar un volcado, enumera qué defensas hay ([[07 - Enumeración de controles de seguridad]]): ¿LSASS es PPL (`RunAsPPL` en el registro)? ¿Credential Guard activo (`DeviceGuard` / `LsaCfgFlags`)? Lanzar un volcado clásico contra un LSASS protegido **no da credenciales y sí una alerta**. Si CG está activo, replantea: ataca la SAM local, DPAPI, o captura en el login — no pierdas un disparo contra memoria que está vacía.

# El checklist de credential access

1. **Enumera las protecciones primero** (PPL, Credential Guard, WDigest).
2. **Minimiza el handle**: robar/duplicar o clonar antes que abrir; extraer lo justo antes que volcar todo.
3. **Parsea offline**: el `.dmp` sale del host y se procesa en tu máquina.
4. **Si CG está activo**, cambia de frente (SAM, DPAPI, captura en login); no insistas contra LSASS.
5. **Prefiere robar tickets/claves a volcar hashes** cuando sea posible — operar por Kerberos ([[41 - OPSEC de Kerberos]]) es más silencioso que PtH con un NT hash volcado.

El último frente es llevar esas credenciales a otros hosts sin que la ejecución remota y el movimiento lateral te delaten — y los honeytokens que el defensor deja por el camino.

> [!info]+ Fuentes
> [Microsoft — Protecting LSA (RunAsPPL)](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection), [Credential Guard](https://learn.microsoft.com/en-us/windows/security/identity-protection/credential-guard/), [Sysmon Event ID 10 — ProcessAccess](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon), [microsoft-recommended driver block rules (BYOVD)](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/microsoft-recommended-driver-block-rules). Extracción concreta en [[07 - Ataque a LSASS]] (Pentesting). La evasión de endpoint pura (hooking, callbacks de kernel) está en el área [[Evasión de defensas.base|Evasión de defensas]].
