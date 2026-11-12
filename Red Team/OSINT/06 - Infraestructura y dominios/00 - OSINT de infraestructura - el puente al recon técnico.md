---
tags:
  - OSINT/Infraestructura
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Qué parte de la infraestructura cubre OSINT y qué remite al recon web/footprinting: el puente y lo específicamente OSINT (device search, ASN, DNS pasivo)"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA]]"
Area: "[[OSINT de infraestructura.base|OSINT de infraestructura]]"
---
---

La infraestructura (dominios, IPs, servicios) es donde el OSINT se solapa con el recon técnico que el vault ya cubre. Esta nota marca **la frontera**: qué está ya hecho y hay que enlazar, y qué es específicamente OSINT y se desarrolla aquí.

# Lo que ya está cubierto (enlázalo, no lo repitas)

El recon web y el footprinting ya desarrollan el grueso del recon técnico de infraestructura:

| Técnica | Dónde vive |
| - | - |
| `WHOIS` y propiedad de dominio | [[01 - WHOIS]] |
| DNS: fundamentos, `dig`, transferencias de zona | [[02 - DNS - fundamentos]], [[03 - Enumeración DNS con dig]], [[04 - Transferencias de zona DNS]] |
| Enumeración de subdominios (pasiva, brute, CT) | [[05 - Enumeración de subdominios]], [[06 - Fuerza bruta de subdominios]], [[07 - Certificate Transparency logs]] |
| Virtual hosts y fingerprinting web | [[08 - Virtual Hosts]], [[09 - Fingerprinting web]] |
| Origen real tras WAF/CDN | [[28 - Origen real tras WAF, CDN y balanceadores]] |
| Enumeración de servicios por protocolo | [[00 - Principios y metodología de enumeración\|Footprinting]] |

<mark style="background: #FF5582A6;">No reexplico nada de eso aquí</mark>. Si necesitas enumerar subdominios o resolver DNS, ve a esas notas.

# Lo que es específicamente OSINT (y se desarrolla aquí)

Lo que esas notas tocan de pasada pero no desarrollan como disciplina OSINT — todo **pasivo**, preguntando a índices de terceros que ya escanearon Internet por ti:

- <mark style="background: #ADCCFFA6;">**Motores de búsqueda de dispositivos** (Shodan, Censys, FOFA)</mark>: índices de servicios y banners de todo Internet. Encuentras los servicios expuestos del objetivo **sin escanearlo** → [[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA]].
- <mark style="background: #ADCCFFA6;">**ASN, BGP y rangos IP**</mark>: de la empresa a su número de sistema autónomo y a **todos** sus rangos de red → [[02 - ASN, BGP y mapeo de rangos IP]].
- <mark style="background: #ADCCFFA6;">**Inteligencia de certificados y DNS pasivo**</mark>: pivotar por certificados y ver el **histórico** de qué resolvía a dónde → [[03 - Inteligencia de certificados y DNS pasivo]].

# Por qué esta capa es "puente"

<mark style="background: #8000E1A6;">El OSINT de infraestructura te da el inventario sin tocar al objetivo; el recon técnico lo confirma y profundiza tocándolo</mark>. El flujo encadena ambos: empiezas en Shodan/Censys/ASN (pasivo, este bloque), construyes el mapa de IPs y servicios, y solo entonces escalas al escaneo activo con Nmap y a la enumeración por servicio del [[00 - Principios y metodología de enumeración|footprinting]] — sobre lo que ya sabes que existe y está en alcance.

> [!info]+ Herramientas en Tools/
> Varias herramientas de este bloque ya tienen referencia propia en el vault: la suite [[Tools.base|ProjectDiscovery]] (`asnmap`, `dnsx`, `tlsx`, `uncover`), que automatiza buena parte de esto. Aquí se enseña el **método y la consulta**; el detalle de cada herramienta, en su nota de Tools.

> [!info]+ Fuentes
> Marco de attack-surface mapping pasivo: [Intigriti, *Finding vulnerabilities with Shodan and Censys*](https://www.intigriti.com/researchers/blog/hacking-tools/complete-guide-to-finding-more-vulnerabilities-with-shodan-and-censys).
