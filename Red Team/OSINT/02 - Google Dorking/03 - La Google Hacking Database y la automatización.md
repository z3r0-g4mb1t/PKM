---
tags:
  - OSINT/Dorking
  - Pentesting/Enumeracion
Descripción: "Qué es la GHDB y cómo automatizar el dorking: pagodo (archivado), la Custom Search JSON API como vía fiable y por qué Google bloquea el scraping"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - Dorks por objetivo]]"
Nota siguiente: "[[04 - Dorking más allá de Google]]"
Area: "[[Google Dorking.base|Google Dorking]]"
---
---

Lanzar dorks a mano no escala. Esta nota cubre el **catálogo** del que tirar (la GHDB) y cómo **automatizar**, con el matiz que todo el mundo descubre tarde: Google combate activamente la automatización.

# La Google Hacking Database (GHDB)

<mark style="background: #ADCCFFA6;">La GHDB es un catálogo público de dorks mantenido por OffSec dentro de Exploit-DB</mark>, organizado en **14 categorías** (ficheros con contraseñas, dispositivos conectados, mensajes de error, ficheros vulnerables, directorios sensibles, portales de login, etc.). Es el punto de partida: en vez de inventar dorks, parte de los que ya se sabe que funcionan y adáptalos a tu objetivo.

Uso manual: navega la categoría que te interese en [exploit-db.com/google-hacking-database](https://www.exploit-db.com/google-hacking-database), coge el dork y **añádele `site:acme.com`** para acotarlo a tu objetivo. Un dork genérico (`intitle:"index of" "config.php"`) busca en toda la web; con `site:` lo diriges.

<mark style="background: #FFB8EBA6;">OffSec publica la GHDB también como XML y CSV descargables</mark>, lo que permite parsearla y alimentar tus propias herramientas.

# Automatizar: pagodo (y por qué está archivado)

La herramienta histórica de referencia es **`pagodo`** (*passive Google dork*). Dos scripts:

- `ghdb_scraper.py` — descarga los dorks actuales de la GHDB a ficheros locales.
- `pagodo.py` — ejecuta esos dorks contra un dominio, con retardos aleatorios y proxies.

```shell-session
# 1. Clonar e instalar
$ git clone https://github.com/opsdisk/pagodo.git && cd pagodo
$ python3 -m venv .venv && source .venv/bin/activate
$ pip install -r requirements.txt

# 2. Descargar los dorks de la GHDB (texto + JSON + por categoría)
$ python ghdb_scraper.py -s -j -i

# 3. Lanzarlos contra un dominio, con retardos y salida a fichero
$ python pagodo.py -d acme.com -g dorks/all_google_dorks.txt -i 30 -x 60 -o resultados.json -s urls.txt
```

Flags clave: `-d` acota con `site:`, `-i`/`-x` el retardo mínimo/máximo entre consultas (clave para no ser bloqueado), `-m` resultados por dork (máx. 100 por petición), `-p` lista de proxies HTTP/SOCKS5 round-robin, `-o`/`-s` salida JSON/texto.

> [!warning]+ pagodo está archivado (2024)
> El repo está **archivado y sin mantenimiento**: Google cambió cómo devuelve los resultados y rompió la librería subyacente (`yagooglesearch`). Puede fallar o dar resultados vacíos. Lo documento porque sigue siendo la referencia conceptual y el `ghdb_scraper.py` para **bajar los dorks** sigue sirviendo; pero para **ejecutar** dorks de forma fiable en 2026, la vía es la API.

# La vía fiable: Custom Search JSON API

<mark style="background: #FF5582A6;">Google **bloquea el scraping**: tras ~10-20 consultas rápidas desde una IP aparecen captchas y `HTTP 429`</mark>, y el dorking masivo "a pelo" simplemente no funciona (el propio pagodo avisa de que 7300 consultas seguidas no van a ninguna parte). La solución oficial y estable es la **Programmable Search Engine + Custom Search JSON API**:

1. Crea un *Programmable Search Engine* (configúralo para "buscar en toda la web") y obtén su ID (`cx`).
2. Genera una API key en Google Cloud.
3. Consulta el endpoint — sin captchas, con cuota:

```shell-session
$ curl "https://www.googleapis.com/customsearch/v1?key=API_KEY&cx=SEARCH_ENGINE_ID&q=site:acme.com+inurl:admin"
```

<mark style="background: #FFB8EBA6;">Cuota: 100 consultas/día gratis; más allá, de pago (≈5 $/1000 hasta 10.000/día)</mark>. Es la forma legítima y fiable de automatizar dorks a escala moderada sin que te bloqueen ni violar los `ToS`.

# Otras herramientas vivas

- **GooFuzz** — hace *dorking* con `site:` para enumerar ficheros, directorios y correos **sin tocar al objetivo** (todo vía Google). Útil cuando quieres recon pasivo de rutas.
- **theHarvester** y **recon-ng** — integran consultas a buscadores dentro de un flujo OSINT más amplio (ver [[Frameworks OSINT.base|frameworks]]).
- **SERP APIs de pago** (SerpAPI y similares) — resuelven el captcha por ti a cambio de dinero; útiles para volumen.

> [!success]+ Flujo recomendado 2026
> Baja los dorks de la GHDB (con `ghdb_scraper.py` o el CSV de OffSec) → fíltralos por relevancia para tu objetivo → ejecútalos vía **Custom Search API** (fiable) o, para pocos y críticos, **a mano**. Olvida el scraping directo masivo: ni funciona ni es sigiloso.

> [!info]+ Fuentes
> GHDB: [Exploit-DB / OffSec](https://www.exploit-db.com/google-hacking-database) (14 categorías, export XML/CSV). `pagodo` y su estado archivado: [github.com/opsdisk/pagodo](https://github.com/opsdisk/pagodo) (README, verificado 2026-11-08). Bloqueo y captchas: [CybelAngel](https://cybelangel.com/blog/google-dorks-for-osint-security-teams-guide/). API: Google *Programmable Search Engine* / *Custom Search JSON API* (documentación oficial).
