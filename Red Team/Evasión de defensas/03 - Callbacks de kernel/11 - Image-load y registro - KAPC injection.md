---
tags:
  - Evasion
  - Windows
  - EDR
  - Kernel
Descripción: "Cómo el driver del EDR mete su DLL de hooking en tu proceso vía KAPC, y los huecos que el coste de rendimiento abre en los callbacks de registro"
Fecha de actualización: 2026-11-20
Nota previa: "[[10 - Object callbacks y robo de handles]]"
Nota siguiente: "[[12 - Minifilter drivers de filesystem]]"
Area: "[[Callbacks de kernel.base|Callbacks de kernel]]"
---
---

Quedan los dos últimos tipos de notificación del kernel, y van juntos por una razón práctica: <mark style="background: #ADCCFFA6;">la notificación de **image-load** es el mecanismo con el que el propio EDR inyecta su DLL de hooking en tu proceso</mark>, y la de **registro** es la que más telemetría genera y, precisamente por eso, la que más huecos deja. Entender la primera cierra el círculo del [[04 - Function-hooking DLLs|capítulo de hooking]]: ahora sabes de dónde sale esa DLL. La segunda enseña una lección transferible a todo el área — los compromisos de rendimiento del defensor son superficie de evasión.

# Notificaciones de image-load

El registro es trivial: `nt!PsSetLoadImageNotifyRoutine()` con la función de callback como único parámetro. El puntero acaba en el array `nt!PspLoadImageNotifyRoutine`, enumerable igual que el de procesos:

```text
kd> dx ((void**[0x40])&nt!PspLoadImageNotifyRoutine).Where(a => a != 0)
      .Select(a => @$getsym(@$getCallbackRoutine(a).Function))
  [0] : WdFilter+0x467b0                  <-- Defender
  [1] : ahcache!CitmpLoadImageCallback    <-- Customer Interaction Tracker
```

Verás **menos** callbacks aquí que en los de proceso: las notificaciones de proceso tienen usos no relacionados con seguridad (compatibilidad, telemetría de uso), mientras que el *image load* interesa casi en exclusiva a productos de seguridad. Si hay un EDR instalado, aparece aquí.

El callback recibe la ruta completa de la imagen y un puntero a `IMAGE_INFO`. Tres campos son los que mueven decisiones:

| Campo | Qué dice | Por qué te afecta |
| --- | --- | --- |
| `SystemModeImage` | `0` = se mapea en espacio de usuario (EXE/DLL); `1` = es un **driver** cargando en kernel | Distingue tooling de usuario de un intento de [[22 - BYOVD y kill del EDR\|carga de driver]], que pesa muchísimo más |
| `ImageSignatureLevel` | Nivel asignado por *Code Integrity*: `SE_SIGNING_LEVEL_UNSIGNED` (1), `AUTHENTICODE` (4), `ANTIMALWARE` (7), `MICROSOFT` (8), `WINDOWS` (12), `WINDOWS_TCB` (14)… | Es el dato con el que se construyen las políticas de "solo código firmado" y las reglas de *unsigned DLL en `%APPDATA%`* |
| `ImageSignatureType` | Cómo se aplicó la firma: `SeImageSignatureNone`, `…Embedded`, `…Cache`, `…CatalogCached` | Permite distinguir una firma incrustada de una validada por catálogo |

`ImagePartialMap` (desde Windows 10) marca que la imagen no se mapeó completa — típico de un `MapViewOfFile` de un trozo de un fichero enorme. Y `ImageBase` da la dirección base donde quedó mapeada.

> [!important]+ La imagen ya está mapeada cuando llega la notificación
> Si pones un breakpoint en `nt!PsCallImageNotifyRoutines` y miras los VAD del proceso, la DLL aparece ya como `Mapped`. <mark style="background: #8000E1A6;">Eso significa que el código está en el espacio de direcciones y listo para ejecutarse, pero el *loader* aún tiene pendiente resolver dependencias y llamar a `DllMain`</mark>. Es exactamente el hueco que usa el EDR en modo prevención para bloquear una DLL antes de que corra… y es el mismo hueco en el que se apoya el `KAPC injection` de abajo.

