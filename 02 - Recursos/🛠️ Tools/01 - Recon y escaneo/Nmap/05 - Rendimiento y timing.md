---
tags:
  - Pentesting/Enumeracion
  - Escaneo/Redes
  - Linux
Descripción: "El rendimiento importa cuando escaneas una red grande o con poco ancho de banda"
Fecha de actualización: 2026-11-12
Nota previa: "[[04 - Nmap Scripting Engine (NSE)]]"
Nota siguiente: "[[06 - Guardar y explotar resultados]]"
Area: "[[Nmap.base|Nmap]]"
---
El **rendimiento importa cuando escaneas una red grande o con poco ancho de banda**. Nmap deja controlar cada palanca del tiempo: <mark style="background: #ADCCFFA6;">agresividad global</mark> (`-T <0-5>`), <mark style="background: #ADCCFFA6;">paralelismo</mark> (`--min-parallelism`), <mark style="background: #ADCCFFA6;">timeouts</mark> (`--max-rtt-timeout`), <mark style="background: #ADCCFFA6;">tasa de paquetes</mark> (`--min-rate`) y <mark style="background: #ADCCFFA6;">reintentos</mark> (`--max-retries`). <mark style="background: #FF5582A6;">Todo ajuste de velocidad es un intercambio: más rápido casi siempre significa menos preciso o más ruidoso</mark>.

# Timeouts (RTT)

Cada paquete tarda un *Round-Trip-Time* en volver. Nmap arranca con un *timeout* inicial alto (**1000 ms** por defecto, `--initial-rtt-timeout`) y lo adapta, con un suelo de 100 ms (`--min-rtt-timeout`) y un techo de 10 s (`--max-rtt-timeout`):

```shell-session
$ sudo nmap 10.129.2.0/24 -F                                          # 256 hosts, top 100
... scanned in 39.44 seconds  (10 hosts up)

$ sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
... scanned in 12.29 seconds  (8 hosts up)
```

> [!warning]+ Acelerar tiene coste
> El escaneo optimizado tardó **un cuarto** del tiempo… pero encontró **2 hosts menos**. Un `--initial-rtt-timeout` demasiado corto hace que hosts lentos (o con latencia real) no lleguen a responder a tiempo y se den por muertos. En redes reales con latencia variable, <mark style="background: #ADCCFFA6;">timeouts agresivos = falsos negativos</mark>.

# Reintentos (`--max-retries`)

Si un puerto no responde, Nmap reintenta hasta `--max-retries` veces (**10 por defecto**). Bajarlo a `0` significa "un intento y a otra cosa":

```shell-session
$ sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l            # 23 puertos
$ sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l   # 21 puertos
```

Otra vez el mismo patrón: <mark style="background: #FFB8EBA6;">acelerar puede hacerte perder información</mark> (2 puertos que solo respondieron al reintento).

# Tasa de paquetes (`--min-rate`)

Si conoces el ancho de banda (típico en una caja blanca donde te han *whitelisteado*), `--min-rate` fuerza un mínimo de paquetes por segundo. <mark style="background: #FF5582A6;">Aquí sí ganas tiempo sin perder resultados</mark>:

```shell-session
$ sudo nmap 10.129.2.0/24 -F -oN tnet.default            # scanned in 29.83s
$ sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300   # scanned in 8.67s
```

Ambos encontraron los mismos 23 puertos, pero el segundo tardó menos de un tercio. <mark style="background: #FFB8EBA6;">`--min-rate` es hoy la palanca de velocidad preferida en escaneos autorizados</mark>: predecible y sin sacrificar cobertura si el enlace aguanta.

# Plantillas de timing (`-T 0-5`)

Cuando no puedes calcular los valores a mano (caja negra), Nmap trae seis perfiles preconfigurados. El desarrollador ya eligió los `--min-rtt`, paralelismo y delays de cada uno:

| **Plantilla** | **Nombre** | **Uso**                                                      |
| ------------- | ---------- | ------------------------------------------------------------ |
| `-T0`         | paranoid   | Evasión de IDS extrema: serializa sondas, delays de minutos. |
| `-T1`         | sneaky     | Evasión: lento, sondas espaciadas.                           |
| `-T2`         | polite     | Reduce carga sobre el objetivo.                              |
| `-T3`         | normal     | **Por defecto**.                                             |
| `-T4`         | aggressive | El *de facto* en pentest autorizado: rápido y razonable.     |
| `-T5`         | insane     | Máxima velocidad; pierde precisión y puede tumbar hosts.     |

Los valores concretos que mueve cada plantilla (los más relevantes, [doc oficial](https://nmap.org/book/man-performance.html)):

| Plantilla | Delay entre sondas | initial-RTT | max-RTT | max-retries | Extra |
| --------- | ------------------ | ----------- | ------- | ----------- | ----- |
| `-T0`     | **5 min** (serializado, 1 puerto a la vez) | — | — | — | — |
| `-T1`     | **15 s**           | —           | —       | —           | — |
| `-T2`     | **0,4 s**          | —           | —       | —           | — |
| `-T3`     | — (por defecto)    | 1000 ms     | 10 s    | 10          | — |
| `-T4`     | tope TCP 10 ms     | 500 ms      | 1250 ms | 6           | — |
| `-T5`     | tope TCP 5 ms      | 250 ms      | 300 ms  | 2           | host-timeout 15 min · script-timeout 10 min |

<mark style="background: #ADCCFFA6;">`-T0`/`-T1` juegan con el **espaciado temporal**</mark> (5 min / 15 s entre sondas) para diluir el patrón de *portscan*; <mark style="background: #FFB86CA6;">`-T4`/`-T5` recortan *timeouts* y reintentos</mark> (de 10 s/10 a 1250 ms/6 y a 300 ms/2), por eso van rápido pero dan por muertos los hosts lentos.

```shell-session
$ sudo nmap 10.129.2.0/24 -F              # -T3 por defecto: 32.44s
$ sudo nmap 10.129.2.0/24 -F -T 5         # insane: 18.07s
```

> [!important]+ El triángulo velocidad / precisión / sigilo
> <mark style="background: #FF5582A6;">No se optimizan las tres a la vez</mark>. **`-T4` + `--min-rate`** para <mark style="background: #FFB86CA6;">pentest con permiso</mark> (rápido y fiable). **`-T0`/`-T1`** cuando <mark style="background: #FFB86CA6;">lo que importa es no despertar al IDS</mark> (ver [[07 - Evasión de firewalls, IDS e IPS]] y [[08 - Detección de escaneos y evasión moderna]]): los delays largos difuminan el patrón de *portscan*. Un `-T5` contra un servicio frágil puede provocar caídas — y un `-T4/-T5` es una <mark style="background: #FFB86CA6;">firma clarísima para cualquier IPS</mark>.

# Enfoque profesional 2026

- **`--host-timeout <t>`**: <mark style="background: #ADCCFFA6;">abandona hosts que tardan demasiado</mark> (útil en `/16` con máquinas muertas o filtradas que ralentizan todo).
- **Descubrir rápido, enumerar despacio**: para rangos enormes, el `-p-` de Nmap no compite con `masscan`/`RustScan` a `--rate` alto; Nmap entra después con `-sCV` dirigido y timing moderado. Ver [[09 - Arsenal de herramientas de escaneo]].
- **No confundas velocidad con sigilo**: `-T4` es rápido **y** ruidoso. Sigilo real = `-T1`/`-T0` + probes mínimas, no `-T5` con menos retries.
- **Resolución DNS en paralelo (builds 7.96+)**: la resolución directa de nombres ahora corre **en paralelo** (antes era secuencial y un cuello de botella con listas grandes de hostnames), y desde `7.97` el *resolver* consulta `/etc/hosts` y trata bien `localhost`. Si escaneas por IP, `-n` sigue siendo lo más rápido — se salta el DNS inverso del todo. (Fuente: [nmap.org/changelog](https://nmap.org/changelog.html).)

Valores exactos de cada plantilla: [nmap.org/book/performance-timing-templates.html](https://nmap.org/book/performance-timing-templates.html).
