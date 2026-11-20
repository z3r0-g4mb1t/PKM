---
tags:
  - Active-Directory
  - Windows
  - Pentesting/Post-Explotacion
  - Tipo/Deteccion
Descripción: "El perfil de ruido de cada método de ejecución remota (PsExec/WMI/WinRM/DCOM), los logon types y pipes que delatan, y los honeytokens que el defensor deja en el camino"
Fecha de actualización: 2026-11-20
Nota previa: "[[43 - OPSEC de credential access]]"
Nota siguiente: ""
Area: "[[Evasión y OPSEC.base|Evasión y OPSEC]]"
---
---

Mover de un host a otro siempre deja rastro: una autenticación, una ejecución, a veces un servicio o una tarea nuevos. La OPSEC del movimiento lateral es elegir el **método con el perfil de ruido más bajo para el objetivo** y saber qué correla el defensor. Y, transversal a todo, reconocer las **trampas** (honeytokens, cuentas señuelo) que están puestas precisamente para que un atacante que "va a lo fácil" muerda. Los métodos de acceso en sí están en [[28 - Acceso privilegiado]]; esto es su cara de detección.

# El perfil de ruido de cada método de ejecución

No hay un método "sigiloso" universal; hay un método adecuado según qué vigila el objetivo. Lo que los distingue a ojos del defensor:

| Método | Mecanismo | Rastro característico |
| --- | --- | --- |
| **PsExec** (SMB) | Sube un binario a `ADMIN$`, crea y arranca un **servicio** | `7045` (servicio instalado), `4624` logon tipo 3, fichero en `ADMIN$`, named pipe `\psexecsvc`. **Ruidoso y muy firmado** |
| **WMI** (`wmiexec`) | `Win32_Process.Create` vía DCOM/RPC | `4688` (proceso creado) con padre `WmiPrvSE.exe`, `4624` tipo 3. Sin servicio ni fichero: más limpio que PsExec |
| **WinRM** (`evil-winrm`, PSRemoting) | HTTP(S) 5985/5986 | `4624` tipo 3, log operacional de WinRM, proceso hijo de `wsmprovhost.exe`. Normal en entornos que administran por WinRM |
| **DCOM** (MMC20, ShellWindows…) | Activación de objetos COM remotos | `4624` tipo 3, proceso hijo de `mmc.exe`/`explorer.exe`. Menos vigilado, más frágil |
| **Tarea programada / servicio** | Crear tarea o servicio remoto | `4698` (tarea creada) / `7045`. Útil para persistencia, ruidoso |

<mark style="background: #ADCCFFA6;">El factor que más pesa: un `cmd.exe`/`powershell.exe` cuyo **proceso padre** es `services.exe`, `WmiPrvSE.exe` o `wsmprovhost.exe` es la firma clásica de ejecución remota</mark> — el EDR correla esa cadena padre-hijo. PsExec añade encima un servicio (`7045`) y un named pipe con nombre conocido, por eso es el más detectado pese a ser el más cómodo.

> [!warning]+ El named pipe te delata aunque cambies el binario
> PsExec y muchos frameworks usan **named pipes** con nombres por defecto (`\psexecsvc`, o los pipes por defecto de Cobalt Strike como `\msagent_*`). El EDR los caza por nombre — es un indicador de la base de la [[39 - OPSEC en AD - el modelo de detección y el mindset|pirámide del dolor]], barato de cambiar pero letal si lo dejas por defecto. En C2, personalizar el nombre de pipe es OPSEC básica; con herramientas de impacket, saber que `-service-name` y el pipe por defecto son firmas.

# Logon types: el campo que lo cuenta todo

Cada autenticación genera un `4624` con un **Logon Type** que revela *cómo* entraste, y el defensor los correla para detectar movimiento anómalo:

