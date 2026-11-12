---
tags:
  - OSINT/Infraestructura
  - Pentesting/Enumeracion
Descripción: "Pivotar por certificados (crt.sh, Censys) y ver el histórico de resoluciones con DNS pasivo para hallar subdominios y el origen real tras un CDN"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - ASN, BGP y mapeo de rangos IP]]"
Nota siguiente:
Area: "[[OSINT de infraestructura.base|OSINT de infraestructura]]"
---
---

Dos fuentes pasivas cierran el mapa de infraestructura: los **certificados** (que delatan nombres de host) y el **DNS pasivo** (el histórico de qué resolvía a dónde). Ambas ven cosas que una consulta DNS en vivo no muestra — subdominios olvidados, el origen real tras un CDN, infraestructura relacionada.

# Inteligencia de certificados

Cada certificado TLS emitido es **público** (se registra en los [[07 - Certificate Transparency logs|Certificate Transparency logs]]). Eso convierte los certificados en una mina de nombres de host:

- <mark style="background: #ADCCFFA6;">**crt.sh** busca en los CT logs por dominio</mark> y lista todos los certificados emitidos — cada uno con su `CN` y sus `SAN` (nombres alternativos), que son subdominios:

```shell-session
# Todos los certificados (y por tanto subdominios) de acme.com
$ curl -s 'https://crt.sh/?q=%25.acme.com&output=json' | jq -r '.[].name_value' | sort -u
```

El `%25` es `%` URL-encoded (comodín SQL): `%.acme.com` saca todos los subdominios. <mark style="background: #FFB86CA6;">Un certificado para `vpn-internal.acme.com` revela que ese host existe aunque no esté en el DNS público ni enlazado en ninguna parte</mark>.

- **Censys** tiene uno de los mejores índices de certificados: permite pivotar al revés (dado un certificado, qué **IPs** lo presentan) y encontrar infraestructura que comparte el mismo cert o la misma clave → [[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA|Censys]].

El pivote por certificado es potente porque los SAN agrupan dominios "hermanos" (un cert con `acme.com`, `acme.de`, `acme-internal.net` te revela toda la familia de dominios de golpe).

# DNS pasivo

<mark style="background: #ADCCFFA6;">El DNS pasivo es el **histórico** de resoluciones DNS observadas por sensores repartidos por Internet</mark>. En vez de preguntar "¿a qué resuelve `acme.com` ahora?" (DNS en vivo), pregunta "¿a qué ha resuelto `acme.com` a lo largo del tiempo?" y "¿qué dominios han resuelto a esta IP?". Proveedores: **SecurityTrails**, **Validin**, **DNSDB** (DomainTools/Farsight), **Mnemonic**.

Dos usos que valen oro:

- <mark style="background: #FF5582A6;">**El origen real tras un CDN**</mark>: si hoy `acme.com` resuelve a Cloudflare, el DNS pasivo puede mostrar la **IP de origen** a la que resolvía *antes* de ponerse detrás del CDN. Esa IP, si sigue respondiendo, salta el WAF por completo. Complementa [[28 - Origen real tras WAF, CDN y balanceadores|origen real tras WAF/CDN]].
- **Subdominios y hosts olvidados**: el histórico revela subdominios que ya no están en la zona pero cuyo host sigue vivo.

```text
# Consulta conceptual de DNS pasivo (vía SecurityTrails / Validin)
acme.com      → histórico de registros A:  198.51.100.10 (2023) → 104.x (Cloudflare, 2024→)
               ↑ 198.51.100.10 puede ser el origen real, aún alcanzable
```

# Reverse IP: quién más vive en esa IP

El DNS pasivo también hace **reverse IP**: dada una IP, qué dominios han apuntado a ella. <mark style="background: #8000E1A6;">Útil para vhosts compartidos</mark> — si el objetivo comparte servidor con otros sitios, descubres vecinos (y posibles vías laterales), y confirmas qué dominios son realmente suyos. El operador `ip:` de [[04 - Dorking más allá de Google|Bing]] hace una versión ligera de esto.

# Dónde encaja

Esta nota cierra el OSINT de infraestructura: certificados + DNS pasivo + [[02 - ASN, BGP y mapeo de rangos IP|ASN]] + [[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA|device search]] te dan el mapa de hosts, IPs y servicios del objetivo **sin tocarlo**. La enumeración de subdominios "en vivo" (brute, resolución masiva) y el escaneo activo viven en [[05 - Enumeración de subdominios|recon web]] y el [[00 - Principios y metodología de enumeración|footprinting]] — este bloque es el **inventario pasivo** que los precede.

> [!info]+ Fuentes
> CT logs y pivote por certificado: [crt.sh](https://crt.sh), Censys. Concepto y proveedores de DNS pasivo: [SecurityTrails](https://securitytrails.com), Validin, DNSDB (DomainTools). Origen tras CDN vía histórico: técnica estándar, ver [[28 - Origen real tras WAF, CDN y balanceadores]].
