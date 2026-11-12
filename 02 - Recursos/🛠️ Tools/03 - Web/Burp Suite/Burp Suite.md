---
tags:
  - Web/Red-Team
  - Proxies
  - Herramientas
  - Tipo/Introduccion
Descripción: "Burp Suite (PortSwigger) es el proxy de interceptación estándar del pentest web: interceptar, repetir, fuzzear y escanear tráfico HTTP(S)"
Fecha de actualización: 2026-11-12
Nota previa:
Nota siguiente:
Area: "[[Burp Suite.base|Burp Suite]]"
---
---

<mark style="background: #ADCCFFA6;">[Burp Suite](https://portswigger.net/burp) (PortSwigger) es el proxy de interceptación **estándar de facto** del pentest de aplicaciones web.</mark> Se sitúa entre el navegador y el servidor y deja ver, modificar, repetir, fuzzear y escanear cada petición HTTP(S). Trae un **Chromium embebido** preconfigurado para probar sin tocar el navegador del sistema.

> [!info]+ Esta nota es la ficha de la herramienta
> El *qué es, ediciones, instalación y mapa de funciones* vive aquí. El **cómo se usa** —interceptar, Repeater, Intruder, Match & Replace, scanner, extensiones, OPSEC— está desarrollado paso a paso en el sub-tema [[Proxies web.base|Proxies web]], enlazado en cada fila de la tabla de abajo. Esta ficha no reexplica la metodología: la indexa.

# Ediciones y precio (2026)

| Edición | Precio | Qué incluye / qué falta |
| - | - | - |
| **Community** | Gratis | Proxy, Repeater, Decoder, Comparer. <mark style="background: #FFB86CA6;">Sin scanner activo, Intruder **con retardo artificial**, solo proyectos temporales (no se guarda el progreso)</mark> y BApp Store limitada |
| **Professional** | <mark style="background: #FF5582A6;">~$499/usuario/año</mark> | Todo: scanner DAST, Intruder a full, Burp Collaborator privado, project files, toda la BApp Store, Burp AI |
| **DAST** (ex-*Enterprise*) | Bajo presupuesto | Escaneo automatizado continuo para CI/CD y flotas de apps; no es la herramienta manual |

> [!warning]+ El precio subió en 2026 y la licencia es por asiento
> PortSwigger aplicó una **subida global el 6 de enero de 2026**: Professional pasó de **$449 a ~$499/usuario/año** ([lista de precios actual](https://portswigger.net/buy/pro)). La licencia es **por asiento** (cada persona del equipo que lo toque necesita el suyo). Si un writeup o una nota vieja dice £449/$449, está desfasado — confirma siempre en `portswigger.net` antes de presupuestar.

# Instalación

Instaladores nativos (Windows/Linux/macOS) desde la [página de releases](https://portswigger.net/burp/releases/), o el `JAR` multiplataforma. Burp **incluye su propio JRE**, así que no hace falta Java aparte:

```shell-session
$ burpsuite                       # si está en el PATH / instalado
$ java -jar /ruta/burpsuite.jar   # JAR portable
```

Al arrancar se elige tipo de proyecto (en Community, solo **temporal**) y configuración (`Use Burp Defaults`). Viene preinstalado en Kali, Parrot y Pwnbox.

# Qué incluye (y dónde está el "cómo")

<mark style="background: #ADCCFFA6;">Todo gira en torno al **Proxy**: captura el tráfico y lo reparte a las demás herramientas.</mark> El mapa de componentes → nota del vault que lo desarrolla:

| Componente | Para qué | Metodología en |
| - | - | - |
| **Proxy / Intercept** | Ver y pausar peticiones y respuestas al vuelo | [[02 - Interceptación de peticiones]] · [[03 - Interceptación de respuestas]] |
| **Repeater** | Repetir y editar una petición manualmente | [[05 - Repeater - repetir y modificar peticiones]] |
| **Intruder** | Fuzzing y fuerza bruta (sniper, cluster bomb, pitchfork…) | [[08 - Fuzzing web - Burp Intruder y ZAP Fuzzer]] |
| **Match & Replace** | Reescribir peticiones/respuestas automáticamente | [[04 - Modificación automática (Match and Replace)]] |
| **Inspector / Decoder** | Codificar/decodificar y editar estructuras (`base64`, URL, JWT) | [[06 - Codificación y decodificación]] |
| **Scanner** (Pro) | DAST activo y pasivo | [[09 - Escáner de vulnerabilidades - Burp y ZAP Scanner]] |
| **Extensiones / BApp Store** | Ampliar Burp (Autorize, Param Miner, InQL, Turbo Intruder…) | [[10 - Extensiones y BApp Store]] |
| **Collaborator** (Pro) | Canal out-of-band para bugs ciegos (SSRF, XXE, SQLi OOB) | [[18 - Exfiltración de datos ciega (OOB)]] |
| **Sequencer** | Analizar la entropía de tokens de sesión | [[13 - Session IDs débiles]] |

# Estado y novedades (2026)

- <mark style="background: #FFB86CA6;">**Burp AI** (Pro, desde 2025)</mark>: funciones asistidas por IA —*Shadow Repeater* (muta automáticamente lo que pruebas en Repeater), *Explore Issue*, *Explainer* y reducción de falsos positivos— más una plataforma de IA que las **extensiones** pueden consumir. Gasta *AI credits* de la cuenta.
- <mark style="background: #FFB86CA6;">**Burp AT** (*agentic AI*, beta pública para Pro)</mark>: agentes que **operan** las herramientas de Burp con el contexto del proyecto y una *skill* de pentesting, en modos **Manual / Smart / Autónomo** con aprobaciones por herramienta y **todo auditado** en el proyecto. Es el salto de "IA que asiste" a "IA que ejecuta pasos" dentro del scope que Burp impone.
- <mark style="background: #FF5582A6;">**Servidor MCP oficial**</mark>: la extensión [`PortSwigger/mcp-server`](https://github.com/PortSwigger/mcp-server) (GPL-3, abr-2025) expone Burp a clientes **MCP** (Claude Desktop, agentes) por SSE o un *stdio proxy*; se activa en la pestaña `MCP`. Deja pilotar Proxy/Repeater/scanner desde un LLM y enlaza esta herramienta con la superficie de [[MCP.base|MCP y seguridad de agentes]] y su auditoría con [[MCP-Scan.base|mcp-scan]]. <mark style="background: #FFB86CA6;">Exponer Burp por MCP es dar a un modelo control de tu tráfico</mark> — trátalo como superficie de ataque, no solo como comodidad.
- **Montoya API**: la API moderna de extensiones (sustituye a la legacy *Extender*/`IBurpExtender`). Las BApps nuevas se escriben contra ella — relevante al desarrollar [[12 - Herramientas para pentesting de APIs|tooling propio]].
- **HTTP/2 nativo** y soporte de WebSockets en Proxy/Repeater; el Inspector edita pseudo-headers HTTP/2 (clave para [[14 - HTTP2 Downgrading|downgrade smuggling]]).
- **Burp Suite DAST** es el rebranding de *Burp Suite Enterprise*. El versionado es **mensual** (`AAAA.N`): para el build exacto, mira [releases](https://portswigger.net/burp/releases); la *currency* real hoy es el bloque de IA (AI / AT / MCP).

> [!info]+ Alternativas
> **Libre y open-source**: [[ZAP.base|ZAP]] (sin funciones de pago). **Moderna**: **Caido** (escrito en Rust, UI por proyectos, creciendo fuerte en bug bounty) — comparadas en [[13 - Flujo profesional y alternativas modernas]]. Para recon a escala, el proxy se combina con el pipeline de [[ProjectDiscovery.base|ProjectDiscovery]] (`httpx -proxy`).

> [!info]+ Fuentes
> [PortSwigger — Burp Suite](https://portswigger.net/burp) · [precios](https://portswigger.net/buy/pro) · [Web Security Academy](https://portswigger.net/web-security) · [Montoya API](https://portswigger.github.io/burp-extensions-montoya-api/javadoc/). La referencia operativa completa está en [[Proxies web.base|Proxies web]].