| Type | Significado | Relevancia |
| --- | --- | --- |
| 2 | Interactivo (teclado físico) | — |
| 3 | Red (SMB, WMI, WinRM) | El del movimiento lateral; el más común y por eso el mejor para *blending* |
| 9 | NewCredentials (`runas /netonly`, overpass) | Delata uso de credenciales alternativas |
| 10 | RemoteInteractive (RDP) | Sesión interactiva remota |
| 11 | CachedInteractive | Login con credenciales cacheadas |

<mark style="background: #FFB86CA6;">La correlación que caza al atacante no es un `4624` suelto, sino el patrón</mark>: la misma cuenta autenticándose por red (tipo 3) en diez hosts en pocos minutos, o un DA con logon tipo 10 en una workstation (un admin de dominio **nunca** debería iniciar sesión interactiva en un endpoint — la base del modelo de *tiering*). El evento `4648` (logon con credenciales explícitas) es otra joya para el defensor: marca cuándo un proceso usa credenciales distintas a las de su sesión, firma de PtH/overpass.

# Deception: no muerdas el anzuelo

El defensor moderno no solo detecta; **tiende trampas**. Reconocerlas es OPSEC de CRTO, porque tocarlas es una alerta de altísima fidelidad (sin periodo de aprendizaje en MDI → dispara a la primera):

- **Honeytoken (cuenta/equipo señuelo)**: una cuenta "jugosa" etiquetada en MDI que **nadie usa legítimamente**. Cualquier interacción (kerberoastearla, autenticarte, consultarla por LDAP) dispara. Señales de que algo es un señuelo: <mark style="background: #FF5582A6;">SPN muy apetecible (`svc_backup_admin`) con `lastLogon` nulo o antiquísimo, cuenta privilegiada que nunca inicia sesión, contraseña sin cambiar en años pero habilitada</mark>.
- **Honey SPN**: un SPN falso colgado de una cuenta señuelo para cazar el reconocimiento previo al Kerberoasting.
- **Objetos con SACL de auditoría de lectura**: no son trampas visibles, pero convierten una simple lectura en un evento `4662` — razón de más para no barrer.
- **Canary / archivos señuelo** en shares: ficheros que al abrirse avisan (relacionado con el *Snaffling* de [[31 - Configuraciones erróneas varias]]).

La defensa contra la deception es la **validación antes de tocar**: ante una cuenta demasiado apetecible, comprueba su patrón de uso (`lastLogon`, `pwdLastSet`, pertenencia real) antes de interactuar. Lo que parece el *quick win* del engagement es, demasiado a menudo, el anzuelo.

# El checklist de movimiento lateral

1. **Elige el método por lo que vigila el objetivo**: WMI/WinRM se mimetizan mejor que PsExec donde se administra así; PsExec solo si el servicio extra no importa.
2. **Cuida la cadena padre-hijo y los named pipes**: no dejes nombres por defecto.
3. **Logon type 3 para blending**, y nunca un DA con logon interactivo en un endpoint.
4. **Valida antes de tocar** cualquier objetivo sospechosamente apetecible — puede ser un honeytoken.
5. **Reparte y usa cuentas esperadas**: diez autenticaciones por red desde una cuenta que ya lo hace a diario son ruido; desde una recién comprometida, un patrón.

Con esto se cierra el sub-tema de evasión. La contraparte resumida —qué telemetría deja cada ataque de todo el área, de un vistazo— está en [[37 - Detección y evasión en AD]]; el mindset que ordena todo, en [[39 - OPSEC en AD - el modelo de detección y el mindset]].

> [!info]+ Fuentes
> [Microsoft — 4624 Logon Types](https://learn.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types), eventos [`4648`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4648) y [`7045`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-7045), [impacket lateral tools](https://github.com/fortra/impacket), [MDI — honeytoken activity](https://learn.microsoft.com/en-us/defender-for-identity/alerts-mdi-classic). Métodos de acceso en [[28 - Acceso privilegiado]].
