---
tags:
  - OSINT/Dorking
  - Pentesting/Enumeracion
Descripción: "Recetas de dorks listas por tipo de hallazgo: paneles, configs, backups, listados, errores, documentos y dispositivos, con qué destapa cada una"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Operadores de búsqueda de Google]]"
Nota siguiente: "[[03 - La Google Hacking Database y la automatización]]"
Area: "[[Google Dorking.base|Google Dorking]]"
---
---

El catálogo operativo. Cada bloque es un **objetivo de recon** con dorks listos y la explicación de **qué destapa** y **por qué importa**. Sustituye `acme.com` por tu objetivo (en alcance). La mayoría combina `site:` para acotar — sin él, buscas en toda la web.

# Paneles de login y administración

```text
site:acme.com (inurl:login OR inurl:admin OR inurl:signin OR inurl:portal OR inurl:dashboard)
site:acme.com intitle:"login" OR intitle:"admin" OR intitle:"control panel"
site:acme.com inurl:wp-admin          # WordPress
site:acme.com inurl:/phpmyadmin/      # phpMyAdmin expuesto
```

<mark style="background: #FFB86CA6;">Un panel de login es una superficie de ataque directa</mark>: fuerza bruta, credenciales por defecto, *default creds*, CVEs del software. Un phpMyAdmin o un wp-admin indexados son puntos de entrada de primer orden.

# Ficheros de configuración y secretos

```text
site:acme.com (ext:env OR ext:ini OR ext:conf OR ext:cnf OR ext:yml OR ext:yaml)
site:acme.com inurl:config.php
site:acme.com inurl:wp-config
site:acme.com (filetype:xml inurl:web.config)
site:acme.com intext:"DB_PASSWORD"    # busca credenciales en el cuerpo
```

<mark style="background: #FF5582A6;">Un `.env` o un `config.php` indexados son credenciales de base de datos, claves de API y tokens servidos en texto</mark>. Es, con diferencia, el hallazgo más crítico del dorking: acceso directo sin explotar nada.

# Backups y volcados de base de datos

```text
site:acme.com (ext:sql OR ext:bak OR ext:old OR ext:backup OR ext:swp)
site:acme.com (inurl:backup OR inurl:dump OR inurl:db)
site:acme.com intitle:"index of" (sql OR backup OR dump)
```

Un backup olvidado (`site.sql`, `db_backup.old`) puede contener **toda la base de datos**: usuarios, hashes, datos. Los `.swp`/`.old` son copias temporales de editores que filtran el código fuente.

# Listados de directorio (directory listing)

```text
site:acme.com intitle:"index of /"
site:acme.com intitle:"index of" (passwd OR password OR admin OR backup OR .git)
site:acme.com intitle:"index of" "parent directory"
```

<mark style="background: #ADCCFFA6;">El *directory listing* es un servidor mostrando el contenido de una carpeta sin `index.html`</mark>. Expone la estructura y a menudo ficheros que no deberían ser navegables. `intitle:"index of" .git` es especialmente goloso: un repositorio `.git` expuesto permite reconstruir el código fuente.

# Mensajes de error y fugas de información

```text
site:acme.com intext:"sql syntax near" OR intext:"syntax error has occurred"
site:acme.com intext:"Warning: mysql_connect()"
site:acme.com intext:"Fatal error" intext:"on line"
site:acme.com intitle:"phpinfo()" OR intext:"PHP Version"
```

<mark style="background: #FFB86CA6;">Un error indexado revela rutas absolutas, versiones, consultas SQL y la pila tecnológica</mark>. Un `phpinfo()` expuesto es un mapa completo de la configuración del servidor. Todo esto afina el ataque posterior.

# Documentos y sus metadatos

```text
site:acme.com (filetype:pdf OR filetype:xlsx OR filetype:docx OR filetype:pptx)
site:acme.com (filetype:pdf OR filetype:docx) (intext:"confidential" OR intext:"internal use only")
site:acme.com filetype:xls (intext:"email" OR intext:"password" OR intext:"ssn")
```

Los documentos públicos del objetivo contienen datos internos **y metadatos** (autor, software, rutas, a veces nombres de usuario de red). Descargar y extraer esos metadatos con `exiftool` da inteligencia de personas e infraestructura (ver [[GEOINT e imágenes.base|metadatos]]).

# Dispositivos y servicios expuestos

```text
intitle:"webcamXP" OR intitle:"Live View / - AXIS"     # cámaras IP
inurl:"/view.shtml" OR inurl:"ViewerFrame?Mode="        # cámaras
intitle:"HP LaserJet" OR intitle:"printer status"        # impresoras
intitle:"Grafana" inurl:/login                           # dashboards
```

Cámaras, impresoras y paneles IoT con interfaz web, a menudo con credenciales por defecto. Para **dispositivos** a escala, el motor adecuado no es Google sino Shodan/Censys (ver [[04 - Dorking más allá de Google]] y [[OSINT de infraestructura.base|infraestructura]]).

# Credenciales y claves en ficheros

```text
site:acme.com (ext:log intext:password)
site:acme.com filetype:txt (intext:"username" intext:"password")
site:acme.com (ext:json OR ext:yml) intext:"api_key"
```

Logs y ficheros de texto que filtran credenciales en claro. Para secretos en **repositorios de código** (que no están en el dominio del objetivo), el *dorking* se traslada a GitHub ([[04 - Dorking más allá de Google]]).

> [!warning]+ El falso positivo y el cebo
> No todo resultado es real ni seguro. Un `config.php` "con credenciales" puede ser un [[05 - Verificación, fiabilidad de fuentes y sesgos#Datos envenenados: cuando la fuente te tiende una trampa|honeytoken]] que alerta al defensor al usarlo. Corrobora, y recuerda: **encontrar no es explotar**. Abrir o usar un hallazgo fuera de alcance es [[04 - Marco legal y ético del OSINT|acceso ilícito]].

> [!success]+ Cómo priorizar los hallazgos
> Orden de criticidad típico: secretos/configs (`.env`, `config.php`) → backups de BBDD (`.sql`) → `.git` expuesto → paneles de admin → errores/`phpinfo` → documentos. Un solo `.env` suele valer más que cien documentos.

Lanzar cientos de estos dorks a mano no escala. La vía para automatizarlos —y por qué Google te lo pone difícil— está en la [[03 - La Google Hacking Database y la automatización|siguiente nota]].

> [!info]+ Fuentes
> Dorks por categoría: [Google Hacking Database (Exploit-DB)](https://www.exploit-db.com/google-hacking-database), 14 categorías mantenidas por OffSec. Recetas contrastadas con [CybelAngel, *Google Dorks for OSINT 2026*](https://cybelangel.com/blog/google-dorks-for-osint-security-teams-guide/).
