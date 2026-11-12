---
tags:
  - OSINT/SOCMINT
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Qué murió y qué sigue vivo en el SOCMINT de 2026 tras el cierre de las APIs de X, Reddit y Meta, y cómo cambia el método"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - LinkedIn - la mina del red teamer]]"
Area: "[[SOCMINT.base|SOCMINT]]"
---
---

<mark style="background: #ADCCFFA6;">El `SOCMINT` (*Social Media Intelligence*) es el OSINT sobre redes sociales: perfiles, publicaciones, relaciones y los metadatos que arrastran</mark>. Es, para el red teamer, la fuente de pretextos de ingeniería social, de la estructura humana de la organización y de la rutina de un objetivo. Pero <mark style="background: #FF5582A6;">el SOCMINT de 2026 no se parece al de los tutoriales de hace tres años</mark>: la mayoría de las vías automáticas están muertas. Entender qué cambió evita perder el tiempo con herramientas que ya no funcionan.

# Lo que murió

El gran cierre de las plataformas (2023-2026) para frenar el *scraping* y entrenar a sus propias IAs:

- **X/Twitter**: se acabó el acceso gratuito masivo. <mark style="background: #FFB86CA6;">La API es de pago por uso desde febrero de 2026</mark> (~0,005 $ por post leído). Nitter y la mayoría de scrapers gratuitos, muertos.
- **Reddit**: el archivo **Pushshift** —la base de casi todas las herramientas de Reddit— fue cortado, y desde el 30 de junio de 2026 Old Reddit exige login. Muchas herramientas quedaron rotas o parciales.
- **Meta** (Facebook/Instagram): el *Graph Search* de Facebook murió en 2019; Instagram está muy cerrado. El scraping tras login es, además, [[04 - Marco legal y ético del OSINT|legalmente arriesgado]].

<mark style="background: #8000E1A6;">Moraleja: si una guía te dice "usa esta herramienta para volcar todo Twitter", es vieja</mark>. Comprueba siempre la fecha y el estado.

# Lo que sigue vivo

El método se desplaza del *scraping* masivo a la **recolección dirigida y manual** con disciplina:

- **Operadores de búsqueda logueado**: en X, `from:`, `since:`, `until:`, `conversation_id:` siguen funcionando en el navegador con una cuenta (sock puppet). Es la vía principal hoy.
- **Buscar desde fuera**: Google con `site:` sobre la plataforma (`site:t.me`, `site:linkedin.com/in`) esquiva muchos muros.
- **Plataformas abiertas**: Telegram, Mastodon, Bluesky y Discord siguen siendo relativamente accesibles — Telegram es hoy la más rica para OSINT ([[04 - Telegram y Discord - las plataformas abiertas|ver]]).
- **Archivos**: cuando el contenido se borra o se cierra, los archivos (Wayback, Archive.today, índices de terceros) lo conservan.
- **APIs de pago y datos de panel**: para volumen, quedan las APIs oficiales de pago y los proveedores comerciales — fuera del presupuesto de la mayoría.

# El método 2026

1. **Enumerar** la presencia social del objetivo (persona o empresa) — qué plataformas, qué cuentas.
2. **Recolectar dirigido** con operadores y cuentas desechables, no scrapers masivos.
3. **Archivar** al momento (el contenido es volátil — ver [[05 - Verificación, fiabilidad de fuentes y sesgos|volatilidad]]).
4. **Analizar**: relaciones, actividad temporal, correlación entre plataformas ([[05 - Análisis de red, actividad y correlación|análisis]]).

> [!warning]+ Marco legal del SOCMINT
> La práctica defendible: contenido **público**, capturado **sin bypass de autenticación** y **sin acceder a cuentas** ajenas. El scraping tras un login (aceptando el `ToS`) es donde se pierde legalmente (ver [[04 - Marco legal y ético del OSINT|hiQ / Meta v. Bright Data]]). En la UE, base jurídica de interés legítimo y minimización. Y OPSEC: toda interacción social va desde un [[01 - Sock puppets - crear y mantener identidades de investigación|sock puppet]], nunca tu cuenta.

Empezamos por la plataforma que más ventaja da en un engagement corporativo: **LinkedIn**.

> [!info]+ Fuentes
> Estado de X/Reddit 2026: [usersearch.com, *X (Twitter) OSINT in 2026*](https://usersearch.com/resources/intel-hub/blog/x-twitter-osint-2026/); [THINKPOL, *Reddit OSINT in 2026*](https://think-pol.com/blogs/reddit-osint-guide-2026); [Reddit API controversy (Wikipedia)](https://en.wikipedia.org/wiki/Reddit_API_controversy). Reconstrucción del SOCMINT: [Techicy, *How SOCMINT Is Being Rebuilt in 2026*](https://www.techicy.com/signal-in-the-noise-how-socmint-is-being-rebuilt-in-2026.html).
