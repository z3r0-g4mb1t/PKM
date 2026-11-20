---
tags:
  - Web/Red-Team
  - Deserializacion
  - Tipo/Defensa
Descripción: "Por qué el HMAC es un parche y no la cura, la jerarquía real de defensas contra deserialización y cómo migrar a formatos de datos seguros"
Fecha de actualización: 2026-11-20
Nota previa: "[[12 - Detección y evasión de deserialización]]"
Nota siguiente: "[[14 - Arsenal de herramientas de deserialización]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

La defensa tiene una jerarquía clara, y el error habitual —el que comete el propio módulo al presentar el HMAC como solución— es quedarse en el escalón equivocado. <mark style="background: #FF5582A6;">La única defensa que cierra la clase de vulnerabilidad es no deserializar datos no confiables; todo lo demás es mitigación compensatoria con agujeros conocidos</mark>.

# La jerarquía, de mejor a peor

| Nivel | Defensa | Qué cierra | Agujero |
| --- | --- | --- | --- |
| 1 | **No deserializar entrada de usuario** | La clase entera | Ninguno — es la cura |
| 2 | **Formato de datos puro** (JSON/XML sin tipos) | La instanciación de clases | Hay que rehacer la lógica |
| 3 | **Allowlist de clases** (`allowed_classes`, `find_class`) | Las clases no esperadas | Difícil de acertar; un `gadget` en la allowlist basta |
| 4 | **Integridad** (HMAC, cifrado autenticado) | La falsificación por el atacante | Cae si se filtra la clave |
| 5 | **Blacklist / WAF** | Los `payloads` conocidos | Infinitos bypass |

El módulo enseña los niveles 4 y 2. Conviene entender por qué el 4 no basta y por qué el 2 es el objetivo realista.

# Por qué el HMAC es un parche

El módulo parcha `HTBooks` firmando la cookie con un HMAC: al emitirla se calcula `HMAC(clave, pickle)` y se adjunta; al recibirla se recalcula y se compara antes de `pickle.loads`.

```python
def cookieToSession(cookie):
    pickled_b64, hmac_given_b64 = cookie.split(".")
    pickled = base64.b64decode(pickled_b64)
    hmac_expected = hmac.new(SECRET_KEY.encode(), pickled, hashlib.sha512).digest()
    if hmac_expected != base64.b64decode(hmac_given_b64):
        return None
    return pickle.loads(pickled)   # <-- sigue siendo pickle.loads
```

Esto **sí** frena al atacante que solo controla la cookie: sin la clave no puede forjar un HMAC válido, y un `pickle` manipulado se rechaza antes de deserializar. Es una mejora real contra el vector original.

Pero no cierra la vulnerabilidad, y el propio módulo lo demuestra: <mark style="background: #FFB86CA6;">si el atacante consigue leer `SECRET_KEY` —por un LFI, un `.env` expuesto, un `File read` de PHPGGC, un repositorio filtrado—, calcula el HMAC él mismo y el ataque vuelve intacto, solo con un paso más</mark>. El `sink` sigue siendo `pickle.loads`. El HMAC traslada el problema de «no deserialices datos no confiables» a «mantén esta clave en secreto para siempre», que es una propiedad frágil: las claves se filtran.

> [!warning]+ Por qué una comparación de HMAC mal hecha añade un bug
> Comparar el HMAC con `==` (como en el código de ejemplo) introduce un **timing side-channel**: la comparación byte a byte retorna antes al primer byte distinto, lo que en teoría permite recuperar el HMAC esperado midiendo tiempos. La comparación correcta es en tiempo constante: `hmac.compare_digest(a, b)` en Python, `hash_equals()` en PHP. Un detalle, pero es el tipo de cosa que se señala en un informe.

# La defensa real: formato de datos, no de objetos

El nivel 2 elimina la deserialización de objetos sustituyéndola por un formato que **solo transporta datos**: JSON o XML sin tipos. El módulo migra `HTBank` a JSON:

```php
// Exportar: array simple, no objeto
$userSettings = array("name" => $user->name, "email" => $user->email, ...);
$exportedSettings = base64_encode(json_encode($userSettings));

// Importar: json_decode devuelve stdClass, NO instancia clases del proyecto
$userSettings = json_decode(base64_decode($request['settings']));
$user->name = $userSettings->name;
```

