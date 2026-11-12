---
tags:
  - OSINT/Fundamentos
  - Pentesting/Enumeracion
Descripción: "Cómo corroborar un hallazgo, documentar su procedencia y no caer en datos envenenados, deepfakes ni sesgos del analista"
Fecha de actualización: 2026-11-08
Nota previa: "[[04 - Marco legal y ético del OSINT]]"
Nota siguiente:
Area: "[[Fundamentos OSINT.base|Fundamentos OSINT]]"
---
---

El OSINT falla no por falta de datos, sino por **datos falsos tratados como ciertos**. Un dossier impecable construido sobre un dato envenenado lleva a un ataque fallido —o peor, a un *honeypot*—. Esta nota es la higiene que separa la inteligencia de la especulación.

# Corroborar: la regla de las dos fuentes

<mark style="background: #ADCCFFA6;">Un hallazgo de una sola fuente no corroborada es una hipótesis, no un hecho</mark>. La corroboración exige una **segunda fuente independiente** — independiente de verdad, no la misma noticia copiada en tres webs ni tres cuentas que se citan entre sí (*circular reporting*, el error clásico).

Ejemplo aplicado: encuentras que el correo de ACME sigue el patrón `inicial+apellido@acme.com` en **una** oferta de empleo. Antes de generar 300 correos con ese patrón, confírmalo con una segunda fuente independiente: un correo real visto en un `WHOIS` antiguo, una fuga de `git`, o la verificación contra el servidor de correo (`SMTP VRFY`/O365). Dos fuentes que coinciden → patrón fiable; una sola → `B3` (posiblemente cierto), úsalo pero sabiendo que puede fallar.

Califica cada dato con el [[03 - Taxonomía de fuentes y disciplinas OSINT#Fiabilidad: el sistema Admiralty|código Admiralty]] y **anótalo en el dossier**. "Correo confirmado `A1`" y "correo deducido `C3`" se tratan distinto en la fase de ataque.

# Procedencia: documenta en el momento

Cada dato necesita su **cadena de procedencia**: de dónde salió, cuándo, y una captura o copia. Dos razones:

- **Verificabilidad**: un dato sin fuente localizable no se puede re-comprobar ni defender en un informe. "Lo leí en algún sitio" no es inteligencia.
- **Volatilidad**: lo público desaparece. Un tuit se borra, un perfil se cierra, una web cambia. <mark style="background: #FFB8EBA6;">Si no capturaste la evidencia al verla, puede que ya no exista cuando la necesites</mark>. Captura con fecha (y, para algo delicado, archívalo en Wayback o con una captura con *hash*).

# Datos envenenados: cuando la fuente te tiende una trampa

El defensor moderno siembra **cebos** precisamente para detectar al que hace OSINT:

- <mark style="background: #FFB86CA6;">***Honeytokens* y *canary tokens***</mark>: credenciales, documentos, subdominios o entradas DNS falsas que, al usarlas o resolverlas, **alertan al defensor**. Un `config.php` con credenciales "olvidado" en un repo puede ser el anzuelo. Úsalo y delatas tu operación.
- **Desinformación deliberada**: perfiles de empleado falsos, infraestructura señuelo (*honeypots*), datos plantados para desviar al atacante. Si algo parece **demasiado fácil** (credenciales de administrador en texto plano en la primera búsqueda), sospecha.

La defensa: corrobora antes de **actuar** sobre un hallazgo; cuanto más jugoso y más fácil, más escrutinio. Esto conecta con la perspectiva defensiva en [[Contra-OSINT.base|contra-OSINT]].

# Contenido sintético: el problema de 2026

La corroboración visual ya no basta. <mark style="background: #FFB86CA6;">Las imágenes, voces y perfiles generados por IA son indistinguibles a simple vista</mark>, y un objetivo puede tener perfiles falsos (propios o plantados). Contramedidas prácticas:

- **Búsqueda inversa de imagen** siempre (una foto de perfil "de stock" o generada aparece en otros sitios o en ninguno — ver [[GEOINT e imágenes.base|GEOINT]]).
- **Señales de generación**: fondos incoherentes, manos/orejas/gafas defectuosas, metadatos ausentes o imposibles.
- **Procedencia `C2PA`**: el estándar de credenciales de contenido (Content Credentials) firma el origen de una imagen; su ausencia no prueba nada, pero su presencia ayuda.
- **Cuentas demasiado nuevas o demasiado limpias**: un perfil creado hace tres semanas con 2.000 seguidores y sin historia huele a fabricado.

# Los sesgos del analista

El último enemigo eres tú. Los sesgos que arruinan un análisis OSINT:

- <mark style="background: #8000E1A6;">**Sesgo de confirmación**</mark>: buscar solo lo que confirma tu hipótesis e ignorar lo que la contradice. El antídoto: formula qué dato **refutaría** tu conclusión y búscalo.
- **Anclaje**: fijarte en el primer dato y no actualizar con los siguientes.
- **Mirror-imaging**: asumir que el objetivo piensa/nombra/organiza como tú (que su formato de correo o su stack es "el obvio").

> [!important]+ Cierre del método
> Dirección → recolección → procesamiento → análisis → difusión, con cada dato **calificado, corroborado y documentado**. Con el método claro, el resto del área es *cómo* recolectar en cada dimensión. El primer paso antes de recolectar nada en serio: proteger tu propia identidad — el [[OPSEC OSINT.base|OPSEC del investigador]].

> [!info]+ Fuentes
> Corroboración y *circular reporting*: doctrina de análisis de inteligencia (Heuer, *Psychology of Intelligence Analysis*, CIA). Sesgos cognitivos del analista: ídem. *Canary tokens*: [canarytokens.org](https://canarytokens.org) (Thinkst). Procedencia de contenido: estándar [C2PA](https://c2pa.org) / Content Credentials.
