---
tags:
  - Evasion
  - Windows
  - EDR
  - Kernel
Descripción: "El sensor que decide si consigues leer la memoria de lsass.exe, y por qué hoy la respuesta es no pedir el handle que espera"
Fecha de actualización: 2026-11-20
Nota previa: "[[09 - Manipulación de la imagen de proceso]]"
Nota siguiente: "[[11 - Image-load y registro - KAPC injection]]"
Area: "[[Callbacks de kernel.base|Callbacks de kernel]]"
---
---

Los callbacks de proceso e hilo te ven **nacer**; los *object callbacks* te ven **tocar**. En Windows casi todo recurso —fichero, proceso, token, clave de registro— se abstrae como un **objeto**, y de su ciclo de vida se encarga un broker centralizado: el <mark style="background: #ADCCFFA6;">**object manager**, que además avisa a los drivers registrados cuando alguien pide un *handle* a un proceso, a un hilo o a un escritorio</mark>. Como media tonelada de tradecraft pasa por abrir un handle —volcar credenciales, inyectar, matar el agente—, este es el sensor más rentable que tiene un EDR, y el que mejor aguanta.

# Cómo se registra y qué puede vigilar

Un driver se registra con una sola llamada, `nt!ObRegisterCallbacks()`, pasándole una `OB_CALLBACK_REGISTRATION`. Lo interesante de esa estructura no es la función en sí, es **cuánto acota el alcance**:

- `Altitude` — una cadena numérica que ordena la ejecución entre drivers. En *pre-operation* **corre antes el de altitud más alta**; en *post-operation*, al revés. Microsoft admite decimales (`"28133.08004"`) justamente para evitar colisiones entre vendors.
- `OperationRegistration` — array que declara, por cada entrada, **qué tipo de objeto** y **qué operación**.

El tipo de objeto solo admite **tres** valores, y esa es la primera grieta estructural del sensor:

| `ObjectType` | Qué cubre |
| --- | --- |
| `PsProcessType` | Handles a procesos — el 95 % de lo que registra un EDR |
| `PsThreadType` | Handles a hilos |
| `ExDesktopObjectType` | Handles a escritorios |

<mark style="background: #FFB86CA6;">No existe callback de objeto para **tokens**, **ficheros**, **secciones** ni **claves de registro**</mark>. Si tu técnica se apoya en duplicar un token privilegiado en vez de abrir un proceso, este sensor concreto es ciego por diseño (lo recoge otro: ver [[03 - Privilegios de token en Windows|privilegios de token]] y [[04 - SeImpersonate y la familia Potato|la familia Potato]]). Y la operación solo puede ser `OB_OPERATION_HANDLE_CREATE` (handle nuevo) u `OB_OPERATION_HANDLE_DUPLICATE` (duplicado de uno existente) — recuérdalo, porque el libro apoya media evasión en que muchos productos solo registraban la primera.

# El truco que define este sensor: recortar la máscara de acceso

Aquí está lo que hace a los object callbacks distintos de todo lo visto hasta ahora. En *pre-operation*, el callback recibe una `OB_PRE_OPERATION_INFORMATION` cuyo campo `Parameters` apunta a una estructura con **dos** máscaras de acceso:

```c
typedef struct _OB_PRE_CREATE_HANDLE_INFORMATION {
    ACCESS_MASK DesiredAccess;          // la que se CONCEDERÁ — el driver puede escribirla
    ACCESS_MASK OriginalDesiredAccess;  // la que TÚ pediste — solo lectura, para la telemetría
} OB_PRE_CREATE_HANDLE_INFORMATION;
```

<mark style="background: #8000E1A6;">El driver no tiene que denegarte el handle: puede concedértelo **recortado**</mark>. Si pides `PROCESS_ALL_ACCESS` sobre `lsass.exe`, el EDR apaga el bit de `PROCESS_VM_READ` en `DesiredAccess` (un `&= ~PROCESS_VM_READ`) y te devuelve un handle perfectamente válido… con el que `MiniDumpWriteDump` falla. Esto es literalmente el modo **deceive** de [[00 - Anatomía de un EDR|la nota 00]] materializado en una API documentada:

> [!danger]+ El fallo que te comerás y malinterpretarás
> `OpenProcess` devuelve un handle no nulo, `GetLastError()` dice `ERROR_SUCCESS`, y la lectura de memoria revienta después con `ERROR_ACCESS_DENIED` o `ERROR_PARTIAL_COPY`. <mark style="background: #FF5582A6;">Vas a pensar que tu código tiene un bug y vas a perder una hora depurando un recorte de máscara</mark>. La comprobación correcta no es "¿tengo handle?", es `NtQueryObject`/`GetHandleInformation` sobre los **derechos concedidos** antes de seguir. Y si los derechos llegan recortados, ya estás en el dashboard: la operación se registró igual.

