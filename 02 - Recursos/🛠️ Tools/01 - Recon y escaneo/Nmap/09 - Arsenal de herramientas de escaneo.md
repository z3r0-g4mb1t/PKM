---
tags:
  - Pentesting/Enumeracion
  - Escaneo/Redes
  - Linux
  - Tipo/Arsenal
Descripción: "Nmap es preciso pero lento a escala: un -p- sobre un /16 es inviable en tiempo real"
Fecha de actualización: 2026-11-12
Nota previa: "[[08 - Detección de escaneos y evasión moderna]]"
Nota siguiente:
Area: "[[Nmap.base|Nmap]]"
---
Nmap es preciso pero **lento a escala**: un `-p-` sobre un `/16` es <mark style="background: #FF5582A6;">inviable en tiempo real</mark>. El pentest moderno lo resuelve con un pipeline de **dos fases** — <mark style="background: #ADCCFFA6;">descubrir puertos rápido con un escáner asíncrono</mark>, y luego pasar <mark style="background: #ADCCFFA6;">Nmap `-sCV` solo a lo que está vivo</mark>. Esta nota es el arsenal que rodea a Nmap para automatizar detección, enumeración y registro en un engagement real.

# Fase 1 — Descubrimiento rápido a escala

## masscan

[[Masscan.base|masscan]] <mark style="background: #ADCCFFA6;">usa una pila de red propia stateless que inyecta paquetes a velocidades imposibles para Nmap</mark>: puede barrer Internet entero en minutos. Es la opción para auditar un `/8` o `/16` cloud (netalith, [*Nmap vs Masscan vs RustScan 2026*](https://netalith.com/blogs/cybersecurity/nmap-vs-masscan-vs-rustscan-2026-comparison)). No hace *fingerprinting* — solo dice qué puertos están abiertos.

```shell-session
$ sudo masscan 10.129.0.0/16 -p1-65535 --rate 10000 -oL masscan.txt
```

> [!warning]+ masscan es ruidoso y frágil en redes malas
> A `--rate` alto es un cañón: <mark style="background: #FFB86CA6;">sube el riesgo de tumbar servicios y de saltar cualquier detección de flujo</mark>. Además **degrada con pérdida de paquetes** — en redes inestables da falsos negativos. Ajusta `--rate` al enlace y confirma siempre con Nmap.

## RustScan

[[RustScan.base|RustScan]] <mark style="background: #ADCCFFA6;">(Rust, asíncrono) recorre los 65.535 puertos en segundos y canaliza automáticamente los abiertos a Nmap</mark> para el análisis profundo (netalith, 2026). Es el pegamento ideal de las dos fases en un solo comando:

```shell-session
$ rustscan -a 10.129.2.28 --range 1-65535 -- -sCV -oA target
#           └ descubre rápido              └ todo lo tras '--' va a Nmap
```

**Contrapartida**: es <mark style="background: #FF5582A6;">poco fiable en redes inestables</mark> (s0cm0nkey, [*Port Scanner Shootout*](https://s0cm0nkey.gitbook.io/port-scanner-shootout/)).

## naabu

`naabu` (Go, de ProjectDiscovery) hace descubrimiento SYN/CONNECT concurrente con bajo consumo, soporta TCP/UDP y **no depende de Nmap**. <mark style="background: #FFB8EBA6;">Su fuerte es integrarse con el resto de la suite PD</mark> (`subfinder → naabu → httpx → nuclei`):

```shell-session
$ naabu -host 10.129.2.28 -top-ports 1000 -o ports.txt
$ naabu -list hosts.txt -p - -nmap-cli 'nmap -sCV'   # descubre y lanza Nmap
```

| **Herramienta**     | **Velocidad** | **Precisión**                     | **Cuándo**                                         |
| ------------------- | ------------- | --------------------------------- | -------------------------------------------------- |
| [[Nmap.base\|Nmap]] | Baja          | **Máxima** (stateful, reintentos) | Enumeración profunda dirigida; redes con pérdidas. |
| *masscan*           | **Máxima**    | Baja (solo puerto)                | Rangos enormes en redes estables (`/16`, `/8`).    |
| *RustScan*          | Alta          | Media (delega en Nmap)            | Un host / rango pequeño, todo-en-uno con Nmap.     |
| *naabu*             | Alta          | Media                             | Pipelines automatizados (suite ProjectDiscovery).  |

# Fase 2 — Profundidad (Nmap)

<mark style="background: #ADCCFFA6;">Nmap sigue siendo el patrón oro para versión</mark>, SO y NSE, y <mark style="background: #FFB8EBA6;">el que mejor aguanta redes con pérdida de paquetes</mark> gracias a sus reintentos (cyberleveling, [*Nmap vs RustScan vs Masscan*](https://cyberleveling.com/blog/nmap-vs-rustscan-vs-masscan)). <mark style="background: #FFB86CA6;">Recibe la lista de puertos de la fase 1 y hace el trabajo fino</mark>: `nmap -p<puertos> -sCV`.

# Recon pasivo (sin tocar el objetivo)

La <mark style="background: #ADCCFFA6;">forma más sigilosa de "escanear" es consultar a quien ya lo hizo</mark> (ver [[08 - Detección de escaneos y evasión moderna]]):

- **Shodan / Censys / ZoomEye** -> puertos, servicios, versiones y certificados ya indexados de cualquier IP pública.
- **`uncover`** (*ProjectDiscovery*) -> consulta *Shodan/Censys/FOFA* por *API* desde la terminal y devuelve hosts:

```shell-session
$ uncover -q 'org:"Inlanefreight"' -e shodan,censys -o passive.txt
```

# Ecosistema Nmap y automatización

- `vulners` / `vulscan` ([[04 - Nmap Scripting Engine (NSE)|NSE]]) -> cruce [[04 - CVE|CVE]] por CPE contra bases externas (ver [[04 - Nmap Scripting Engine (NSE)]]).
- `AutoRecon` / `nmapAutomator` -> <mark style="background: #ADCCFFA6;">orquestan la enumeración completa</mark> (descubrimiento → puertos → scripts por servicio) y guardan todo ordenado. <mark style="background: #FFB8EBA6;">Ahorran horas</mark> en cajas con muchos servicios.
- `nmap-parse-output` -> <mark style="background: #ADCCFFA6;">convierte el XML en CSV, lista de puertos o HTML moderno</mark> (mejor que `grep` sobre `.gnmap`, ver [[06 - Guardar y explotar resultados]]).

# Identificación web y vuln scanning

<mark style="background: #ADCCFFA6;">Para servicios HTTP</mark>, más rápidos y actuales que `-sV`:

- **`httpx`** — <mark style="background: #ADCCFFA6;">sondea hosts/puertos web</mark>, saca título, tecnología, status, TLS.
- **`whatweb`** — *fingerprinting* de <mark style="background: #ADCCFFA6;">tecnologías web</mark>.
- **`nuclei`** — <mark style="background: #ADCCFFA6;">motor de plantillas comunitarias</mark> (actualizadas a diario) que <mark style="background: #FFB8EBA6;">ha desplazado en la práctica a los scripts NSE</mark> `vuln`.

```shell-session
$ naabu -host target -silent | httpx -silent -tech-detect | nuclei -severity critical,high
```

# Registro e integración

- **`db_import target.xml`** en [[Metasploit.base|Metasploit]] puebla `hosts`/`services` para todo el engagement.
- El XML de Nmap alimenta Faraday, DefectDojo o dashboards propios para el [[Documentación y reporting.base|informe]].

> [!important]+ Workflow de referencia
> Rangos grandes: `masscan`/`naabu` (descubrir) → `nmap -sCV -p<abiertos>` (enumerar) → `httpx`/`nuclei` (web) → `db_import` (registrar). Un solo host: `rustscan -a host -- -sCV` y a enumerar. Recon previo pasivo con `uncover`/Shodan para llegar sabiendo qué buscar y minimizar el ruido.

> [!info]+ Fuentes
> - netalith — [Nmap vs Masscan vs RustScan (2026)](https://netalith.com/blogs/cybersecurity/nmap-vs-masscan-vs-rustscan-2026-comparison); cyberleveling — [comparativa](https://cyberleveling.com/blog/nmap-vs-rustscan-vs-masscan); s0cm0nkey — [Port Scanner Shootout](https://s0cm0nkey.gitbook.io/port-scanner-shootout/).
> - ProjectDiscovery (naabu, httpx, nuclei, uncover) — [docs.projectdiscovery.io](https://docs.projectdiscovery.io/).
