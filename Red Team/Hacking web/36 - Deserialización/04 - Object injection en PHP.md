---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Falsificar propiedades de un objeto PHP deserializado para saltarse validaciones de registro y encadenar hacia XSS almacenado o reflejado"
Fecha de actualización: 2026-11-20
Nota previa: "[[03 - PHP - del sink al laboratorio local]]"
Nota siguiente: "[[05 - Magic methods de PHP y la ruta a RCE]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

El primer grado del ataque no toca el sistema operativo: manipula los valores del objeto que el servidor reconstruye. Es menos vistoso que una RCE, pero en una aplicación real suele ser el camino que de verdad rompe la lógica de negocio, porque <mark style="background: #FFB86CA6;">permite inyectar valores que los controles de la aplicación nunca habrían aceptado por la vía normal</mark>.

# Saltarse una validación de entrada

El escenario del módulo: `HTBank` prohíbe registrar cuentas con dominio `@htbank.com` porque es el de los administradores. La validación está en el formulario de registro y en el de cambio de email. Pero la función de importar ajustes deserializa un objeto `UserSettings` y copia sus campos al usuario **sin revalidar**:

```php
$userSettings = unserialize(base64_decode($request['settings']));
$user->email = $userSettings->getEmail();
$user->save();
```

<mark style="background: #8000E1A6;">La validación vive en una puerta de entrada y el objeto deserializado entra por otra</mark>. Como el atacante genera el objeto en su propia máquina, pone en `Email` el valor que quiera, lo serializa, lo importa y el servidor lo guarda:

```php
echo base64_encode(serialize(
    new \App\Helpers\UserSettings('pentest', 'attacker@htbank.com', '$2y$10$...', 'default.jpg')
));
```

Importado el `payload`, el email queda como `attacker@htbank.com`. Es el patrón de **validación incompleta por canal alternativo**: la misma propiedad se valida en un flujo y se confía en otro. La lección de diseño —validar en el modelo, no en el formulario— está en [[13 - Defensa - formatos seguros y restricción de clases]].

Generalizado, `object injection` sirve para falsificar cualquier campo que el objeto transporte y la aplicación confíe: un rol (`user` → `admin`), un identificador de usuario (IDOR vía deserialización), un precio, un flag de «cuenta verificada», una ruta de fichero. El límite es qué propiedades lleva el objeto y qué hace la aplicación con ellas.

# Encadenar a XSS

El segundo salto del módulo muestra cómo `object injection` se compone con otras vulnerabilidades. Tras importar, la aplicación muestra un mensaje de confirmación con el nombre del usuario:

```php
Session::flash('ie-message', "Imported settings for '" . $userSettings->getName() . "'");
```

Y la plantilla Blade lo renderiza **sin escapar**:

```php
<p class="text-success">{!! Session::get('ie-message') !!}</p>
```

En Blade, `{{ $x }}` pasa el valor por `htmlspecialchars`; <mark style="background: #FF5582A6;">`{!! $x !!}` lo imprime crudo</mark>. Como el `Name` sale del objeto deserializado, es territorio del atacante. Poniendo un `payload` XSS en ese campo:

```php
echo base64_encode(serialize(
    new \App\Helpers\UserSettings('<script>alert(1)</script>', 'attacker@htbank.com', '$2y$10$...', 'default.jpg')
));
```

Al importar, el `<script>` se refleja sin escapar y ejecuta. Es [[00 - Introducción a XSS|XSS]] reflejado alcanzado a través de la deserialización — el objeto deserializado es el vehículo que transporta el `payload` hasta un `sink` de HTML.

> [!warning]+ Por qué esto es más que «un XSS normal»
> El `payload` no viaja en un parámetro visible de la URL ni en un campo de formulario; viaja **dentro de un blob serializado en Base64**. Un WAF que inspeccione parámetros buscando `<script>` no lo ve, porque está codificado. Y como la aplicación persiste el `Name` en el usuario, el mismo `payload` puede acabar siendo **XSS almacenado** si ese nombre se muestra en otra pantalla —un panel de administración, un listado de usuarios— sin escapar: el impacto escala de reflejado a almacenado sin cambiar el `payload`.

# El patrón reutilizable

Lo que hace a `object injection` valioso en una auditoría no es este caso concreto sino el molde:

1. La aplicación deserializa un objeto cuyos campos controla el atacante.
2. Alguno de esos campos se usa **después** en una operación sensible: una consulta, un render, una comprobación de autorización, una ruta de fichero.
3. El campo llega a esa operación **sin pasar por la validación** que sí se aplica en el flujo normal.

<mark style="background: #ADCCFFA6;">Cada propiedad del objeto es un parámetro de entrada adicional que la aplicación probablemente no considera entrada de usuario</mark>, y por eso no la valida. El trabajo del pentester es mapear, para cada propiedad, a qué `sink` llega: `getEmail()` a una validación de dominio, `getName()` a un render HTML, y —en la nota siguiente— un `getName()` que acaba dentro de un `shell_exec`, que es donde `object injection` se convierte en RCE.

> [!info]+ Fuentes
> Escape de variables en Blade: [documentación de Laravel Blade](https://laravel.com/docs/blade#displaying-unescaped-data) (la distinción `{{ }}` vs `{!! !!}`). PHP Object Injection como clase: [OWASP — PHP Object Injection](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection). Caso base del módulo 169 de HTB Academy, con el patrón de validación por canal alternativo generalizado.
