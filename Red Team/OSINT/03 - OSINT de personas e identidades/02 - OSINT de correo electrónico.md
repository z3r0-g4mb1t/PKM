---
tags:
  - OSINT/Personas
  - Pentesting/Enumeracion
Descripción: "El correo como identificador: descubrir dónde está registrado con holehe, la cuenta de Google con GHunt, permutación y verificación de correos corporativos"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Enumeración de usernames cross-platform]]"
Nota siguiente: "[[03 - OSINT de teléfono]]"
Area: "[[OSINT de personas.base|OSINT de personas]]"
---
---

El correo es el identificador más valioso de una persona: es a la vez su **login** en media Internet y la llave para pivotar a cuentas, brechas y a su cuenta de Google. Tres frentes: dónde está registrado, qué cuenta de Google/Microsoft tiene, y —para un objetivo corporativo— cómo construir y verificar su correo de empresa.

# Dónde está registrado: holehe

<mark style="background: #ADCCFFA6;">`holehe` comprueba si un correo está registrado en 120+ sitios</mark> (Twitter, Instagram, Spotify, Imgur…) **sin alertar al titular**. El truco es elegante: para cada sitio usa el flujo de **recuperación de contraseña o de registro** como oráculo — si el sitio responde "ya existe una cuenta con ese correo", la cuenta existe.

```shell-session
$ holehe test@gmail.com
```

La salida marca por sitio si la cuenta `exists`, y si hubo `rateLimit`. Interpretación: las cuentas confirmadas son pivotes (ese correo tiene Spotify → mira ese perfil; tiene un foro → busca ese alias con [[01 - Enumeración de usernames cross-platform|Maigret]]).

> [!warning]+ holehe golpea los endpoints desde tu IP
> Las comprobaciones salen de **tu** dirección. Sitios como Google, Instagram o Yahoo tienen *rate limit* frecuente; el propio README lo resume: *"¿Rate limit? Cambia tu IP"*. Ejecútalo desde la [[02 - Infraestructura de investigación - VM, red y navegador|VM con proxy/VPN]], no desde tu red. Y aunque "no alerta al titular", sí dejas rastro en cada sitio consultado.

# La cuenta de Google: GHunt

Si el correo es `@gmail.com` (o una Google Workspace), <mark style="background: #FFB86CA6;">`GHunt` extrae una cantidad sorprendente de datos de la cuenta de Google asociada</mark>: nombre y foto de perfil, el `Gaia ID` (identificador interno de Google que permite más pivotes), reseñas de Google Maps (que **geolocalizan** a la persona por los sitios que reseña), ficheros de Drive públicos y la visibilidad del calendario.

GHunt se autentica con **tus propias cookies de Google** (vía la extensión *Companion* para Firefox/Chrome, o pegándolas):

```shell-session
# 1. Autenticarse una vez (Companion, base64 o manual)
$ ghunt login
# 2. Investigar un correo → datos de la cuenta Google
$ ghunt email target@gmail.com --json user_data.json
# 3. Pivotar por el Gaia ID obtenido
$ ghunt gaia <gaia_id>
```

<mark style="background: #FF5582A6;">Las reseñas de Maps son el hallazgo más potente</mark>: una persona que reseña su gimnasio, su cafetería y su dentista te está dibujando su mapa de vida. Cruza con [[GEOINT e imágenes.base|GEOINT]].

> [!important]+ GHunt usa TUS cookies: cuenta desechable
> Como GHunt se autentica con cookies de Google, **usa una cuenta de Google desechable** ([[01 - Sock puppets - crear y mantener identidades de investigación|sock puppet]]), nunca la tuya real. Las cookies son tu sesión: si usas tu cuenta, vinculas la investigación contigo.

# Alternativa web: Epieos

**Epieos** es una herramienta alojada (sin instalar nada) que, a partir de un correo, recupera el nombre y foto de la cuenta de **Google y Microsoft** cuando existen, y comprueba Skype y otros servicios. Su plan gratuito es limitado; el de pago (Osinter, ~30 €/mes) amplía. Útil cuando no quieres montar GHunt o para una comprobación rápida.

# Correo corporativo: construir y verificar

Para un objetivo de empresa, a menudo **no tienes** el correo, tienes el nombre (de LinkedIn, vía [[03 - Enumeración de personal]]). El proceso:

1. **Descubre el patrón** de la organización (`nombre.apellido@`, `inicial+apellido@`…) con `hunter.io`, `phonebook.cz`, o un correo real ya conocido.
2. **Permuta** el patrón sobre los nombres reales para generar candidatos:

```shell-session
# Generar permutaciones de correo a partir de nombre y dominio
$ echo "John Doe" | ... # o herramientas como namemash.py / genosint
# john.doe@acme.com, jdoe@acme.com, j.doe@acme.com, doej@acme.com, ...
```

3. **Verifica** cuáles existen **sin enviar correo**: la enumeración vía `SMTP` (`RCPT TO`) o contra O365/Teams valida un correo sin alertar. El detalle operativo está en [[08 - SMTP|enumeración SMTP]] y en la nota de [[03 - Enumeración de personal|personal]].

<mark style="background: #8000E1A6;">El resultado es una lista de correos válidos</mark> — la munición para *password spraying* y *phishing*. De ahí, el pivote natural: buscar esos correos en [[Brechas y filtraciones.base|brechas y stealer logs]].

> [!info]+ Fuentes
> [holehe](https://github.com/megadose/holehe) (megadose, flujo de reset como oráculo). [GHunt](https://github.com/mxrch/GHunt) (mxrch, módulos email/gaia/drive/geolocate, auth por cookies vía Companion), verificado 2026-11-08. [Epieos](https://epieos.com). Descubrimiento de patrón de correo: hunter.io, phonebook.cz.
