---
tags:
  - OSINT/Personas
  - Pentesting/Enumeracion
Descripción: "Del número a la persona: formato E.164 y operador con PhoneInfoga, fugas de las apps de mensajería, caller-ID colaborativo y pivote a brechas"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - OSINT de correo electrónico]]"
Nota siguiente: "[[04 - Buscadores de personas y registros públicos]]"
Area: "[[OSINT de personas.base|OSINT de personas]]"
---
---

El teléfono es un identificador fuerte —cambia menos que un correo y va atado a verificaciones— pero también el más resistente al OSINT puro: gran parte de la información útil está en bases colaborativas o en fugas, no indexada. El enfoque realista tiene cuatro capas.

# Capa 1: formato, país y operador (PhoneInfoga)

Lo primero es normalizar y caracterizar el número. <mark style="background: #ADCCFFA6;">`PhoneInfoga` identifica país, zona, operador y tipo de línea (fijo/móvil/VoIP) a partir de un número internacional</mark>, y después genera enlaces de búsqueda (motores y apps de mensajería) para rastrear su huella:

```shell-session
# Escanear un número en formato E.164 (+código de país)
$ phoneinfoga scan -n "+34600112233"
# Levantar la interfaz web local
$ phoneinfoga serve
```

> [!warning]+ Qué es y qué NO es PhoneInfoga
> <mark style="background: #FFB8EBA6;">Es un **formateador + generador de huella**, no una base de datos mágica</mark>. Te da los metadatos del número y te construye las búsquedas, pero si el número **no tiene huella online, devuelve poco**. El tipo de línea VoIP es un dato útil (números desechables), y el operador ayuda a geolocalizar y a pretextar. El trabajo de verdad lo haces tú con los enlaces que genera.

# Capa 2: fugas de las apps de mensajería

Aquí está el valor real. <mark style="background: #FFB86CA6;">Al añadir un número a tus contactos, WhatsApp, Telegram y Signal pueden revelar foto de perfil, nombre, estado y "última hora"</mark> — según la configuración de privacidad de la víctima. Un número confirmado en WhatsApp con foto es una identificación visual.

La técnica: desde una **cuenta sock puppet** de la app, importa el número y observa qué perfil aparece. Caveats operativos:

- Hazlo desde un número/cuenta **desechable**, nunca el tuyo.
- Puede dejar rastro: añadir a alguien puede hacerte aparecer en sus "contactos que te tienen" o sugerencias.
- Telegram es especialmente generoso si la víctima permite "quién puede encontrarme por número".

# Capa 3: caller-ID colaborativo

Apps como **Truecaller** (y similares) construyen una base de datos de "número → nombre" con las agendas de millones de usuarios. <mark style="background: #FFB8EBA6;">Si alguien guardó ese número con un nombre, puede aparecer</mark>. Es información colaborativa (y a menudo imprecisa o desactualizada), pero a veces da el nombre directamente. Úsala como pista `C`-grade, a corroborar.

# Capa 4: el número en brechas

Muchas filtraciones incluyen teléfonos. <mark style="background: #FF5582A6;">La fuga de Facebook de 2021 (533 M de registros) asoció números a nombres, ubicaciones y correos</mark>, y servicios como Dehashed permiten **buscar por número de teléfono** y recuperar el resto del perfil. Es, a menudo, la vía más directa de número → identidad completa. Se desarrolla en [[Brechas y filtraciones.base|brechas y filtraciones]].

# De número a persona

El flujo que encadena las cuatro capas:

```text
+34600112233
  │ PhoneInfoga → operador móvil español, no VoIP
  │ mensajería  → foto de perfil en WhatsApp (sock puppet)
  │ Truecaller  → "Juan (trabajo)"
  │ brechas     → juan.perez@gmail.com + ciudad (fuga de 2021)
  ▼
persona identificada → pivota a correo ([[02 - OSINT de correo electrónico]]) y social
```

> [!danger]+ No contactes al objetivo como técnica
> El OSINT de teléfono es **observacional**. Llamar o mandar un SMS al número "para ver qué pasa" ya no es OSINT: es interacción directa, delata tu investigación y puede constituir acoso o un pretexto no autorizado. Mantente en la recolección pasiva salvo que el *pretexting* esté explícitamente en el alcance.

> [!info]+ Fuentes
> [PhoneInfoga](https://github.com/sundowndev/phoneinfoga) (sundowndev; metadatos + generación de huella con recursos gratuitos). Fugas de mensajería y privacidad por número: documentación de privacidad de WhatsApp/Telegram. Búsqueda por teléfono en brechas: ver [[Brechas y filtraciones.base|brechas]].
