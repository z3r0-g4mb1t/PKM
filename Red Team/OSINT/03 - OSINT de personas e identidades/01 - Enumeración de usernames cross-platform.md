---
tags:
  - OSINT/Personas
  - Pentesting/Enumeracion
Descripción: "Encontrar todas las cuentas de un alias: cómo funciona el oráculo HTTP, Sherlock vs Maigret vs WhatsMyName y por qué todo hallazgo se verifica a mano"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - OSINT de personas - el pegamento de la investigación]]"
Nota siguiente: "[[02 - OSINT de correo electrónico]]"
Area: "[[OSINT de personas.base|OSINT de personas]]"
---
---

Dado un alias (`jdoe_1987`), quieres saber **en qué plataformas existe esa cuenta**. Reunir todas las cuentas de un username abre docenas de pivotes: cada perfil tiene bio, foto, enlaces y actividad que llevan al nombre real, al correo y a más alias.

# Cómo funciona: el oráculo HTTP

El mecanismo es simple y conviene entenderlo para no fiarse ciegamente de las herramientas. <mark style="background: #ADCCFFA6;">Para cada plataforma, la herramienta pide la URL del perfil (`https://github.com/jdoe_1987`) y deduce si existe según la respuesta</mark>:

- **Por código de estado**: `200` = existe, `404` = no existe. Rápido, pero frágil.
- **Por contenido**: busca una cadena concreta en el cuerpo ("user not found" vs el nombre del perfil).

<mark style="background: #FF5582A6;">El problema son los *soft-404*</mark>: muchas plataformas devuelven `200 OK` para un usuario inexistente (sirven una página genérica en vez de un `404`). Una herramienta que solo mira el estado marcará esa cuenta como **existente** cuando no lo está. De ahí nacen los falsos positivos, y por eso las herramientas buenas combinan estado **y** coincidencia de contenido/URL.

# Las herramientas, y cuándo usar cada una

| Herramienta | Enfoque | Sitios | Cuándo |
| - | - | - | - |
| **Sherlock** | Estado HTTP + string, rápido | ~400 | Primera pasada veloz |
| **Maigret** | Estado + cuerpo + URL, recursivo, informes | ~2.500-5.900 | Barrido exhaustivo |
| **WhatsMyName** | *Dataset* de cadenas de detección (no un escáner) | 600+ | La base de datos que usan otros |

## Sherlock — la primera pasada

```shell-session
# Un alias en cientos de sitios; imprime las URLs encontradas
$ sherlock jdoe_1987
# Varios alias de golpe
$ sherlock jdoe_1987 john.doe jdoe
```

Rápido pero con más falsos positivos (se apoya mucho en el estado). Úsalo para una foto inicial.

## Maigret — el barrido serio

Maigret es un fork de Sherlock (2021, de `soxoj`) que añade coincidencia por cuerpo y URL, **descubrimiento recursivo** (extrae de un perfil nuevos usernames y correos y los vuelve a buscar) y **parseo del perfil** (saca datos de cada cuenta encontrada):

```shell-session
# Barrido completo con reporte navegable en HTML
$ maigret jdoe_1987 --html
# Todos los sitios (incluidos los de poca fiabilidad) y salida JSON
$ maigret jdoe_1987 -a --json simple
# Extraer también los IDs/datos de cada perfil encontrado
$ maigret jdoe_1987 --extract
```

<mark style="background: #FFB86CA6;">El `--extract` y el descubrimiento recursivo son lo que convierte Maigret en una máquina de pivote</mark>: no solo te dice dónde existe el alias, sino que saca de cada perfil los datos para el siguiente salto.

## WhatsMyName — la base de datos

WhatsMyName no es un escáner, es el **dataset** comunitario de cadenas de detección (qué pedir y qué buscar en cada sitio) que mantiene la comunidad. Lo consumen una web (`whatsmyname.app`) y herramientas como **Blackbird**. Cuando una plataforma cambia y una herramienta falla, suele ser porque su dataset quedó desfasado — el de WhatsMyName es de los más actualizados.

# Verificar: el paso que nadie debe saltarse

<mark style="background: #FF5582A6;">Todo hallazgo de estas herramientas es una hipótesis hasta que lo verificas a mano</mark>. Los falsos positivos se disparan con:

- **Usernames comunes o cortos** (`alex`, `admin`, `john`): colisionan con cuentas de otras personas y con usuarios reservados. Un alias común puede dar 30-50 % de falsos positivos.
- **Sitios con *soft-404*** o con protección anti-bot que responde raro.

El protocolo de verificación:

1. **Abre cada cuenta "encontrada"** (con tu [[02 - Infraestructura de investigación - VM, red y navegador|navegador compartimentado]], no logueado con nada real).
2. **Corrobora que es la misma persona**: misma foto, misma bio, mismos enlaces, mismo correo. Dos `jdoe` con fotos distintas son dos personas.
3. **Pivota**: de las cuentas confirmadas saca nombre real, otros alias, correo, localización → vuelve a enumerar los alias nuevos.

> [!warning]+ OPSEC: estas herramientas salen desde tu IP
> Sherlock/Maigret lanzan cientos de peticiones a cientos de sitios **desde tu IP**. Es ruidoso y puede hacer que te marquen o limiten. Ejecútalas desde tu infraestructura de investigación (VM + VPN/proxy), nunca desde tu red real. Algunas plataformas registran la visita al perfil.

> [!info]+ Fuentes
> Mecánica de detección y comparación 2026: [Max Intel, *Username OSINT 2026*](https://maxintel.org/username-osint-guide-2026.html). Herramientas: [Sherlock](https://github.com/sherlock-project/sherlock), [Maigret](https://github.com/soxoj/maigret) (soxoj), [WhatsMyName](https://github.com/WebBreacher/WhatsMyName) (WebBreacher). Lista curada de *namecheckers*: [soxoj/osint-namecheckers-list](https://github.com/soxoj/osint-namecheckers-list).
