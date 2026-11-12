---
tags:
  - OSINT/SOCMINT
  - Pentesting/Enumeracion
Descripción: "Recolección en X con operadores de búsqueda avanzada, Reddit tras el muro de login y foros, y qué inteligencia saca cada uno"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - LinkedIn - la mina del red teamer]]"
Nota siguiente: "[[03 - Instagram, Facebook y TikTok]]"
Area: "[[SOCMINT.base|SOCMINT]]"
---
---

X, Reddit y los foros son donde la gente habla de más — sobre su trabajo, sus rutinas, sus frustraciones. Tras el cierre de las APIs ([[00 - SOCMINT en 2026 - el panorama tras el cierre de las APIs|panorama]]), la recolección es dirigida y manual, pero sigue siendo muy productiva.

# X/Twitter: los operadores de búsqueda avanzada

La API es de pago, pero <mark style="background: #ADCCFFA6;">la **búsqueda avanzada** en el navegador (con una cuenta sock puppet) sigue siendo la mejor vía gratuita</mark>. Los operadores se combinan en la barra de búsqueda:

| Operador | Qué filtra | Ejemplo |
| - | - | - |
| `from:` | Tuits de una cuenta | `from:jdoe` |
| `to:` | Respuestas a una cuenta | `to:acme_support` |
| `since:` / `until:` | Rango de fechas | `from:jdoe since:2026-01-01 until:2026-06-30` |
| `filter:` | Tipo de contenido | `filter:media`, `filter:links` |
| `geocode:` | Área geográfica (lat,long,radio) | `geocode:40.4,-3.7,5km acme` |
| `min_faves:` | Umbral de likes | `acme breach min_faves:50` |
| `conversation_id:` | Un hilo concreto | `conversation_id:1234...` |

<mark style="background: #FFB86CA6;">`from:empleado` + rango de fechas reconstruye toda su actividad</mark>: a qué hora tuitea (zona horaria y rutina), de qué se queja del trabajo (pretextos), a quién responde (red). `geocode:` encuentra tuits **desde** una ubicación — útil para eventos o sedes.

Para volumen, **twscrape** consulta los endpoints de X usando un *pool* de cuentas (necesitas cuentas, y es zona gris de `ToS`). Úsalo solo si la búsqueda manual no basta.

# Reddit: tras el muro de login

Reddit es una mina de intereses y opiniones, pero más cerrado desde 2026:

- <mark style="background: #FFB8EBA6;">Old Reddit exige login desde junio de 2026</mark> y el archivo Pushshift fue cortado. Navega con un sock puppet.
- **El historial de un usuario** (`reddit.com/user/<nombre>`) revela subreddits, intereses, husos horarios (horas de actividad) y, a veces, pistas de localización ("mi ayuntamiento de [ciudad]").
- **Archivos**: los sucesores de Pushshift (p. ej. Arctic Shift) conservan contenido borrado de forma **parcial**. El contenido eliminado por moderadores aún es accesible vía la API en vivo mientras exista.
- **El pivote de alias**: un username de Reddit suele reutilizarse en otras plataformas → [[01 - Enumeración de usernames cross-platform|enumeración de usernames]].

# Foros y comunidades de nicho

<mark style="background: #8000E1A6;">Donde más se abre un objetivo técnico es en los foros especializados</mark>: Stack Overflow, foros de administradores, comunidades de un producto. Un empleado preguntando "cómo configurar [producto X] con [versión Y]" te está revelando el stack interno y, a veces, configuraciones o errores concretos. Busca el dominio del objetivo, nombres de empleados y alias en estos foros (con `site:` desde Google). El post de `n3onf0x` del [[05 - De un dato a la persona - pivote completo|caso práctico]] salió precisamente de aquí.

> [!warning]+ Archiva lo que veas
> Tuits, posts y comentarios se borran. Si un hallazgo importa, **archívalo al momento** (captura con fecha, Archive.today, Wayback). Un tuit comprometedor que viste ayer puede no existir mañana — y sin evidencia, no es inteligencia ([[05 - Verificación, fiabilidad de fuentes y sesgos|verificación]]).

> [!info]+ Fuentes
> Operadores de X vigentes 2026: [usersearch.com, *X OSINT in 2026*](https://usersearch.com/resources/intel-hub/blog/x-twitter-osint-2026/). Estado de Reddit y archivos: [THINKPOL, *Reddit OSINT in 2026*](https://think-pol.com/blogs/reddit-osint-guide-2026); [Indicator, *Reddit OSINT: what still works*](https://indicator.media/p/reddit-osint-what-still-works-for-search-monitoring-and-archives). `twscrape`: [github.com/vladkens/twscrape](https://github.com/vladkens/twscrape).
