---
tags:
  - Web/Red-Team
  - Pentesting/Enumeracion
  - Recon
Descripción: "El search engine discovery (o OSINT con buscadores) usa los motores de búsqueda como herramientas para descubrir información sobre un objetivo: documentos sensibles, páginas de…"
Fecha de actualización: 2026-11-08
Nota previa: "[[11 - Spidering con Scrapy]]"
Nota siguiente: "[[13 - Web Archives]]"
Area: "[[Reconocimiento Web.base|Reconocimiento Web]]"
---
---

<mark style="background: #ADCCFFA6;">El `search engine discovery` (o `OSINT` con buscadores) usa los motores de búsqueda como herramientas para descubrir información sobre un objetivo</mark>: documentos sensibles, páginas de login ocultas, credenciales expuestas, datos de empleados. Aprovecha que los buscadores ya han indexado una porción enorme de la web por ti.

# Por qué importa

- <mark style="background: #FFB8EBA6;">**Open source y legal**: la información es de acceso público</mark>, así que recopilarla no toca al objetivo ni viola nada.
- **Amplitud**: los buscadores indexan una parte gigantesca de la web.
- **Coste cero y sin barrera técnica**: gratis y sin necesidad de herramientas especializadas.

Su límite: no todo está indexado, y parte de la información se oculta o protege deliberadamente.

# Operadores de búsqueda

Los operadores son el lenguaje de precisión del buscador. La sintaxis varía algo entre motores, pero el principio es común:

| Operador | Qué hace | Ejemplo |
| - | - | - |
| `site:` | Limita a un dominio | `site:example.com` |
| `inurl:` | Término en la URL | `inurl:login` |
| `filetype:` / `ext:` | Tipo de fichero | `filetype:pdf` |
| `intitle:` | Término en el título | `intitle:"confidential report"` |
| `intext:` | Término en el cuerpo | `intext:"password reset"` |
| `cache:` | Versión cacheada (⚠️ Google lo **retiró en 2024**; usa [[13 - Web Archives|Wayback]]) | `cache:example.com` |
| `allintext:` | Todos los términos en el cuerpo | `allintext:admin password reset` |
| `allinurl:` | Todos los términos en la URL | `allinurl:admin panel` |
| `AND` / `OR` / `NOT` | Combinan o excluyen términos | `site:example.com AND (inurl:admin OR inurl:login)` |
| `*` | Comodín (cualquier palabra) | `site:x.com filetype:pdf user* manual` |
| `..` | Rango numérico | `site:shop.com "price" 100..500` |
| `" "` | Frase exacta | `"information security policy"` |
| `-` | Excluye un término | `site:news.com -inurl:sports` |

# Google Dorking

<mark style="background: #ADCCFFA6;">`Google Dorking` (o `Google Hacking`) combina estos operadores para destapar información sensible, vulnerabilidades o contenido oculto</mark> —paneles de login, backups `.sql`, ficheros `.env`/`config.php`, listados de directorio, errores reveladores—. <mark style="background: #FF5582A6;">Un `.sql` de backup o un `.env` indexado son credenciales servidas en bandeja</mark>.

> [!important]+ Referencia completa: área OSINT › Google Dorking
> El dorking tiene desarrollo propio en el área de OSINT: [[00 - Qué es el Google Dorking|qué es]], la [[01 - Operadores de búsqueda de Google|referencia de operadores (2026, con los muertos marcados)]], las [[02 - Dorks por objetivo|recetas por objetivo]], la [[03 - La Google Hacking Database y la automatización|GHDB y automatización]] (y por qué Google bloquea el scraping), el [[04 - Dorking más allá de Google|dorking en Bing/Yandex/GitHub con trufflehog y gitleaks]] y la [[05 - Defensa y detección|defensa frente al dorking]]. Aquí basta con saber que existe y encaja en el recon pasivo.

Los buscadores indexan el presente. Para ver lo que el sitio **mostraba antes** —rutas borradas, parámetros antiguos, versiones vulnerables— se recurre a los archivos históricos: [[13 - Web Archives]].