El uso de detección más citado: correlacionar la carga de **`clr.dll`** con atributos de la cabecera PE del proceso anfitrión. Un proceso que no es .NET y de pronto carga el *Common Language Runtime* está casi con certeza ejecutando un ensamblado en memoria — el patrón de `execute-assembly` ([[28 - Ejecución in-process de .NET y CLR|nota 28]]).

## Y la respuesta estratégica: no cargar nada en el host

La evasión más efectiva frente a este sensor no es técnica, es de diseño operativo: **no ejecutar el tooling en el objetivo**. Si `impacket`, `BloodHound` o cualquier utilidad que solo necesita ruta de red y credenciales corre en tu máquina y sale al entorno por un túnel, no hay imagen que cargar y desaparece una familia entera de indicadores de host. El coste es conocido y real: los túneles exigen latencia baja, y bajar el *sleep* del agente a casi cero convierte el canal de C2 en un flujo de volumen anómalo. <mark style="background: #FFB86CA6;">Es un intercambio de indicadores de host por indicadores de red</mark>, no una eliminación — y el libro propone separar el canal de tasking del canal de proxy (dos túneles) para no pagar los dos peajes a la vez. Mecánica completa en [[Pivoting y túneles.base|pivoting y túneles]] y [[15 - Detección y evasión de túneles|su detección]].

# KAPC injection: cómo entra la DLL del EDR

<mark style="background: #ADCCFFA6;">No existe forma soportada de inyectar una DLL en un proceso desde modo kernel</mark>. Así que los vendors usan la misma técnica que el malware que persiguen: **inyección por APC**, encolada desde el kernel (`KAPC`). El flujo aprovecha que `ntdll.dll` es de lo primero que carga cualquier proceso nuevo:

1. El driver se registra para image-load y **espera la carga de `ntdll.dll`** en el proceso recién creado. La notificación llega en el contexto del proceso objetivo, antes de que el hilo principal ejecute nada.
2. Resuelve el puntero a `ntdll!LdrLoadDll()` (no documentada, equivalente kernel-friendly de `LoadLibrary`) **en la copia ya mapeada** de `ntdll`. Por eso conviene un callback *post-operation*: la imagen está completa y, al estar en el proceso actual, **el ASLR deja de ser un problema**.
3. Reserva memoria en el objetivo para un contexto con el nombre de la DLL:

```c
ZwAllocateVirtualMemory(ZwCurrentProcess(), (PVOID*)&ctx, 0,
                        sizeof(INJECTION_CTX), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
RtlUnicodeStringCopyString(&ctx->Dll, L"hooks.dll");
```

4. Reserva el `KAPC` **en `NonPagedPool`**:

```c
PKAPC pKapc = (PKAPC)ExAllocatePoolWithTag(NonPagedPool, sizeof(KAPC), 'CPAK');
```

> [!warning]+ Por qué `NonPagedPool` y no `PagedPool`
> El hilo destino puede estar corriendo a un IRQL alto (`DISPATCH_LEVEL`), y a ese nivel **no se puede tocar memoria paginable**: provoca un bug check `IRQL_NOT_LESS_OR_EQUAL`, o sea BSOD. Es el tipo de detalle que explica por qué un driver de EDR mal escrito tumba flotas enteras — y por qué los vendors son tan conservadores tocando el kernel.

5. Inicializa y encola el APC:

```c
KeInitializeApc(pKapc, KeGetCurrentThread(), OriginalApcEnvironment,
                (PKKERNEL_ROUTINE)OurKernelRoutine,   // corre en kernel a APC_LEVEL; libera el KAPC
                NULL,                                  // RundownRoutine: si el hilo muere antes
                (PKNORMAL_ROUTINE)pfnLdrLoadDll,       // corre en USERLAND a PASSIVE_LEVEL
                UserMode, NULL);
KeInsertQueueApc(pKapc, &ctx->Dll.Buffer, &ctx->Dll.Length, 0);
```

El APC se entrega cuando un hilo del proceso entra en estado *alertable* (un `SleepEx`, un `WaitForSingleObjectEx`). A partir de ahí la DLL está dentro y los hooks puestos.

## Impedir la inyección, y por qué ya no basta