<mark style="background: #8000E1A6;">`json_decode` nunca instancia una clase del proyecto ni dispara un `magic method`, así que no hay `sink`</mark>. El mismo principio en Python es usar `json` en vez de `pickle`, `yaml.safe_load` en vez de `yaml.load`. El coste es que hay que reconstruir la lógica manualmente a partir de los datos, pero ese es precisamente el punto: la aplicación decide qué clases crea, no el atacante.

> [!important]+ Migrar a JSON no arregla los otros bugs
> El módulo deja claro un matiz que se pasa por alto: `HTBank` tenía, además de la deserialización, un XSS (`{!! !!}` en Blade), una inyección de comandos (`shell_exec` en `__wakeup`) y una subida de ficheros sin validar. Cambiar a JSON cierra el `sink` de deserialización pero **no toca los otros tres**. Cada uno se arregla aparte: `{{ }}` en vez de `{!! !!}`, funciones nativas (`fwrite`) en vez de `shell_exec`, validación `image` en la subida, y actualizar a PHP ≥ 8.0 para el PHAR. La deserialización era el vehículo, no la única herida.

# Allowlist de clases, cuando hay que deserializar

Si deserializar objetos es inevitable (interoperar con un sistema que lo exige), el nivel 3 restringe qué clases se admiten:

- **PHP**: `unserialize($data, ['allowed_classes' => ['MiClase']])` — solo esas clases; `false` degrada todo a `__PHP_Incomplete_Class`.
- **Python**: subclase de `pickle.Unpickler` que sobreescribe `find_class` para rechazar todo lo que no esté en una allowlist. `find_class` es el único punto de control.
- **Java**: `ObjectInputFilter` / `jdk.serialFilter` (JEP 290) para filtrar por clase y por profundidad.

<mark style="background: #FFB8EBA6;">La allowlist es frágil porque un solo `gadget` útil dentro de ella reabre el camino</mark>: si `MiClase` o cualquier cosa que admita tiene un `magic method` encadenable, la restricción no sirve. Es defensa de última línea, no primera.

Lo que cada lenguaje da de serie para esto es muy desigual, y conviene tenerlo claro al auditar:

| Lenguaje | Control nativo de allowlist |
| --- | --- |
| PHP | `allowed_classes` en `unserialize()` / `$unserializeOptions` en `getMetadata()` |
| Java | `ObjectInputFilter` / `jdk.serialFilter` (JEP 290), filtro por contexto (JEP 415) |
| Python | subclasear `Unpickler.find_class`; audit hook del evento `pickle.find_class` (proceso entero) |
| .NET | `SerializationBinder` + `TypeNameHandling=None`; `BinaryFormatter` **eliminado en .NET 9** |
| **Ruby** | **Ninguno.** `Marshal.load` no tiene allowlist en stdlib; el `proc` llega tarde (post-deserialización) |

<mark style="background: #FF5582A6;">Ruby es el caso extremo: no hay defensa nativa, solo «no llames a `Marshal.load` con datos no confiables»</mark>. El `freeze:` no es seguridad y el `proc` recibe los objetos ya construidos. Que un lenguaje no ofrezca el control no significa que el problema sea menor, sino que la única salida es el nivel 2 (formato de datos puro).

# Defensa en profundidad

Lo que un sistema maduro combina, sabiendo que ninguna capa es suficiente sola:

1. **Arquitectura**: no deserializar entrada de usuario; formato de datos puro.
2. **Si no hay más remedio**: allowlist estricta de clases en el punto de deserialización.
3. **Integridad**: firmar/cifrar con clave bien gestionada (rotada, no en el repo), comparación en tiempo constante.
4. **Runtime**: filtros de deserialización (`jdk.serialFilter`), audit hooks de Python (el evento `pickle.find_class`), agentes RASP.
5. **Detección**: logging de excepciones de deserialización, alertas sobre procesos hijos del intérprete web (ver [[12 - Detección y evasión de deserialización]]).
6. **Mantenimiento**: versiones al día — PHP ≥ 8.0 (PHAR), PyYAML ≥ 6.0 (`yaml.load`), .NET moderno (sin `BinaryFormatter`).

> [!info]+ Fuentes
> Jerarquía de defensa y recomendaciones: [OWASP — Deserialization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html). `allowed_classes`: [php.net — unserialize](https://www.php.net/manual/en/function.unserialize.php). Comparación en tiempo constante: [hmac.compare_digest](https://docs.python.org/3/library/hmac.html#hmac.compare_digest). Caso de patching del módulo 169 de HTB Academy, con la jerarquía de defensa añadida y el timing del HMAC señalado.
