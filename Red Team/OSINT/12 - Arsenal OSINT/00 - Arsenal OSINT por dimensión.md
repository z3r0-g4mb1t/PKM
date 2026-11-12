---
tags:
  - OSINT/Arsenal
  - Tipo/Arsenal
Descripción: "Índice consolidado del arsenal OSINT por dimensión: qué herramienta usar para qué, con enlace a la nota que la enseña"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Estado del ecosistema OSINT en 2026]]"
Area: "[[Arsenal OSINT.base|Arsenal OSINT]]"
---
---

Referencia rápida del arsenal OSINT, organizado por dimensión. Cada herramienta se **enseña** en su nota (columna "Dónde"); esto es el índice para "¿qué cojo para esto?". El [[01 - Estado del ecosistema OSINT en 2026|estado de cada tool]] (viva/muerta/cambiada) va en la nota siguiente.

# OPSEC e infraestructura del investigador

| Herramienta | Para qué | Dónde |
| - | - | - |
| fakenamegenerator, thispersondoesnotexist | Identidad/rostro de sock puppet | [[01 - Sock puppets - crear y mantener identidades de investigación\|Sock puppets]] |
| Trace Labs OSINT VM, CSI Linux, Tsurugi, Tails, Whonix | Máquina de investigación aislada | [[02 - Infraestructura de investigación - VM, red y navegador\|Infra]] |
| Mullvad Browser, Firefox Containers | Compartimentar sesiones | [[02 - Infraestructura de investigación - VM, red y navegador\|Infra]] |
| browserleaks, ipleak | Probar fugas (IP/DNS/WebRTC) | [[03 - Cómo no quemar la operación - fugas y errores comunes\|Fugas]] |

# Google Dorking

| Herramienta | Para qué | Dónde |
| - | - | - |
| Operadores + GHDB (Exploit-DB) | Construir dorks dirigidos | [[01 - Operadores de búsqueda de Google\|Operadores]] |
| Custom Search JSON API | Automatizar dorks de forma fiable | [[03 - La Google Hacking Database y la automatización\|GHDB]] |
| trufflehog, gitleaks | Secretos en repos | [[04 - Dorking más allá de Google\|Más allá de Google]] |
| publicwww, grep.app | Buscar en código fuente de la web | [[04 - Dorking más allá de Google\|Más allá de Google]] |

# Personas e identidades

| Herramienta | Para qué | Dónde |
| - | - | - |
| Sherlock, Maigret, WhatsMyName, Blackbird | Enumerar username cross-platform | [[01 - Enumeración de usernames cross-platform\|Usernames]] |
| holehe, GHunt, Epieos | Correo → cuentas y Google | [[02 - OSINT de correo electrónico\|Correo]] |
| PhoneInfoga | Teléfono → metadatos y huella | [[03 - OSINT de teléfono\|Teléfono]] |
| OpenCorporates, BORME, people search | Nombre → registros y personas | [[04 - Buscadores de personas y registros públicos\|Registros]] |

# SOCMINT

| Herramienta | Para qué | Dónde |
| - | - | - |
| CrossLinked, linkedin2username | Empleados de una empresa | [[01 - LinkedIn - la mina del red teamer\|LinkedIn]] |
| twscrape + operadores de X | Recolección en X/Twitter | [[02 - X-Twitter, Reddit y foros abiertos\|X/Reddit]] |
| Instaloader | Perfiles públicos de Instagram | [[03 - Instagram, Facebook y TikTok\|Instagram/FB]] |
| Telethon, Telepathy, TGStat, Lyzem | OSINT de Telegram | [[04 - Telegram y Discord - las plataformas abiertas\|Telegram]] |

# Empresas e infraestructura