Desde la build **10586** de Windows 10, un proceso puede rechazar DLLs no firmadas por Microsoft con la política de mitigación `PROCESS_CREATION_MITIGATION_POLICY_BLOCK_NON_MICROSOFT_BINARIES_ALWAYS_ON` — Microsoft la añadió para que los navegadores se protegieran de DLL de terceros inestables, y es el `blockdlls` de Cobalt Strike. La carga rechazada devuelve `STATUS_INVALID_IMAGE_HASH`. Mecánica y detección en [[05 - Unhooking y remapping de ntdll|la nota 05]]; aquí van los tres matices que pertenecen a este capítulo:

- **Solo aplica al proceso hijo.** El atributo viaja en el `STARTUPINFOEX` de la creación, no al proceso actual. Para el proceso propio existe `SetProcessMitigationPolicy` con `ProcessSignaturePolicy`, pero <mark style="background: #FF5582A6;">cuando tu código llega a ejecutar esa llamada, la DLL del EDR ya está cargada y los hooks ya están puestos</mark>: llega tarde por construcción. Por eso encaja de forma natural con arquitecturas *fork&run*, donde cada tarea crea un proceso sacrificial nuevo.
- **Los vendors se han puesto a salvo.** Varios firman su DLL por **attestation signing** de Microsoft (el libro muestra la DLL de CrowdStrike Falcon con contrafirma de Microsoft), con lo que pasa el filtro y `blockdlls` deja de servir contra ese producto.
- **ACG como alternativa.** Adam Chester, en *«Protecting Your Malware with blockdlls and ACG»*, usa `PROCESS_CREATION_MITIGATION_POLICY_PROHIBIT_DYNAMIC_CODE_ALWAYS_ON` (*Arbitrary Code Guard*) para impedir que se modifiquen regiones ejecutables — requisito para poner un hook. Funciona, pero rompe a la mayoría de agentes comerciales, porque casi todos marcan páginas como `RWX` para su propio shellcode. Es una mitigación que te dispara a ti en el pie.

# Notificaciones de registro

Se registran con `nt!CmRegisterCallbackEx()` (`Cm` = *configuration manager*, el componente del kernel que gestiona el registro) y el callback tiene una firma deliberadamente opaca:

```c
NTSTATUS ExCallbackFunction(PVOID CallbackContext, PVOID Argument1, PVOID Argument2);
// Argument1 -> miembro de REG_NOTIFY_CLASS: QUÉ operación
// Argument2 -> puntero a la estructura específica de ESA operación
```

`REG_NOTIFY_CLASS` tiene 62 miembros, pero los prefijos `RegNt`, `RegNtPre` y `RegNtPost` describen la misma actividad en momentos distintos: deduplicando quedan **24 operaciones únicas** (`CreateKey(Ex)`, `SetValueKey`, `DeleteKey`, `EnumerateKey`, `QueryValueKey`, `LoadKey`, `RestoreKey`, `SaveKey`, `SetKeySecurity`…). Las *pre-operation* permiten **modificar o abortar** la petición; las *post-operation* usan `REG_POST_OPERATION_INFORMATION`, cuyo campo `ReturnStatus` se devuelve al llamante si el callback retorna `STATUS_CALLBACK_BYPASS`.

El mapa de por qué le importa al EDR:

