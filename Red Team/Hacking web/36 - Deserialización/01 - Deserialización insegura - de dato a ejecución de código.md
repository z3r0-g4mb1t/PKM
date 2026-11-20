---
tags:
  - Web/Red-Team
  - Deserializacion
  - Introduccion
Descripción: "Por qué deserializar datos controlados por el usuario es ejecución de código en potencia, los dos grados del ataque y dónde cae hoy en OWASP y CWE"
Fecha de actualización: 2026-11-20
Nota previa: "[[00 - Serialización y deserialización - el formato por dentro]]"
Nota siguiente: "[[02 - Identificar datos serializados y sus sinks]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

La regla cabe en una frase: <mark style="background: #FF5582A6;">si una aplicación deserializa datos que el usuario controla, hay una vulnerabilidad de deserialización en potencia</mark>. El ataque consiste en tomar los bytes serializados que genera la aplicación y modificarlos para beneficio propio, o directamente fabricar y enviar bytes nuevos. Lo que determina el impacto no es esa manipulación, sino lo que el deserializador haga con ella.

# Por qué esto no es «manipular un dato»

Un objeto no es solo sus valores. Es una instancia de una clase, y la clase trae **comportamiento**: constructores, destructores y, en varios lenguajes, hooks que se disparan automáticamente en momentos del ciclo de vida del objeto. Al reconstruir el objeto, el deserializador tiene que instanciar la clase, y al hacerlo activa parte de ese comportamiento sin que nadie lo invoque explícitamente.

<mark style="background: #8000E1A6;">De ahí sale el salto conceptual que organiza todo el tema: deserializar datos no confiables equivale a permitir que un atacante elija qué clases se instancian y con qué valores, dentro del proceso del servidor</mark>. El atacante no inyecta código directamente; inyecta un **objeto**, y el código ya está en la aplicación o en sus dependencias, esperando a ejecutarse cuando ese objeto cobre vida. Esta indirección es la que hace el tema contraintuitivo y la que lo vuelve tan potente: no hace falta que el servidor tenga un `eval`, basta con que tenga clases cuyos hooks hagan algo útil.

# Los dos grados del ataque

Todo el espectro se reduce a dos objetivos, de menor a mayor:

- **Object injection** — alterar el objeto que el servidor reconstruye para que lleve información distinta. El ejemplo canónico es un objeto de sesión que guarda el rol del usuario: si se controla, se cambia `user` por `admin` y la deserialización instala esa mentira como verdad. No se ejecuta código; se corrompe el estado. Desarrollo en [[04 - Object injection en PHP]] y [[09 - Object injection y RCE en Python]].
- **Remote Code Execution** — hacer que la propia reconstrucción del objeto desemboque en ejecución de comandos. Puede venir de un hook vulnerable en la clase objetivo (un `__wakeup` que llama a `shell_exec`), o de encadenar varias clases ajenas hasta llegar a un `sink` peligroso: una **cadena de `gadgets`**.

<mark style="background: #FFB86CA6;">Entre esos dos extremos hay más: según qué `magic methods` haya disponibles, la deserialización puede desembocar en SQL injection, LFI, SSRF o DoS</mark>. El objeto deserializado es un primitivo que alimenta a otro sink; la creatividad está en encontrar qué cadena de llamadas conecta la instanciación con algo aprovechable. Un `__get` que consulta la base de datos con una propiedad controlada es SQLi; un `__toString` que abre una URL con una propiedad controlada es SSRF.

# Un poco de historia, porque explica el presente

La deserialización se conoce como vector desde 2011, pero se volvió un tema de primera línea en 2015-2016 con el llamado *Java Deserialization Apocalypse*. El detonante fue una charla de Gabriel Lawrence y Chris Frohoff que explicó el mecanismo en detalle y liberó `ysoserial`, la herramienta que automatiza la generación de cadenas de `gadgets` en Java. A partir de ahí el patrón se reconoció en todos los lenguajes con serialización nativa.

La consecuencia para hoy es doble. Primero, que las cadenas de `gadgets` públicas existen y se mantienen: para PHP está [[07 - PHPGGC y cadenas de gadgets|PHPGGC]], para Java `ysoserial`, para .NET `ysoserial.net`. Segundo, que los vendors han ido cerrando la puerta por defecto —.NET eliminó `BinaryFormatter`, Java introdujo filtros de deserialización, las librerías YAML cambiaron su API a modo seguro—, lo que desplaza el problema hacia **código que usa APIs inseguras heredadas** y hacia **ecosistemas que aún no han cerrado la puerta**, señaladamente el de machine learning (ver [[11 - Pickle y la deserialización insegura de modelos]]).

# Dónde cae en los marcos

Situarlo importa para el informe y para justificar la severidad ante un cliente:

- **OWASP Top 10.** La deserialización insegura tuvo categoría propia (`A8:2017`) y después se fusionó en la categoría de fallos de integridad. La edición vigente es la de **2025** (ya release final), y ahí cae en <mark style="background: #ADCCFFA6;">`A08:2025 – Software or Data Integrity Failures`</mark> — nótese el cambio de nombre fino respecto a 2021, que decía *«Software **and** Data Integrity Failures»*. La categoría agrupa los fallos de integridad de software, de datos serializados y de la cadena de suministro. La trayectoria completa: `A8:2017 Insecure Deserialization` → `A08:2021 Software and Data Integrity Failures` → `A08:2025 Software or Data Integrity Failures`.
- **CWE.** El identificador preciso es <mark style="background: #ADCCFFA6;">`CWE-502: Deserialization of Untrusted Data`</mark> (catálogo CWE 5.0, 2026-10-28), que es el que se cita en los CVE y el que conviene usar en el reporte.

> [!important]+ El dato que vende la severidad al cliente
> `CWE-502` ocupa el puesto **#15** del CWE Top 25 de 2025 por volumen de CVEs — mediocre. Pero en la lista **CWE Top 10 KEV Weaknesses 2025**, ponderada por explotación real en el catálogo KEV de CISA, está en el **#5**, con 11 entradas KEV. Es decir: no abunda, pero <mark style="background: #FFB86CA6;">cuando aparece, se explota de verdad y en libertad</mark>. Ese contraste es el argumento fuerte para priorizar un hallazgo de deserialización en un informe.

# El patrón de reconocimiento

Operativamente, el atacante busca tres señales juntas. Si aparecen las tres, hay caso:

1. Un blob opaco viaja entre cliente y servidor —cookie, campo oculto, parámetro, cabecera.
2. Al decodificarlo (Base64, hex, URL-decode) aparece una estructura serializada reconocible.
3. Modificarlo cambia el comportamiento del servidor, o el servidor lo reconstruye en un objeto.

La segunda señal —reconocer el formato de un vistazo— y la primera —saber dónde mirar— son el objeto de la nota siguiente.

> [!info]+ Fuentes
> Historia y charla original: [«Marshalling Pickles», Frohoff & Lawrence, AppSecCali 2015](https://frohoff.github.io/appseccali-marshalling-pickles/). Clasificación: [OWASP Top 10:2025](https://top10.owasp.org/2025/) y su categoría [A08:2025](https://top10.owasp.org/2025/A08_2025-Software_or_Data_Integrity_Failures); [CWE-502](https://cwe.mitre.org/data/definitions/502.html) y [CWE Top 25 2025](https://cwe.mitre.org/top25/archive/2025/2025_cwe_top25.html). Material de partida: módulo *Introduction to Deserialization Attacks* de HTB Academy, con la clasificación por grados de impacto reorganizada y la taxonomía actualizada a 2025-2026.