| Herramienta | Para qué | Dónde |
| - | - | - |
| OpenCorporates, SEC EDGAR, Companies House, North Data, Aleph | Registros y propiedad | [[01 - Registros mercantiles y financieros\|Registros]] |
| BuiltWith, Wappalyzer | Tech stack web | [[02 - Tech stack y activos digitales\|Tech stack]] |
| `dig` MX/SPF/DMARC | Proveedor de correo y spoofability | [[02 - Tech stack y activos digitales\|Tech stack]] |
| Shodan, Censys, FOFA | Servicios/dispositivos expuestos | [[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA\|Device search]] |
| bgp.he.net, asnmap, Team Cymru | ASN → rangos IP | [[02 - ASN, BGP y mapeo de rangos IP\|ASN]] |
| crt.sh, SecurityTrails, Validin | Certificados y DNS pasivo | [[03 - Inteligencia de certificados y DNS pasivo\|Certs/DNS]] |

# Brechas y credenciales

| Herramienta | Para qué | Dónde |
| - | - | - |
| HIBP, Hudson Rock | Exposición y stealer logs | [[01 - Buscadores de brechas - HIBP, Dehashed e IntelX\|Buscadores]] |
| Dehashed | Credenciales reales por dominio/correo | [[01 - Buscadores de brechas - HIBP, Dehashed e IntelX\|Buscadores]] |
| IntelX | Dónde aparece un selector (leaks/pastes) | [[01 - Buscadores de brechas - HIBP, Dehashed e IntelX\|Buscadores]] |

# GEOINT, imágenes y metadatos

| Herramienta | Para qué | Dónde |
| - | - | - |
| exiftool, MAT2 | Leer/limpiar metadatos (GPS, autor) | [[01 - Metadatos y EXIF\|Metadatos]] |
| Yandex, Google Lens, TinEye | Búsqueda inversa de imagen | [[02 - Búsqueda inversa de imágenes y geolocalización por IA\|Inversa]] |
| PimEyes, FaceCheck.ID | Reconocimiento facial (⚠️ RGPD) | [[02 - Búsqueda inversa de imágenes y geolocalización por IA\|Inversa]] |
| GeoSpy, GeoSeer | Geolocalización por IA (hipótesis) | [[02 - Búsqueda inversa de imágenes y geolocalización por IA\|Inversa]] |
| Overpass Turbo, Google Earth, Mapillary | Geolocalización por contenido | [[03 - Geolocalización - el método Bellingcat\|Geolocalización]] |
| SunCalc, Shadow Finder, InVID/WeVerify | Cronolocalización y vídeo | [[04 - Cronolocalización, vídeo y satélite\|Cronolocalización]] |

# Dark web y frameworks

| Herramienta | Para qué | Dónde |
| - | - | - |
| Ahmia, OnionSearch, dark.fail, ransomware.live | Buscar/monitorizar la dark web | [[02 - Buscar en la dark web - motores y directorios\|Dark web]] |
| SpiderFoot | Agregador OSINT autónomo | [[01 - SpiderFoot\|SpiderFoot]] |
| recon-ng, Amass, Maltego | Modular, infra y análisis de enlaces | [[02 - recon-ng y Maltego\|recon-ng/Maltego]] |
| ProjectDiscovery (subfinder/dnsx/httpx/uncover) | Pipeline de recon | [[03 - Pipelines, OSINT asistido por IA y deconfliction\|Pipelines]] + [[Tools.base\|Tools]] |

# Contra-OSINT

| Herramienta | Para qué | Dónde |
| - | - | - |
| DeleteMe, Incogni, DROP (DELETE Act) | Reducir huella personal | [[01 - Reducción de huella personal y corporativa\|Reducción]] |
| crt.sh, CertSpotter, dnstwist | Detectar phishing/typosquat | [[02 - Detección y deception contra OSINT\|Deception]] |
| canarytokens.org | Trampas que delatan al atacante | [[02 - Detección y deception contra OSINT\|Deception]] |

> [!info]+ Herramientas con ficha propia en Tools/
> La suite [[Tools.base\|ProjectDiscovery]] (`subfinder`, `dnsx`, `httpx`, `naabu`, `asnmap`, `uncover`, `notify`) ya tiene referencia en `02 - Recursos/🛠️ Tools/`. El resto del arsenal OSINT se enseña dentro de esta área, en las notas enlazadas arriba.
