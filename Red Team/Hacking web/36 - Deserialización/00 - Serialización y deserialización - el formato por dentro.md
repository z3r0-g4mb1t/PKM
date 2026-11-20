---
tags:
  - Web/Red-Team
  - Deserializacion
  - Introduccion
  - Tipo/Introduccion
Descripción: "Qué hace exactamente serialize() y qué hace unserialize(), con la gramática del formato de PHP descodificada byte a byte y el mangling de propiedades privadas"
Fecha de actualización: 2026-11-20
Nota previa: ""
Nota siguiente: "[[01 - Deserialización insegura - de dato a ejecución de código]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

<mark style="background: #ADCCFFA6;">Serializar es convertir un objeto que vive en memoria en una secuencia de bytes que se puede guardar o transmitir; deserializar es reconstruir ese objeto a partir de los bytes</mark>. La operación existe porque la memoria de un proceso no es portable: un objeto PHP es un conjunto de punteros, tablas hash y referencias a clases que solo tienen sentido dentro de ese intérprete, en esa ejecución. Para que el objeto sobreviva a un `redirect`, viaje en una cookie o se guarde en Redis hay que aplanarlo.

La asimetría entre las dos mitades es la raíz de todo el tema, y conviene fijarla antes de ver un solo `payload`:

- **Serializar es un volcado.** Se recorre la estructura y se escriben sus valores. No hay decisiones.
- **Deserializar es una reconstrucción.** El runtime tiene que *instanciar clases*, asignar propiedades y, en muchos lenguajes, ejecutar los hooks que la clase define para ese momento. <mark style="background: #8000E1A6;">Eso significa que la deserialización no es un parser de datos: es un intérprete que ejecuta código elegido por quien escribió los bytes</mark>.

Por eso el bug no se arregla validando la entrada. Cuando el dato llega al deserializador, el daño ya es inevitable: la decisión de qué código correr se tomó al leer los primeros bytes.

# Dónde aparecen datos serializados

No es un detalle de implementación exótico. Es infraestructura cotidiana:

| Uso | Ejemplo concreto |
| --- | --- |
| Sesiones | El `session handler` de PHP serializa `$_SESSION` en disco o en Redis |
| Cookies y tokens | Cookie de autenticación que lleva el objeto `Session` dentro |
| Caché | Objetos de dominio guardados en Memcached o Redis |
| Colas de trabajo | Jobs de Laravel, Celery o Sidekiq que viajan como objetos serializados |
| Import/export | Funciones de «exportar configuración» o «guardar partida» |
| RPC e IPC | Java RMI, .NET Remoting, protocolos binarios propietarios |
| Modelos de ML | Checkpoints de PyTorch y `joblib`, que son `pickle` por dentro (ver [[11 - Pickle y la deserialización insegura de modelos]]) |

<mark style="background: #FFB8EBA6;">Las funciones de importar/exportar y las de «guardar progreso» son el candidato más rentable en una auditoría web</mark>: casi siempre están construidas sobre el serializador nativo, casi nunca tienen firma ni validación, y por diseño aceptan un blob opaco que el usuario sube.

# El formato de PHP, descodificado

PHP es el caso ideal para aprender el concepto porque su formato es **texto legible**. El resto de lenguajes usan formatos binarios que exigen herramientas para leerse, pero la mecánica es la misma.

Serializando un array con tres tipos distintos:

```shell-session
$ php -a
php > echo serialize(array("HTB", 123, 7.77));
a:3:{i:0;s:3:"HTB";i:1;i:123;i:2;d:7.77;}
```

La gramática son pares `tipo:valor` separados por `;`:

| Marca | Tipo | Forma |
| --- | --- | --- |
| `N` | `null` | `N;` |
| `b` | booleano | `b:1;` |
| `i` | entero | `i:123;` |
| `d` | `double` | `d:7.77;` |
| `s` | cadena | `s:<longitud>:"<contenido>";` |
| `a` | array | `a:<nº elementos>:{<clave><valor>...}` |
| `O` | objeto | `O:<longitud>:"<Clase>":<nº props>:{<nombre><valor>...}` |
| `C` | objeto con `Serializable` | `C:<len>:"<Clase>":<len datos>:{<datos>}` |
| `E` | caso de `enum` (PHP 8.1+) | `E:<len>:"Clase:CASO";` |
| `R` / `r` | referencia | `R:<nº>;` |

Leído sobre el ejemplo anterior:

```php
a:3:{              // (a)rray de (3) elementos
  i:0;s:3:"HTB";   // clave (i)ndex 0 → (s)tring de longitud 3
  i:1;i:123;       // clave 1        → (i)nteger 123
  i:2;d:7.77;      // clave 2        → (d)ouble 7.77
}
```

<mark style="background: #FF5582A6;">El campo de longitud de las cadenas es el detalle operativo que más rompe `payloads` artesanales</mark>: si se edita `s:3:"HTB"` a mano para poner `s:3:"ADMIN"`, `unserialize()` devuelve `false` porque la longitud declarada no cuadra. Cualquier modificación obliga a recalcular. De ahí que se genere el `payload` con un script en el mismo lenguaje en vez de con un editor hexadecimal.

