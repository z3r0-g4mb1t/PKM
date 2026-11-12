---
tags:
  - OSINT/Dorking
  - Pentesting/Enumeracion
Descripción: "Dorking en Bing, DuckDuckGo y Yandex, GitHub dorking para secretos con trufflehog y gitleaks, y búsqueda en código fuente de la web"
Fecha de actualización: 2026-11-08
Nota previa: "[[03 - La Google Hacking Database y la automatización]]"
Nota siguiente: "[[05 - Defensa y detección]]"
Area: "[[Google Dorking.base|Google Dorking]]"
---
---

Google no es el único índice, y no es el mejor para todo. <mark style="background: #8000E1A6;">Cada motor indexa cosas distintas: un dork que no da nada en Google puede acertar en otro</mark>. Y para código y secretos, los buscadores generalistas se quedan cortos frente a los especializados.

# Otros buscadores web

| Motor | Operadores propios útiles | Para qué brilla |
| - | - | - |
| **Bing** | `site:`, `filetype:`, `ip:`, `contains:`, `inbody:` | `ip:1.2.3.4` lista sitios en una **IP** (Google no lo tiene) |
| **DuckDuckGo** | `site:`, `filetype:`, *!bangs* | Privacidad; `!g`, `!shodan` redirigen a otros motores |
| **Yandex** | `site:`, `mime:`, `domain:`, `url:` | Mejor índice de imágenes y contenido no anglosajón |

<mark style="background: #FFB86CA6;">El operador `ip:` de Bing es el más infravalorado</mark>: `ip:203.0.113.10` te da todos los sitios que Bing ha indexado en esa IP — *virtual hosts* que comparten servidor, un pivote de infraestructura que Google no ofrece. Yandex, por su parte, es la mejor opción para [[GEOINT e imágenes.base|búsqueda inversa de imágenes y rostros]].

La táctica: **el mismo dork, varios motores**. Construye el dork con `site:` + el eje que busques y pásalo por Google, Bing y Yandex; los tres índices se complementan.

# GitHub dorking: secretos en el código

Los secretos no suelen estar en el dominio del objetivo, sino en **el código de sus desarrolladores**. Un `.env` o una clave de API comiteada por error en un repo público es uno de los hallazgos más rentables del OSINT moderno.

## Búsqueda manual en GitHub

La búsqueda de código de GitHub (renovada en 2023) admite filtros tipo dork:

```text
# Secretos asociados al dominio del objetivo
"acme.com" password
"acme.com" AWS_SECRET_ACCESS_KEY
# Acotar a la organización o a tipos de fichero
org:acme path:*.env
org:acme "api_key"
org:acme filename:config language:yaml
```

Busca el **dominio corporativo**, el nombre de la org, nombres de proyectos internos (vistos en [[03 - Enumeración de personal|OSINT de personal]]) junto a `password`, `secret`, `token`, `api_key`, `BEGIN RSA PRIVATE KEY`. Revisa también los repos **personales** de los empleados, no solo los de la org.

## Automatizar con trufflehog y gitleaks

Buscar a mano no escala; dos herramientas lo automatizan y, además, **verifican** si el secreto es válido:

- <mark style="background: #ADCCFFA6;">**trufflehog**</mark> — escanea repos, organizaciones enteras, y muchas otras fuentes, buscando secretos y comprobando si siguen **vivos**:

```shell-session
# Escanear toda una organización de GitHub, solo secretos verificados (vivos)
$ trufflehog github --org=acme --only-verified
# Escanear un repo concreto (incluye todo su historial de commits)
$ trufflehog git https://github.com/acme/webapp
```

- <mark style="background: #ADCCFFA6;">**gitleaks**</mark> — detecta secretos en un repo (incluido el historial) o en un directorio:

```shell-session
# Escanear el historial completo de un repo git
$ gitleaks git -v path/al/repo
# Escanear un directorio o fichero (código ya clonado, sin git)
$ gitleaks dir -v path/al/codigo
```

> [!warning]+ gitleaks: comandos nuevos
> Desde la v8.19 los comandos son `gitleaks git` (repos) y `gitleaks dir` (directorios/ficheros). El antiguo `gitleaks detect` quedó **deprecado** (sigue funcionando, oculto). gitleaks está *feature complete*: solo recibirá parches de seguridad.

<mark style="background: #FF5582A6;">La diferencia clave de trufflehog: la verificación</mark>. No te dice solo "esto parece una clave AWS", sino "esta clave AWS **funciona ahora mismo**" — separa el ruido del hallazgo real.

# Búsqueda en el código fuente de la web

Para encontrar **quién** usa una tecnología, un snippet o un identificador (un ID de Google Analytics, un dominio, una librería vulnerable):

- **publicwww** — busca en el código fuente (HTML/JS/CSS) de millones de webs. `"UA-12345678"` encuentra todos los sitios que comparten ese ID de Analytics (pivote entre dominios del mismo dueño).
- **grep.app** — busca texto literal en millones de repos públicos de GitHub, muy rápido; ideal para rastrear un endpoint o una cadena concreta.
- **SearchCode** — busca en código de GitHub, GitLab, Bitbucket y más.

# Motores de dispositivos (el pivote a infraestructura)

Google indexa **páginas**; para **dispositivos y servicios** (banners, certificados, paneles IoT) el motor es otro: `Shodan`, `Censys`, `FOFA`, `ZoomEye`. Tienen su propio lenguaje de *dorks* (`http.title:`, `ssl.cert.subject.cn:`, `port:`). Como es una dimensión propia, se desarrolla en [[OSINT de infraestructura.base|OSINT de infraestructura]] — pero la idea mental es la misma: dorkear un índice especializado.

> [!info]+ Fuentes
> `ip:` de Bing y operadores por motor: documentación de Bing/Yandex. `trufflehog`: [github.com/trufflesecurity/trufflehog](https://github.com/trufflesecurity/trufflehog). `gitleaks` (comandos `git`/`dir`, `detect` deprecado v8.19): [github.com/gitleaks/gitleaks](https://github.com/gitleaks/gitleaks), verificado 2026-11-08. Búsqueda en código: publicwww, grep.app, SearchCode.