En la variante de duplicado, `OB_PRE_DUPLICATE_HANDLE_INFORMATION` añade `SourceProcess` y `TargetProcess`, así que el driver sabe también **de quién a quién** viaja el handle.

# Saber quién te vigila

Los object callbacks no viven en un array plano como los de proceso, sino en una **lista doblemente enlazada** que cuelga del propio tipo de objeto: el campo `CallbackList` (offset `0x0c8` en builds recientes) de la `nt!_OBJECT_TYPE` a la que apunta `nt!PsProcessType`. Cada nodo es un `CALLBACK_ENTRY_ITEM` no documentado cuyo `PreOperation` está en `+0x28`. Con un kernel debugger se recorre en una línea:

```text
kd> !list -x ".if (poi(@$extret+0x28) != 0) { lmDva (poi(@$extret+0x28)); }" (poi(nt!PsProcessType)+0xc8)
fffff802`73b80000  WdFilter.sys     <-- Microsoft Defender
```

Es el complemento de la enumeración de `nt!PspCreateProcessNotifyRoutine` de la [[07 - Notificaciones de creación de proceso e hilo|nota 07]], y entre las dos tienes el inventario de drivers que te observan — la base del [[23 - Reconocimiento de defensas y perfilado del EDR|perfilado de defensas]]. Ojo: los offsets son **específicos de build**, no los hardcodees.

# Las tres suposiciones de la detección de LSASS

Toda la lógica anti-*credential-dumping* de un EDR se sostiene en tres premisas sobre el evento de handle:

1. El proceso llamante **abrirá un handle nuevo** a `lsass.exe`.
2. Ese proceso será **atípico** (no está en la lista de los que legítimamente tocan LSASS).
3. La **máscara pedida** incluirá lo necesario para leer memoria (`PROCESS_VM_READ` + `PROCESS_QUERY_INFORMATION`).

<mark style="background: #ADCCFFA6;">Cada evasión de este capítulo ataca una de las tres</mark>. Y de paso, la premisa 3 funciona en los dos sentidos: muchas herramientas públicas piden máscaras **excesivas** (`PROCESS_ALL_ACCESS`) o con una firma reconocible —Mimikatz pide exactamente `PROCESS_VM_READ | PROCESS_QUERY_LIMITED_INFORMATION`—, así que la propia máscara es un IOC de herramienta.

## Evasión 1 — robo de handle (duplicación)

Ataca la premisa 1: si ya existe un handle a LSASS en otro proceso, **duplícalo** en vez de abrir uno nuevo. Lo habilitador es que `ntdll!NtQuerySystemInformation` con la clase `SystemHandleInformation` deja **a un usuario sin privilegios** leer la tabla de handles de todo el sistema. El flujo es: recorrer la tabla → descartar los de `lsass.exe` y del proceso `System` (PID 4) para no autodelatarse → `OpenProcess(PROCESS_DUP_HANDLE)` sobre el dueño → `DuplicateHandle` → confirmar con `NtQueryObject(ObjectTypeInformation)` que es un objeto `Process` → comprobar con `QueryFullProcessImageNameW` que apunta a LSASS y que los derechos concedidos ya incluyen lo que necesitas.

> [!warning]+ Lo que el libro da por bueno y hoy no lo es
> Hand escribe que *«muchos sensores solo comprueban las peticiones de handle nuevas y no las de duplicado»*. <mark style="background: #FF5582A6;">Eso era cierto en 2023 y hoy es una suposición peligrosa</mark>: registrar `OB_OPERATION_HANDLE_DUPLICATE` es una línea de código, la técnica lleva años publicada y las dos estructuras llegan al mismo callback. Trátalo como «compruébalo en este cliente», no como un hueco fiable. Además sigue necesitando `PROCESS_DUP_HANDLE` sobre el proceso dueño — que es en sí mismo un evento de handle vigilado.

El valor residual real es otro: el handle duplicado te permite volcar a fichero y **procesar el dump fuera del objetivo** (`sekurlsa::minidump` en tu máquina), sacando del endpoint toda la parte ruidosa del parseo. Eso sigue siendo buen tradecraft → [[07 - Ataque a LSASS|ataque a LSASS]].

## Evasión 2 — correr más que el callback

Ataca el *timing*. En *«Fast and Furious: Outrunning Windows Kernel Notification Routines from User-Mode»* (DIMVA 2020), Ciholas *et al.* midieron la ventana entre la notificación de creación de proceso y la de object callback: <mark style="background: #FFB8EBA6;">**8,75–14,5 ms**</mark>. Si pides el handle dentro de esa ventana, el driver aún no ha podido recortarte la máscara y te quedas con un handle **íntegro**. Dos formas:

- **Job object sobre el padre conocido.** Creas un *job object* sobre el proceso padre (p. ej. `explorer.exe`, si el objetivo se lanza desde la GUI), lo asocias a un *I/O completion port* con `JobObjectAssociateCompletionPortInformation`, y consumes el puerto en bucle con `GetQueuedCompletionStatus`. <mark style="background: #ADCCFFA6;">La notificación al job llega **antes** que la del object callback</mark>, así que el PID del hijo te llega con ventaja y lo abres al instante.
- **Adivinar el PID.** Los PID se asignan en múltiplos de 4; descartando los ya existentes, el espacio a probar es pequeño. El PoC de los autores (`hThemAll.cpp`) lanza cuatro hilos haciendo `OpenProcess` sobre la lista filtrada. No cubre la reutilización de PID/TID tras una terminación.

> [!important]+ Dos límites que hay que tener claros
> 1. Esto derrota el control **preventivo**, no el **detectivo**: aunque el driver no llegue a recortarte la máscara, el evento se reporta igual. Ganas el acceso y pierdes el sigilo.
> 2. Contra el proceso del agente del EDR es casi inaplicable: arranca desde un servicio muy temprano en el orden de boot, y para adelantarlo necesitarías crear tu propio servicio (admin) sin garantía de ganar la carrera.
>
> Microsoft respondió que **no lo arreglará** por riesgo de compatibilidad de aplicaciones, y que la mitigación corresponde a los vendors. Por eso sigue vivo seis años después.

# Qué ha cambiado desde el libro

El terreno se ha movido en la dirección contraria a estas dos técnicas:

- **Defender bloquea el acceso a LSASS por defecto.** La regla ASR `9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2` (*Block credential stealing from the Windows local security authority subsystem*) pasó de *Not Configured* a **Block por defecto** en 2022. No es un object callback, es una capa aparte que deniega antes — y se enumera desde el endpoint ([[26 - Reglas ASR y hardening de Defender|nota 26]]).
- **LSASS como PPL y Credential Guard.** Con `RunAsPPL` o Credential Guard activos, el problema ya no es la máscara: es que el proceso está en otro nivel de protección o las credenciales no están ahí ([[27 - Credential Guard, VBS y HVCI|nota 27]]).
- **El tradecraft se movió a no pedir el handle que esperan.** Las tres premisas se esquivan mejor pidiendo una máscara **que al EDR no le parece peligrosa** y derivando la lectura a otro objeto:
  - **Fork / clonado** — `PROCESS_CREATE_PROCESS` + `NtCreateProcessEx` genera un clon de LSASS; el volcado se hace sobre el clon, no sobre el original.
  - **Snapshot** — `PssCaptureSnapshot` captura el espacio de direcciones con `PROCESS_QUERY_INFORMATION` + `PROCESS_VM_READ` sobre un objeto *snapshot* intermedio.
  - **`SilentProcessExit`** — se configura la clave de `Image File Execution Options` para que **`WerFault.exe`** (proceso de confianza, con derecho legítimo a tocar LSASS) genere el dump por ti.
  - `nanodump` ([fortra/nanodump](https://github.com/fortra/nanodump)) implementa estas variantes como BOF (`--fork`, `--snapshot`, `--duplicate`, `--silent-process-exit`, `--shtinkering`) y escribe el dump con firma inválida para que no lo reconozcan como minidump.
- **Y la respuesta estratégica: no tocar LSASS.** [[27 - DCSync|DCSync]], [[23 - Kerberoasting|Kerberoasting]], [[08 - Ataque a Credential Manager y DPAPI|DPAPI]] o [[06 - Ataque a SAM, SYSTEM y SECURITY|SAM/SYSTEM]] consiguen material de credenciales **sin generar un solo evento de handle a `lsass.exe`**. Es un [[03 - Tipos de bypass y la cadena de evasión|bypass perceptual]] frente a uno lógico, y por eso envejece mejor.

Fuentes: Matt Hand, *Evading EDR* cap. 4 · [Microsoft — `ObRegisterCallbacks`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-obregistercallbacks) y [`OB_PRE_CREATE_HANDLE_INFORMATION`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/ns-wdm-_ob_pre_create_handle_information) · [Ciholas et al., *Fast and Furious* (DIMVA 2020)](https://eprints.lancs.ac.uk/144565/1/dimva20_paper37_accepted.pdf) · [Microsoft — Attack surface reduction rules reference](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference) · [fortra/nanodump](https://github.com/fortra/nanodump).
