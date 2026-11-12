---
tags:
  - OSINT/SOCMINT
  - Pentesting/Enumeracion
Descripción: "Las plataformas cerradas de Meta y TikTok: qué sigue funcionando, Instaloader para Instagram público y la geolocalización desde publicaciones"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - X-Twitter, Reddit y foros abiertos]]"
Nota siguiente: "[[04 - Telegram y Discord - las plataformas abiertas]]"
Area: "[[SOCMINT.base|SOCMINT]]"
---
---

Instagram, Facebook y TikTok son las plataformas más cerradas al OSINT, pero siguen conteniendo lo más personal de un objetivo: fotos, ubicaciones, rutinas, círculo social. El enfoque es manual y quirúrgico.

# Instagram

Muy cerrado y con *rate limiting* agresivo. Lo que funciona:

- <mark style="background: #ADCCFFA6;">**Instaloader** descarga publicaciones, stories y metadatos de perfiles **públicos**</mark> sin necesidad de login (aunque con login compartimentado ve más):

```shell-session
# Descargar el perfil público: fotos, pies de foto, metadatos, geotags
$ instaloader profile <usuario>
# Solo metadatos (sin medios), para analizar rápido
$ instaloader --no-pictures --no-videos profile <usuario>
```

- **Geolocalización**: los pies de foto, los *tags* de ubicación y el fondo de las fotos sitúan a la persona. Un patrón de ubicaciones (gimnasio, cafetería, trabajo) dibuja su rutina → [[GEOINT e imágenes.base|GEOINT]].
- **La red**: seguidores y seguidos (si son públicos) mapean el círculo. Los comentarios revelan amistades y familia.

> [!warning]+ Instagram quema IPs rápido
> El scraping agresivo de Instagram dispara bloqueos y retos de verificación. Usa Instaloader **despacio**, desde la [[02 - Infraestructura de investigación - VM, red y navegador|VM con proxy]], y con una cuenta desechable si necesitas login. No lo lances sobre decenas de perfiles seguidos.

# Facebook

<mark style="background: #FFB8EBA6;">El *Graph Search* (que permitía consultas tipo "fotos de personas que trabajan en X en Y") murió en 2019</mark>. Lo que sigue dando juego:

- **La sección "Información/About"**: trabajo, estudios, ubicación, familia — si el perfil es público.
- **Lista de amigos y fotos públicas**: el círculo social y, en las fotos, ubicaciones y acompañantes.
- **El ID numérico de Facebook**: servicios como *lookup-id.com* o `find-my-fbid` resuelven el nombre de usuario a su **ID numérico inmutable** (el username cambia; el ID no), que sirve para construir URLs de búsqueda y pivotar.
- **Check-ins y eventos**: sitúan a la persona en el tiempo y el espacio.

Facebook pesa menos para objetivos jóvenes, pero sigue siendo clave para mandos intermedios y directivos de más edad.

# TikTok

- **Perfil y vídeos públicos**: la bio suele enlazar a otras redes (pivote), y los vídeos revelan **localización** (interiores de casa/oficina, rutas, matrículas, uniformes) y **rutina**.
- **Audio y fondo**: un vídeo "inocente" en la oficina puede exponer pantallas, pizarras, badges o la distribución física — inteligencia para un acceso físico.

# El hilo común: geolocalización

<mark style="background: #8000E1A6;">En estas tres plataformas, el valor no es el texto sino la **imagen**</mark>. Cada foto o vídeo es un candidato a geolocalización (fondo, señales, sombras, metadatos) y a extracción de metadatos. Ese trabajo se desarrolla en [[GEOINT e imágenes.base|GEOINT, imágenes y metadatos]]. Aquí basta con recolectarlas y archivarlas.

> [!danger]+ Observar, no interactuar
> Ver un perfil público es OSINT. Seguir, dar like, comentar o mandar un mensaje **no**: delata la investigación, puede alertar al objetivo y, en estas plataformas, tu interacción desde un sock puppet puede quemar la cuenta. Recolección pasiva salvo que el *pretexting* esté en el alcance.

> [!info]+ Fuentes
> [Instaloader](https://github.com/instaloader/instaloader) (descarga de perfiles públicos de Instagram). Graph Search de Facebook retirado en 2019 (histórico documentado). Resolución de ID de Facebook: lookup-id.com / `find-my-fbid`.
