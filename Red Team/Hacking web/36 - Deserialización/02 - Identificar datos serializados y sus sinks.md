---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Enumeracion
Descripción: "Reconocer datos serializados de un vistazo por su firma y localizar la función deserializadora en código fuente, por lenguaje"
Fecha de actualización: 2026-11-20
Nota previa: "[[01 - Deserialización insegura - de dato a ejecución de código]]"
Nota siguiente: "[[03 - PHP - del sink al laboratorio local]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

La vulnerabilidad tiene dos extremos: el **dato** serializado que entra y el **`sink`** que lo deserializa. En whitebox se busca el `sink`; en blackbox se reconoce el dato. Esta nota reúne las dos tablas de reconocimiento, que son material de consulta rápida durante un `engagement`.

# Blackbox: la firma del dato

Cada serializador deja una huella reconocible al principio de los bytes. Decodificar cualquier blob opaco (Base64, hex, URL) y comparar contra esta tabla identifica el lenguaje y a menudo la versión:

| Pista | Serializador |
| --- | --- |
| `a:3:{i:0;s:4:"Test";...}` | PHP (array) |
| `O:8:"stdClass":...` | PHP (objeto) |
| Bytes `80 04 95` + termina en `.` | Python `pickle` protocolo 4 (defecto 3.8+) |
| Bytes `80 05 95` + termina en `.` | Python `pickle` protocolo 5 (3.8+, opcional) |
| `80 03` / `80 02` / `80 01` + `.` | `pickle` protocolos 3 / 2 / 1 |
| `(lp0\nS'Test'\np1\na...` | `pickle` protocolo 0 (texto, Python 2) |
| `AC ED 00 05` (hex) o `rO0AB` (Base64) | Java `ObjectOutputStream` |
| `00 01 00 00 00 FF FF FF FF` o `AAEAAAD/////` | .NET `BinaryFormatter` |
| `04 08` (hex) | Ruby `Marshal` |
| `{"py/object": ...}` o `{"py/reduce": ...}` | Python `jsonpickle` |
| `!!python/object/apply:` en YAML | Python `PyYAML`/`ruamel` inseguro |

<mark style="background: #ADCCFFA6;">La firma de Base64 es el atajo que más se usa</mark>: un token que empieza por `rO0AB` es Java casi con certeza, uno que empieza por `gASV` es `pickle` protocolo 4 (`80 04 95` en Base64), y `Tz` o `YT` suele ser un objeto/array PHP Base64. Memorizar esos tres prefijos cubre la mayoría de los casos de web.

> [!warning]+ La versión de pickle delata la de Python
> El tercer byte de un `pickle` es el número de protocolo, y cada protocolo tiene una versión mínima de Python. `80 05` implica Python ≥ 3.8 en el que alguien pidió explícitamente el protocolo 5; `80 04` es el defecto desde 3.8. Eso acota la versión del intérprete objetivo antes de tocar nada, lo que importa para elegir qué `gadgets` y qué bypass de filtro usar.

Hay extensiones de Burp que automatizan este reconocimiento sobre todo el tráfico; la referencia es **Freddy** (Burp BApp Store), que detecta y ayuda a explotar serialización Java y .NET. Estado y alternativas, en [[14 - Arsenal de herramientas de deserialización]].

# Whitebox: la función que deserializa

Con el código delante, el reconocimiento es un `grep` por el nombre del `sink`. La lista por lenguaje:

| Lenguaje | Funciones a buscar |
| --- | --- |
| PHP | `unserialize()`, y todo acceso a ficheros con `phar://` |
| Python | `pickle.loads` / `pickle.load`, `cPickle.loads`, `_pickle.loads` |
| Python (otras libs) | `jsonpickle.decode`, `yaml.load` (sin `SafeLoader`), `yaml.unsafe_load`, `ruamel.yaml.YAML(typ='unsafe')` |
| Python (ML) | `torch.load`, `joblib.load`, `numpy.load(..., allow_pickle=True)`, `pandas.read_pickle` |
| Java | `readObject`, `readResolve`, `readExternal`, `XMLDecoder.readObject`, `ObjectInputStream` |
| .NET | `BinaryFormatter.Deserialize`, `NetDataContractSerializer`, `LosFormatter`, `ObjectStateFormatter`, `JavaScriptSerializer` con `SimpleTypeResolver`, `fastJSON`, `Json.NET` con `TypeNameHandling != None` |
| Ruby | `Marshal.load`, `YAML.load`, `Oj.load` en modo `object` |
| Node.js | `node-serialize`, `funcster`, `serialize-to-js` |

<mark style="background: #FFB86CA6;">El `sink` más peligroso es siempre el que no se parece a uno</mark>: el `phar://` de PHP (una operación de ficheros que deserializa), el `TypeNameHandling` de Json.NET (un parser de JSON que instancia clases arbitrarias), o el `readObject` escondido en una librería de terceros que se invoca al parsear un formato aparentemente inocuo.

## El detalle de cada lenguaje que cambia la explotación

- **PHP** — `unserialize($x)` es un `sink`; `unserialize($x, ['allowed_classes' => false])` no lo es (desde 7.0). `json_decode` nunca es un `sink` porque no instancia clases del proyecto.
- **Python** — `pickle.loads` es ejecución de código por diseño; no hay versión «insegura» que evitar, la inseguridad es intrínseca. `yaml.load(x)` sin `Loader` dejó de funcionar en PyYAML 6.0 (lanza `TypeError`), lo que convierte en errata el ejemplo clásico `yaml.load(exploit)` a secas — detalle en [[11 - Más allá de pickle - JSONPickle, PyYAML y ruamel]].
- **Java** — la clave no es solo que haya `readObject`, sino **qué hay en el classpath**: sin `gadgets` disponibles (bibliotecas como Commons-Collections, Spring, Groovy) el `readObject` es inofensivo. La explotación empieza por inventariar dependencias.
- **.NET** — `BinaryFormatter` quedó obsoleto y fue **eliminado** en las versiones modernas de .NET; encontrarlo implica que el proyecto corre sobre .NET Framework o deshabilitó la retirada a propósito, lo cual es en sí un hallazgo.

> [!important]+ El `sink` sin integridad por el camino es la condición que importa
> Encontrar `unserialize($userdata)` no basta para gritar RCE. Hay que seguir el flujo del dato: si entre la fuente y el `sink` hay un HMAC verificado, un descifrado autenticado o una restricción de clases, el camino está cerrado. La vulnerabilidad es la **conjunción** de dato controlado + `sink` + ausencia de integridad. Esa verificación de flujo es el núcleo del análisis whitebox dirigido por `taint` descrito en [[02 - Code review - alcance, priorización y lectura]].

# De la identificación a la explotación

Identificado el par dato/`sink`, el camino se bifurca por lenguaje. PHP arranca en [[03 - PHP - del sink al laboratorio local]]; Python, en [[08 - Python - pickle y su máquina virtual]]. En ambos, el primer movimiento es montar un laboratorio local que reproduzca el `sink` para iterar el `payload` sin tocar el objetivo.

> [!info]+ Fuentes
> Firmas de formatos serializados: documentación de cada serializador —[`pickle`](https://docs.python.org/3/library/pickle.html), [detección de objetos Java serializados (Max Chadwick)](https://maxchadwick.xyz/blog/java-serialized-object-detection)— y la [Deserialization Cheat Sheet de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html). Extensión Freddy: [PortSwigger BApp Store](https://portswigger.net/bappstore/ae1cce0c6d6c47528b4af35faebc3ab3). Catálogo de `sinks` ampliado respecto al módulo 169 de HTB con los `sinks` indirectos de cada lenguaje.
