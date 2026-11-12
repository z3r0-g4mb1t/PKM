---
tags:
  - OSINT/Infraestructura
  - Pentesting/Enumeracion
Descripción: "De la empresa a todos sus rangos IP vía su ASN: bgp.he.net, asnmap y Team Cymru, con el matiz de cuándo funciona y cuándo no (cloud)"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA]]"
Nota siguiente: "[[03 - Inteligencia de certificados y DNS pasivo]]"
Area: "[[OSINT de infraestructura.base|OSINT de infraestructura]]"
---
---

<mark style="background: #ADCCFFA6;">Un `ASN` (*Autonomous System Number*) identifica un conjunto de rangos IP bajo un mismo control administrativo</mark>. Si el objetivo tiene su propio ASN, puedes pasar del **nombre de la empresa** a **todos sus rangos de red** de una sola vez — y de ahí a todas sus IPs. Es recon pasivo puro: la información vive en los registros de enrutamiento BGP y en los RIR, no en el objetivo.

# De la organización al ASN a los rangos

El flujo:

```text
"ACME Corporation"  →  AS65000  →  203.0.113.0/24, 198.51.100.0/22, ...  →  todas sus IPs
```

## bgp.he.net (Hurricane Electric)

La vía web más rápida: busca el nombre de la organización en [bgp.he.net](https://bgp.he.net), que lista sus ASN y, por cada ASN, los **prefijos anunciados** (los rangos CIDR). También hace el camino inverso (una IP → su ASN y prefijo).

## asnmap (ProjectDiscovery)

Para la línea de comandos, `asnmap` acepta ASN, IP, dominio o **nombre de organización** y devuelve los CIDR:

```shell-session
# De nombre de org / ASN / dominio a rangos CIDR
$ echo 'AS65000' | asnmap
$ echo 'acme.com' | asnmap
# Encadenar: rangos → escaneo de un puerto con naabu
$ echo 'AS65000' | asnmap | naabu -p 443,3389
```

<mark style="background: #FFB86CA6;">Ese encadenado es el salto de pasivo a activo</mark>: asnmap (pasivo, registros) te da el territorio, y naabu/Nmap (activo) lo confirman. Tiene referencia propia en [[Tools.base|ProjectDiscovery]].

## Team Cymru y bgp.tools

- **Team Cymru** ofrece mapeo IP→ASN por DNS/WHOIS (su servicio de *origin lookup* mapea una IP a su ASN y prefijo de origen BGP).
- **bgp.tools** exporta datos de la tabla de enrutamiento y nombres de ASN — **requiere una cabecera `User-Agent` propia** que te identifique en cada petición.

> [!warning]+ BGPView parece haber cerrado
> `BGPView` (y su API gratuita), muy citado en guías antiguas, habría dejado de funcionar el 26 de noviembre de 2025 *(dato único, sin confirmar — verifícalo)*. Si una guía te manda a BGPView y no responde, usa bgp.he.net, bgp.tools o asnmap.

# El matiz que decide si esto sirve: cloud

<mark style="background: #FF5582A6;">El ASN solo mapea al objetivo si el objetivo **posee** sus rangos</mark>. Esto funciona de maravilla con ISPs, grandes empresas, universidades y hostings que tienen su propio ASN y netblocks. Pero:

- Si el objetivo está **en la nube** (AWS, Azure, GCP), sus IPs pertenecen al ASN del **proveedor cloud**, no al suyo. `asmnap "ACME"` no devolverá nada útil, y buscar el ASN de AWS te da millones de IPs ajenas.
- Para objetivos cloud, el camino es otro: subdominios → resolución → identificar qué IPs son suyas por [[03 - Inteligencia de certificados y DNS pasivo|certificados y DNS pasivo]] y por los servicios que corren, no por el ASN.

<mark style="background: #8000E1A6;">Comprueba primero si el objetivo tiene ASN propio</mark> (bgp.he.net por su nombre); si no aparece, está en infraestructura compartida y el ASN no es tu vía.

# Qué haces con los rangos

Una vez tienes los CIDR propios del objetivo:

- Los pasas a los [[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA|motores de dispositivos]] (`net:203.0.113.0/24` en Shodan) para ver servicios expuestos **sin escanear**.
- Los reservas para el escaneo activo (Nmap/naabu) cuando el alcance lo autorice.
- Detectas rangos "olvidados" — bloques antiguos que aún anuncian y nadie vigila.

> [!info]+ Fuentes
> [bgp.he.net](https://bgp.he.net) (Hurricane Electric), [bgp.tools](https://bgp.tools) (requiere User-Agent propio), [Team Cymru IP-to-ASN](https://team-cymru.com/ip-asn-mapping). [asnmap](https://github.com/projectdiscovery/asnmap) (ProjectDiscovery; org/IP/ASN/dominio → CIDR). Cierre de BGPView (nov-2025): dato único sin confirmar.
