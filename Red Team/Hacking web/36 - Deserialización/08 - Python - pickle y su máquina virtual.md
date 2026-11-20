---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Por qué pickle es ejecución de código por diseño: su máquina de pila, el opcode REDUCE y cómo __reduce__ convierte un objeto en un comando"
Fecha de actualización: 2026-11-20
Nota previa: "[[07 - PHPGGC y cadenas de gadgets]]"
Nota siguiente: "[[09 - Object injection y RCE en Python]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

En PHP la inseguridad de `unserialize` es un efecto secundario de instanciar clases. En Python con `pickle` es el **diseño central**: <mark style="background: #FF5582A6;">un `pickle` no es un dato, es un programa</mark>. Deserializarlo no es reconstruir una estructura, es ejecutar `bytecode` en una pequeña máquina virtual. Por eso `pickle.loads` sobre datos no confiables es RCE directa, sin necesidad de `gadgets`, `magic methods` vulnerables ni frameworks.

# La máquina virtual de pickle

La propia documentación de CPython lo dice sin rodeos: un `pickle` es un *programa* para una máquina virtual de pickle (PM). La PM tiene una **pila** (LIFO) y una **memoria** (`memo`), y un objeto serializado es una secuencia de **`opcodes`** que la PM ejecuta para reconstruir el objeto encima de la pila.

Desensamblando el `pickle` del array `["HTB", 123, 7.77]`:

```python
>>> import pickle, pickletools
>>> pickletools.dis(pickle.dumps(["HTB", 123, 7.77]))
    0: \x80 PROTO      4          # versión de protocolo
    2: \x95 FRAME      ...
   11: ]    EMPTY_LIST            # mete [] en la pila
   12: \x94 MEMOIZE               # guarda la lista en memo
   13: (    MARK                  # marca el inicio de los elementos
   14: \x8c SHORT_BINUNICODE 'HTB'
   ...
   33: e    APPENDS              # añade todo desde MARK a la lista
   34: .    STOP                 # fin; el objeto queda en el tope
```

<mark style="background: #ADCCFFA6;">Cada `opcode` es una instrucción: `PROTO` fija la versión, `EMPTY_LIST` empuja una lista, `MEMOIZE` la cachea, `APPENDS` la rellena, `STOP` termina</mark>. La herramienta `pickletools.dis` es el desensamblador oficial y la vía para leer cualquier `pickle` sospechoso — el equivalente de `objdump` para este `bytecode`.

Lo que hace `pickle` peligroso no es ninguno de esos `opcodes`, sino dos en particular: `GLOBAL` (importa un objeto por nombre: `módulo.nombre`) y **`REDUCE`** (llama a un callable de la pila con unos argumentos de la pila). Con esos dos, un `pickle` puede importar `os.system` y llamarlo con una cadena. No hay nada que explotar: la capacidad de ejecutar es una característica documentada.

# `__reduce__`: el gancho del atacante

¿Cómo controla el atacante qué `opcodes` se generan sin escribir `bytecode` a mano? A través del método `__reduce__`. La documentación de `pickle` dice que, al serializar un objeto, si su clase define `__reduce__`, ese método devuelve una tupla de `(callable, args)` que la PM usará para reconstruirlo — emitiendo precisamente un `REDUCE`.

<mark style="background: #8000E1A6;">Es decir: el atacante define una clase cuyo `__reduce__` devuelve `(os.system, ("comando",))`, la serializa, y cuando la víctima la deserializa, la PM llama a `os.system("comando")`</mark>:

```python
import pickle, os

class RCE:
    def __reduce__(self):
        return os.system, ("id",)

payload = pickle.dumps(RCE())
```

Desensamblado, el `payload` es transparente:

```python
>>> pickletools.dis(payload)
    ... \x8c SHORT_BINUNICODE 'posix'      # GLOBAL posix.system
    ... \x8c SHORT_BINUNICODE 'system'
    ... \x8c SHORT_BINUNICODE 'id'         # el argumento
    ... R    REDUCE                        # system('id')
    ... .    STOP
```

La víctima solo tiene que llamar a `pickle.loads(payload)` y el comando corre. No hay condición adicional: `pickle.loads` sobre entrada controlada **es** la vulnerabilidad.

> [!warning]+ No existe «pickle seguro» por configuración
> A diferencia de PHP (`allowed_classes`) o YAML (`SafeLoader`), `pickle` no tiene un modo seguro que se active con un parámetro. La única forma de restringirlo es subclasear `pickle.Unpickler` y sobreescribir `find_class` para rechazar todo lo que no esté en una allowlist estricta — y hacerlo bien es difícil. La recomendación oficial de la documentación es tajante: **nunca deserializar datos de una fuente no confiable**. Detalle de la defensa en [[13 - Defensa - formatos seguros y restricción de clases]].

# Identificar pickle en el objetivo

Como se vio en [[02 - Identificar datos serializados y sus sinks]], un `pickle` se reconoce por sus bytes: empieza por `\x80` seguido del número de protocolo y termina en `.`. En Base64 —la forma habitual en cookies— eso es un token que empieza por `gAS` (protocolo 4: `\x80\x04\x95`) o `gAN` (protocolo 3).

<mark style="background: #FFB8EBA6;">El protocolo por defecto fue 4 de Python 3.8 a 3.13, y cambió a 5 en Python 3.14</mark> (el 5 existe desde 3.8 pero antes había que pedirlo). Ver el tercer byte acota la versión del intérprete, lo que orienta la construcción del `payload` y del bypass de filtros.

> [!warning]+ El marcador `gASV` se está quedando obsoleto
> El prefijo Base64 clásico de un `pickle` que OWASP y la mayoría de guías citan es `gASV` (de `\x80\x04\x95`, protocolo 4). Pero como Python 3.14 cambió el default a protocolo 5, <mark style="background: #FF5582A6;">todo `pickle` generado por un intérprete 3.14+ con los defaults empieza por `\x80\x05\x95` → Base64 `gAWV`, no `gASV`</mark>. Un IDS, regla de WAF o `grep` que solo busque `gASV` falla en silencio contra `payloads` modernos. Buscar los dos (`gASV` y `gAWV`), más `gAI=`/`gAM=` para los protocolos viejos.

El escenario del módulo —una cookie de sesión que es un objeto `pickle` deserializado en cada petición— es el patrón canónico, y el punto de partida de [[09 - Object injection y RCE en Python]].

> [!info]+ Fuentes
> Diseño de la máquina de pickle y advertencia de seguridad: [docs.python.org — pickle](https://docs.python.org/3/library/pickle.html) y los comentarios de [`Lib/pickletools.py`](https://github.com/python/cpython/blob/main/Lib/pickletools.py) y [`Lib/pickle.py`](https://github.com/python/cpython/blob/main/Lib/pickle.py). Mecánica de `__reduce__`: [docs.python.org — object.__reduce__](https://docs.python.org/3/library/pickle.html#object.__reduce__). Ejemplos adaptados del módulo 169 de HTB Academy, con el desensamblado añadido.