| Técnica | Clave | Operaciones vigiladas |
| --- | --- | --- |
| Persistencia por Run key | `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | `CreateKey(Ex)`, `SetValueKey` |
| Persistencia por SSP | `HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Security Packages` | `SetValueKey` |
| COM hijacking | `HKLM\SOFTWARE\Classes\CLSID\<CLSID>\` | `SetValueKey` |
| Secuestro de servicio (`ImagePath`) | `HKLM\SYSTEM\CurrentControlSet\Services\<nombre>` | `SetValueKey` |
| Envenenamiento LLMNR | `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient` | `QueryValueKey` |
| Volcado de SAM | `HKLM\SAM` | `RegNt(Pre/Post)SaveKey` |

## El peaje de rendimiento es la grieta

<mark style="background: #ADCCFFA6;">El kernel **no deja filtrar** estas notificaciones: el driver recibe *todos* los eventos de registro del sistema y filtra él</mark>. Y el volumen es brutal: en la medición del libro, una VM de Windows **en reposo** generó **19.833 operaciones de registro en un minuto**. Si el driver añade 1 ms de procesado por evento, son ~30 % de degradación del sistema. Ningún vendor puede permitírselo, así que todos recortan: solo ciertas claves, solo ciertos tipos de operación. <mark style="background: #FFB86CA6;">Cada recorte es un punto ciego</mark>:

- **Solo `HKLM`** → ciego a `HKCU`/`HKU`. La persistencia por usuario no se ve.
- **Solo creación de claves** → ciego a *restauración* y *carga de hive*. El libro descompila un producto real y encuentra que no procesa `RegNtPreLoadKey` ni `RegNtPostLoadKey`: se puede crear y poblar la clave de un servicio nuevo con **`RegLoadKey`** (cargando un hive desde fichero como subclave) en vez de `RegCreateKey` + `RegSetValue`, y el servicio nace sin evento.
- Y un detalle de implementación que el libro documenta con la decompilación y que es oro:

> [!success]+ Las operaciones de registro **fallidas** no generan telemetría útil
> Los manejadores *post-operation* resuelven el nombre de la clave llamando a `nt!ObQueryNameString()` sobre el puntero `Object` que les llega. Pero si la operación falló (`Status != STATUS_SUCCESS`), <mark style="background: #FF5582A6;">ese puntero ya no es válido y el nombre no se puede extraer</mark>, así que el driver corta por lo sano y descarta el evento. El resultado es un registro del tipo *«este usuario intentó esta acción de registro a esta hora y falló»*, sin la clave: inservible para crear una detección. El mismo patrón aparece con `nt!CmCallbackGetKeyObjectIDEx()`.
>
> Operativamente: **el coste de un intento fallido contra el registro es casi cero**. Enumerar a ciegas, probar rutas o equivocarse al teclear una clave no te quema. Cambia el cálculo de riesgo de cualquier reconocimiento basado en registro.

# Qué ha cambiado desde el libro

- **El KAPC injection pierde centralidad.** Existe para meter la DLL de hooking, y el hooking de userland ya no es la columna vertebral de los productos punteros ([[04 - Function-hooking DLLs|nota 04]]). Defender for Endpoint apenas inyecta; CrowdStrike, SentinelOne y Cortex se sostienen en driver + [[16 - EtwTi y Protected Processes|ETW-TI]]. Sigue importando saber que existe, porque explica por qué `blockdlls` funciona contra unos productos y no contra otros.
- **Telemetría redundante de registro por ETW.** Además del callback `Cm*`, el provider **`Microsoft-Windows-Kernel-Registry`** entrega eventos de registro sin coste de driver, y `Sysmon` expone los EID **12** (creación/borrado de clave), **13** (`SetValue`) y **14** (renombrado). Un hueco en el callback del EDR no garantiza silencio si el entorno tiene Sysmon o reenvía ese provider ([[14 - Event Tracing for Windows|nota 14]]).
- **Image-load: `Sysmon` EID 7** es la materialización pública de `IMAGE_INFO`, incluidos firma y nivel de firma. Es la fuente que caza la doble carga de `ntdll` del [[05 - Unhooking y remapping de ntdll|unhooking]].
- **El ataque directo al driver se trata aparte.** El libro cierra el capítulo con el *callback entry overwrite* (localizar el array de callbacks y sobrescribir el primer byte de la rutina con un `RETN`, `0xC3`, como hace `mimidrv`). Eso exige driver propio o escritura arbitraria en kernel, depende de offsets por build, y HVCI le ha subido el precio: va completo en [[22 - BYOVD y kill del EDR|la nota 22]].

Fuentes: Matt Hand, *Evading EDR* cap. 5 · [Microsoft — `PsSetLoadImageNotifyRoutine`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/ntddk/nf-ntddk-pssetloadimagenotifyroutine) · [Microsoft — `IMAGE_INFO`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/ntddk/ns-ntddk-_image_info) · [Microsoft — `CmRegisterCallbackEx`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-cmregistercallbackex) y [`REG_NOTIFY_CLASS`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/ne-wdm-_reg_notify_class) · [Microsoft — mitigation policies en `UpdateProcThreadAttribute`](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-updateprocthreadattribute) · [Adam Chester — Protecting Your Malware with blockdlls and ACG](https://blog.xpnsec.com/protecting-your-malware/) · [gentilkiwi/mimikatz — mimidrv](https://github.com/gentilkiwi/mimikatz).
