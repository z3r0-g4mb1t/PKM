---
tags:
  - OSINT/Arsenal
  - Tipo/Arsenal
Descripción: "Qué herramientas OSINT están muertas, adquiridas o cambiadas en 2026 y por qué el ecosistema es un objetivo móvil que hay que verificar siempre"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - Arsenal OSINT por dimensión]]"
Nota siguiente:
Area: "[[Arsenal OSINT.base|Arsenal OSINT]]"
---
---

El tooling OSINT envejece rápido: herramientas que mueren, repos que se archivan, plataformas que cierran sus APIs, empresas que compran proyectos. <mark style="background: #FF5582A6;">Una guía de hace dos años está medio obsoleta</mark>. Este es el estado a **noviembre de 2026** de lo que esta área usa — y, más importante, el hábito de verificarlo.

# Lo que ha muerto o cambiado

| Herramienta / recurso | Estado 2026 | Qué usar en su lugar |
| - | - | - |
| **`pagodo`** | <mark style="background: #FF5582A6;">Archivado</mark> (Google rompió `yagooglesearch`) | [[03 - La Google Hacking Database y la automatización\|Custom Search JSON API]] |
| **`recon-ng`** | Sin commits desde finales de 2024 | OWASP Amass / [[Tools.base\|ProjectDiscovery]] para infra |
| **Telepathy (OSS)** | Sin mantenimiento desde 2024 | [[04 - Telegram y Discord - las plataformas abiertas\|Telethon]] (activo) |
| **`cache:` de Google** | Retirado en 2024 (`link:`/`info:` también) | [[13 - Web Archives\|Wayback]] |
| **Nitter** | Prácticamente muerto | Operadores de X logueado / twscrape |
| **Pushshift (Reddit)** | Cortado; Old Reddit con muro (jun-2026) | Arctic Shift (parcial), API en vivo |
| **BGPView** | Caído (26-nov-2025, *sin confirmar*) | bgp.he.net, bgp.tools, asnmap |
| **Genesis Market** | Incautado (2023) | — (contexto histórico de stealer logs) |

# Lo que ha cambiado de manos o de forma

- <mark style="background: #FFB86CA6;">**SpiderFoot** → adquirido por **Intel 471** (nov-2022)</mark>. El OSS sigue en GitHub (MIT); el comercial HX se integró en TITAN. Las contribuciones al OSS pueden ralentizarse.
- <mark style="background: #FFB86CA6;">**Censys** → migró a **CenQL**</mark> (rompió el `CSL` antiguo y mucho tooling); el tier gratuito quedó para consultas por web, no por script.
- <mark style="background: #FFB86CA6;">**X/Twitter API** → de pago por uso desde feb-2026</mark> (~0,005 $/post). El acceso masivo gratuito desapareció.
- **`gitleaks`** → *feature complete* (solo parches de seguridad); comandos `git`/`dir` (el viejo `detect` deprecado v8.19); el mantenedor se mueve a Betterleaks.

# Lo más vivo (2026)

- **theHarvester**: la más sana del grupo clásico (release en jun-2026, commits frecuentes).
- **Telethon**: activa (v1.45.0, sep-2026) — la base para Telegram.
- **exiftool**: rama 13.x activa (13.48 en ene-2026).
- **ProjectDiscovery** (subfinder/dnsx/httpx/nuclei/uncover): mantenimiento intenso; el estándar de pipeline.
- **La ola de 2026**: IA de geolocalización ([[02 - Búsqueda inversa de imágenes y geolocalización por IA\|GeoSpy, GeoSeer]]) y agentes OSINT asistidos por IA — nuevos, potentes y de **precisión no verificada**: trátalos como hipótesis.

# La tendencia de fondo

Dos corrientes definen el ecosistema OSINT de 2026:

1. <mark style="background: #8000E1A6;">**Cierre de las plataformas**</mark>: X, Reddit, Meta y los motores de dispositivos han cerrado o monetizado sus APIs para frenar el scraping y el entrenamiento de IA. El OSINT gratuito y masivo se ha encarecido; la recolección vuelve a ser más manual y dirigida.
2. <mark style="background: #8000E1A6;">**Entrada de la IA**</mark>: geolocalización por píxeles, correlación y resumen con LLMs, agentes autónomos. Acelera, pero **alucina** — no sustituye la verificación.

# El hábito, no la lista

<mark style="background: #FF5582A6;">Esta tabla caducará</mark>. Lo que no caduca es el método:

- **Verifica el repo antes de confiar**: último commit, release, issues abiertas. Un repo sin tocar en dos años es una señal.
- **Desconfía de un nombre de repo huérfano**: un proyecto abandonado es blanco de suplantación (igual que en el [[03 - Cómo no quemar la operación - fugas y errores comunes|OPSEC]], un `.onion` o un repo con el nombre correcto puede ser un clon).
- **Prefiere lo mantenido** y las fuentes primarias; cuando una herramienta muere, casi siempre hay un sucesor vivo.
- **Contrasta el dato, no solo la herramienta**: una tool viva puede dar un resultado desfasado; corrobora ([[05 - Verificación, fiabilidad de fuentes y sesgos|verificación]]).

Con esto se cierra el área: de la [[00 - Qué es OSINT y por qué da ventaja|disciplina y el método]] a las dimensiones, el OPSEC y el arsenal. El OSINT bien hecho es la ventaja que la mayoría de los operadores junior no explotan — y ahora está documentada de punta a punta.

> [!info]+ Fuentes
> Estados verificados contra los repositorios y comunicados oficiales (2026-11-08): [pagodo](https://github.com/opsdisk/pagodo) (archivado), [SpiderFoot/Intel 471](https://www.thomabravo.com/press-releases/intel-471-acquires-spiderfoot), [gitleaks](https://github.com/gitleaks/gitleaks), [Telethon](https://github.com/LonamiWebs/Telethon). Cambios de plataforma (Censys CenQL, X API, Reddit): guías 2026 citadas en las notas respectivas. Claims sin confirmar señalados como tales (BGPView, fechas de osintbench).
