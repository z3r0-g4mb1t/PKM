---
tags:
  - OSINT/Infraestructura
  - Pentesting/Enumeracion
Descripción: "Encontrar los servicios expuestos del objetivo sin escanearlo: sintaxis y ejemplos de Shodan, Censys (CenQL) y FOFA, y la realidad de sus tiers gratuitos"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - OSINT de infraestructura - el puente al recon técnico]]"
Nota siguiente: "[[02 - ASN, BGP y mapeo de rangos IP]]"
Area: "[[OSINT de infraestructura.base|OSINT de infraestructura]]"
---
---

<mark style="background: #ADCCFFA6;">Shodan, Censys y FOFA escanean Internet entero de forma continua e indexan los **servicios y banners** que encuentran</mark>. Consultarlos es OSINT puro: preguntas a su índice, no al objetivo. Encuentras los puertos, servicios, versiones y certificados expuestos del objetivo **sin enviarle un solo paquete**. Cada motor tiene su lenguaje; aprenderlos es la diferencia entre "buscar una IP" y mapear la superficie.

# Shodan

Filtros `clave:valor`; las palabras sueltas buscan en el texto del banner; el espacio es `AND` implícito; `-` niega. <mark style="background: #FFB8EBA6;">Usa códigos de país de dos letras (`country:ES`, no "Spain")</mark>.

```text
# Servicios de una organización / rango / dominio
org:"ACME Corporation"
net:203.0.113.0/24
ssl.cert.subject.cn:acme.com        # por el CN del certificado
hostname:acme.com

# Por servicio y versión (superficie de CVE)
product:MongoDB port:27017
http.title:"Grafana" country:ES
"RDP" port:3389 org:"ACME"
```

Cuenta antes de volcar, para no gastar créditos:

```shell-session
$ shodan count 'org:"ACME Corporation" port:3389'   # ¿cuántos RDP expuestos?
$ shodan search 'ssl.cert.subject.cn:acme.com'       # búsqueda completa
```

<mark style="background: #FF5582A6;">`ssl.cert.subject.cn:` es el pivote más potente de Shodan</mark>: el certificado delata todos los hosts que presentan ese dominio, aunque la IP no tenga PTR ni el host enlace a nada.

# Censys — ahora en CenQL

> [!warning]+ Censys cambió de lenguaje (verificar en la doc oficial)
> La nueva plataforma de Censys usa **CenQL**, no el antiguo `CSL`, y la migración **rompió mucho tooling viejo**. Además, en 2026 el tier gratuito sirve para consultas puntuales por la web y pivotes de certificado, pero **no para búsqueda por script/API** (de pago). Verifica el esquema de campos en la referencia oficial antes de montar queries. *(Dato de blog 2026, contrástalo.)*

CenQL usa rutas de campo explícitas con `AND`/`OR`:

```text
# Hosts de una organización con HTTP 200
services.http.response.status_code:200 AND autonomous_system.organization:"ACME"
# Por ASN y puerto (CLI)
$ censys search "autonomous_system.asn=12345 AND services.port=27017"
```

Censys destaca en **inteligencia de certificados** (su índice de certs es de los mejores) → [[03 - Inteligencia de certificados y DNS pasivo]].

# FOFA

Operadores: `=` contiene, `==` exacto, `&&` AND, `||` OR, `!=` NOT, `()` agrupa, `*` comodín.

```text
domain="acme.com"
app="Apache" && country="ES"
cert="acme.com" && port!="443"       # certs de acme fuera del 443
```

FOFA (chino) tiene buen índice de Asia y a veces ve lo que Shodan/Censys no. ZoomEye, Netlas y Criminal IP son alternativas con la misma idea y distinto índice.

# La misma búsqueda en los tres

<mark style="background: #8000E1A6;">Cambia la sintaxis, no el concepto</mark>. Buscar un servicio en un país:

| Motor | Consulta |
| - | - |
| Shodan | `port:502 country:PH` |
| Censys | `host.services.port=502 and host.location.country="Philippines"` |
| FOFA | `port="502" && country="PH"` |

Pásale la misma intención a los tres: los índices se complementan.

# La realidad del tier gratuito (2026)

<mark style="background: #FFB86CA6;">Los tiers gratuitos se han estrechado</mark>: sirven para consultas puntuales por la interfaz web, pero la **búsqueda por API/script** quiere dinero en los tres. Para automatizar sin pagar cada motor, `uncover` (de la suite [[Tools.base|ProjectDiscovery]]) unifica las APIs de varios motores tras una sola interfaz:

```shell-session
# uncover: una query, varios motores (necesita las API keys configuradas)
$ echo 'ssl.cert.subject.CN:"acme.com"' | uncover -e shodan,censys,fofa
```

Esto enlaza con el [[01 - Perfilar el perímetro antes de escanear|perímetro]] y con la matriz de escaneo del vault: motores pasivos para el inventario, Nmap para confirmar.

> [!info]+ Fuentes
> Sintaxis por motor y la migración a CenQL: [osintph, *Which OSINT Search Engine Free Tier Works in 2026*](https://blog.osintph.info/which-osint-search-engine-free-tier-actually-works-in-2026-shodan-vs-censys-vs-zoomeye-vs-fofa/); [Intigriti, *Shodan & Censys*](https://www.intigriti.com/researchers/blog/hacking-tools/complete-guide-to-finding-more-vulnerabilities-with-shodan-and-censys). Documentación oficial de cada motor para el esquema de campos vigente.