> [!warning]+ El contador de longitud cuenta **bytes**, no caracteres
> Con cadenas multibyte (UTF-8) `strlen()` y el número de caracteres no coinciden. Un `payload` con acentos o emoji calculado sobre caracteres genera una longitud corta y falla. Esto también se puede explotar en sentido contrario: cuando una aplicación serializa, después pasa la cadena por una función que cambia su longitud (`addslashes`, un filtro de WAF, un `htmlspecialchars`) y vuelve a deserializar, el desajuste permite **inyección de objetos por desplazamiento** — el patrón conocido como *PHP object injection via string truncation*.

## Objetos: el `mangling` que HTB no explica

Serializando un objeto con propiedades privadas aparece algo que a primera vista parece un error de la herramienta. Este es el objeto de ejemplo del módulo, una clase `App\Helpers\UserSettings` con cuatro propiedades **privadas**:

```php
O:24:"App\Helpers\UserSettings":4:{s:30:"App\Helpers\UserSettingsName";s:7:"pentest";...}
```

La lectura ingenua es que la propiedad se llama `App\Helpers\UserSettingsName`. No es así, y la prueba está en el propio número: `"App\Helpers\UserSettings"` son 24 bytes —de ahí el `O:24:`— y `"Name"` son 4; eso suma 28, no 30. Faltan dos bytes.

<mark style="background: #ADCCFFA6;">PHP codifica la visibilidad de las propiedades en el propio nombre serializado, intercalando bytes `NUL` (`\x00`) que no se imprimen</mark>:

| Visibilidad | Nombre serializado | Longitud |
| --- | --- | --- |
| `public` | `Name` | 4 |
| `protected` | `\0*\0Name` | 1 + 1 + 1 + 4 = 7 |
| `private` | `\0App\Helpers\UserSettings\0Name` | 1 + 24 + 1 + 4 = **30** |

Los 30 bytes cuadran exactamente. Lo mismo con el resto: `Email` → 1+24+1+5 = 31, `Password` → 34, `ProfilePic` → 36, que es lo que muestra la salida real.

Esto tiene dos consecuencias prácticas:

1. **Los `NUL` se pierden al copiar y pegar.** Un `payload` sacado de la consola del navegador o de un log pierde los `\x00` y deja de funcionar sin ningún mensaje útil. Hay que transportarlo en Base64 o en hexadecimal.
2. <mark style="background: #FFB86CA6;">El nombre privado lleva el nombre de la clase declarante dentro</mark>. Si se falsifica un objeto reutilizando el nombre de propiedad de otra clase, la asignación falla silenciosamente y la propiedad se queda a `null` — un fallo típico al construir cadenas de `gadgets` a mano sobre jerarquías con herencia.

# Otros lenguajes, el mismo problema

| Lenguaje | Mecanismo nativo | Formato | Legible |
| --- | --- | --- | --- |
| PHP | `serialize` / `unserialize` | Texto delimitado | Sí |
| Python | `pickle` | `bytecode` de una máquina de pila | No, pero se desensambla |
| Java | `ObjectOutputStream` / `readObject` | Binario con cabecera `AC ED 00 05` | No |
| .NET | `BinaryFormatter` (eliminado en versiones modernas) | Binario | No |
| Ruby | `Marshal.dump` / `Marshal.load` | Binario con cabecera `04 08` | No |
| Multiplataforma | YAML, XML con tipos, JSON «enriquecido» | Texto | Sí |

Que un formato sea legible **no lo hace más seguro**, y que sea binario no lo hace más difícil de explotar: `pickle` es ilegible a simple vista y a la vez el caso más trivial de convertir en ejecución de código, porque su formato *es* un lenguaje de programación (ver [[08 - Python - pickle y su máquina virtual]]). La propiedad que importa no es el formato sino **si el deserializador instancia clases arbitrarias**.

Esa es la línea que separa lo peligroso de lo inofensivo:

- `json_decode` en PHP devuelve `array` y `stdClass`. No instancia clases del proyecto. **No es un `sink`.**
- `unserialize` reconstruye cualquier clase cargada. **Es un `sink`.**
- YAML es texto plano… pero `yaml.load` con un `loader` inseguro (`Loader`/`UnsafeLoader`) construye objetos Python arbitrarios. **Es un `sink`**; `yaml.safe_load` no lo es (ver [[11 - Más allá de pickle - JSONPickle, PyYAML y ruamel]]).

Para el detalle de cómo se representan y parsean formatos binarios estructurados en general —campos de longitud, `varints`, TLV— la referencia del vault es [[03 - Formatos binarios estructurados]]; y para los formatos de intercambio desde la óptica del desarrollo, [[12 - JSON, XML y datos estructurados]].

> [!info]+ Fuentes
> Gramática y comportamiento de `serialize()`/`unserialize()`: [manual de PHP](https://www.php.net/manual/en/function.serialize.php). El `mangling` de propiedades privadas y protegidas está documentado en la página de [`serialize`](https://www.php.net/manual/en/function.serialize.php) y verificado aquí recalculando las longitudes de la salida del módulo 169 de HTB Academy.
