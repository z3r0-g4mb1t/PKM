---
tags:
  - Web/Red-Team
  - Proxies
  - Herramientas
  - Tipo/Introduccion
Descripción: "ZAP (antes 'OWASP ZAP') es el proxy de interceptación libre y open-source de referencia: escáner activo gratuito, HUD en navegador y automatización para CI/CD"
Fecha de actualización: 2026-11-09
Nota previa:
Nota siguiente:
Area: "[[ZAP.base|ZAP]]"
---
---

<mark style="background: #ADCCFFA6;">[ZAP](https://www.zaproxy.org/) (*Zed Attack Proxy*) es el proxy de interceptación **libre y open-source** de referencia</mark> (licencia Apache 2.0). Cubre lo mismo que [[Burp Suite.base|Burp]] —interceptar, repetir, fuzzear, escanear— pero <mark style="background: #FFB86CA6;">sin edición de pago: su **escáner activo y la automatización para CI/CD son gratuitos</mark>, justo lo que en Burp exige la licencia Professional.

> [!warning]+ Ya NO es "OWASP ZAP"
> El proyecto **dejó de ser de OWASP en agosto/septiembre de 2023** para asegurar financiación estable: primero pasó al **Software Security Project (SSP)** y desde **septiembre de 2024 lo mantiene [Checkmarx](https://www.zaproxy.org/blog/2024-09-24-zap-has-joined-forces-with-checkmarx/)** (los tres líderes del proyecto son empleados de Checkmarx). <mark style="background: #FF5582A6;">Sigue siendo gratuito, open-source y controlado por el ZAP Core Team</mark>, pero la propia documentación [pide dejar de llamarlo "OWASP ZAP"](https://www.zaproxy.org/docs/zap-ownership/) — cualquier material que use ese nombre está desfasado. (El nombre de este fichero conserva el "OWASP" histórico para no romper backlinks.)

> [!info]+ Esta nota es la ficha de la herramienta
> El *qué es, instalación y mapa de funciones* vive aquí; la **metodología de uso** (interceptar, editar, fuzzear, escanear) se desarrolla en el sub-tema [[Proxies web.base|Proxies web]], donde Burp y ZAP se enseñan en paralelo.

# Instalación

Instaladores nativos (Windows/Linux/macOS) desde la [página de descargas](https://www.zaproxy.org/download/), el `JAR` multiplataforma, o contenedor:

```shell-session
$ zaproxy                                   # GUI (instalado / en PATH)
$ docker run -u zap -p 8080:8080 \
    zaproxy/zap-stable zap.sh -daemon       # modo headless para automatización
```

Viene preinstalado en Kali y Parrot. Al arrancar pide proyecto temporal o persistente (`no` para no guardar). La interceptación está **desactivada por defecto** (botón verde en la barra; `CTRL+B` alterna).

# Qué incluye (y dónde está el "cómo")

| Componente | Para qué | Metodología en |
| - | - | - |
| **Break** (intercept) | Pausar y editar peticiones/respuestas | [[02 - Interceptación de peticiones]] · [[03 - Interceptación de respuestas]] |
| **HUD** (*Heads Up Display*) | Controlar ZAP **desde el propio navegador**: romper, mostrar campos ocultos, ver comentarios HTML | [[00 - Introducción a los proxies web]] |
| **Manual Request Editor** | Repetir y modificar una petición (≈ Repeater de Burp) | [[05 - Repeater - repetir y modificar peticiones]] |
| **Fuzzer** | Fuzzing de valores con payloads y *fuzzers* integrados | [[08 - Fuzzing web - Burp Intruder y ZAP Fuzzer]] |
| **Replacer** | Reescritura automática de peticiones/respuestas (≈ Match & Replace) | [[04 - Modificación automática (Match and Replace)]] |
| **Scanner** (pasivo + activo) | DAST **gratuito**, con *Spider* y *AJAX Spider* para apps JS | [[09 - Escáner de vulnerabilidades - Burp y ZAP Scanner]] |
| **Marketplace de add-ons** | Ampliar ZAP (muchas capacidades vienen como add-on) | [[10 - Extensiones y BApp Store]] |
| **Automation Framework / API** | Correr ZAP **headless** y por API en un pipeline CI/CD | [[13 - Flujo profesional y alternativas modernas]] |

# Fortalezas frente a Burp (y dónde pierde)

- <mark style="background: #8000E1A6;">A favor de ZAP</mark>: **escáner activo gratuito**, **automatización/API de primera** (pensada para DevSecOps y CI/CD), el **HUD** en navegador, y cero coste ni límites artificiales.
- <mark style="background: #FFB8EBA6;">A favor de Burp</mark>: ecosistema de **extensiones** más maduro (BApp Store: Autorize, Turbo Intruder, Param Miner…), **Collaborator** out-of-band integrado, y un flujo manual más pulido que la mayoría de pentesters prefiere.

En la práctica muchos equipos usan **los dos**: ZAP para el escaneo automatizado en el pipeline, Burp para el testing manual profundo. Las alternativas modernas (Caido) y el encaje en el flujo real están en [[13 - Flujo profesional y alternativas modernas]].

> [!info]+ Fuentes
> [zaproxy.org](https://www.zaproxy.org/) · [ZAP se une a Checkmarx (2024)](https://www.zaproxy.org/blog/2024-09-24-zap-has-joined-forces-with-checkmarx/) · [ZAP sale de OWASP (2023)](https://www.zaproxy.org/blog/2023-08-01-zap-is-joining-the-software-security-project/) · [Automation Framework](https://www.zaproxy.org/docs/automate/). Referencia operativa completa en [[Proxies web.base|Proxies web]].
