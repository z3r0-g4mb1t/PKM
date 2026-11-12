---
tags:
  - OSINT/SOCMINT
  - Pentesting/Enumeracion
Descripción: "LinkedIn como fuente nº1 del red teamer: enumerar empleados con CrossLinked, leer el tech stack y evitar el anti-scraping que quema cuentas"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - SOCMINT en 2026 - el panorama tras el cierre de las APIs]]"
Nota siguiente: "[[02 - X-Twitter, Reddit y foros abiertos]]"
Area: "[[SOCMINT.base|SOCMINT]]"
---
---

LinkedIn es, con diferencia, la fuente SOCMINT más valiosa para un engagement corporativo. <mark style="background: #ADCCFFA6;">Es un organigrama público: empleados, cargos, tecnologías, proyectos y jerarquía, mantenido y actualizado por la propia plantilla</mark>. La nota de [[03 - Enumeración de personal|enumeración de personal]] ya introdujo su valor; aquí va la ejecución y cómo no quemarte.

# Qué extraer

De LinkedIn salen tres cosas que valen oro:

- **La lista de empleados** → nombres reales → usuarios y correos corporativos (munición de acceso).
- **El tech stack** → qué administra cada quién. <mark style="background: #FFB86CA6;">Un `SysAdmin` con "Active Directory, Exchange, CrowdStrike Falcon" en su perfil te dice la infraestructura **y las defensas**</mark>. Las ofertas de empleo son aún más explícitas (ver [[03 - Enumeración de personal#Las ofertas de empleo son un regalo|ofertas de empleo]]).
- **La jerarquía y los equipos** → a quién suplantar en un phishing (el CFO para un fraude del CEO, el de IT para un reset de contraseña).

# Enumerar empleados sin tocar LinkedIn: CrossLinked

El problema: LinkedIn **banea agresivamente** el scraping. La solución elegante es no tocar LinkedIn. <mark style="background: #ADCCFFA6;">`CrossLinked` enumera empleados raspando los **resultados de buscador** (`site:linkedin.com/in`), no LinkedIn directamente</mark> — sin credenciales, sin API, sin riesgo de cuenta:

```shell-session
# Genera correos nombre.apellido a partir del nombre de la empresa
$ python3 crosslinked.py -f '{first}.{last}@acme.com' "ACME Corporation"
# Otro formato (dominio\inicialapellido), con timeout y jitter
$ python3 crosslinked.py -f 'acme\{f}{last}' -t 15 -j 2 "ACME Corporation"
```

Produce `names.txt` (usuarios/correos en el formato pedido) y `names.csv` (datos crudos). Flags: `-f` el formato (`{first}`, `{last}`, `{f}`…), `--search google,bing` los motores, `-t`/`-j` timeout y jitter. <mark style="background: #FF5582A6;">La salida es directamente la wordlist de usuarios para *password spraying*</mark> ([[Ataques a contraseñas.base|password attacks]]).

# La vía con credenciales: linkedin2username

**linkedin2username** se **loguea** con una cuenta LinkedIn válida y recoge los empleados visibles, generando permutaciones de usuario. Es más completo (ve lo que ve una cuenta), pero <mark style="background: #FFB86CA6;">arriesga la cuenta</mark>: úsalo con un sock puppet quemable, nunca con tu LinkedIn real. CrossLinked es preferible por no tocar la plataforma.

# El anti-scraping de LinkedIn

LinkedIn tiene uno de los sistemas anti-scraping más agresivos de la web:

- <mark style="background: #FF5582A6;">La automatización con navegador se detecta rápido; raspar ~200 perfiles/día desde una cuenta puede acarrear suspensión permanente</mark> (cifra de proveedor, no verificada — asúmela conservadora).
- **"Quién ha visto tu perfil"**: tus visitas se notifican. Navega en **modo privado** de LinkedIn (ajuste de visibilidad) o con un sock puppet sin modo público.
- **Sales Navigator** da búsqueda avanzada (filtros por empresa, cargo, antigüedad) pero es de pago y igualmente sujeto a límites.

> [!warning]+ La regla de oro en LinkedIn
> Prefiere **no tocar LinkedIn** (CrossLinked vía buscador). Si necesitas navegar, hazlo con **sock puppet quemable**, visibilidad desactivada, despacio, y asumiendo que la cuenta puede caer. Nunca con tu perfil real ni con la cuenta corporativa del cliente.

# Encadenar

El flujo completo: `CrossLinked` da los nombres → el patrón de correo los convierte en emails → se verifican ([[02 - OSINT de correo electrónico|OSINT de correo]]) → se buscan en [[Brechas y filtraciones.base|brechas]] → spraying/phishing. LinkedIn es el primer eslabón de casi toda la cadena de acceso.

> [!info]+ Fuentes
> [CrossLinked](https://github.com/m8sec/CrossLinked) (m8sec; enumeración vía scraping de buscador, sin API ni login), verificado 2026-11-08. [linkedin2username](https://github.com/initstring/linkedin2username) (initstring; vía credenciales). Anti-scraping de LinkedIn: guías de scraping 2026 (claims de proveedor, tratar como conservadores).
