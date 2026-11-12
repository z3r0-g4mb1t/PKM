---
tags:
  - OSINT/Fundamentos
  - Pentesting/Enumeracion
Descripción: "El mapa de disciplinas OSINT (SOCMINT, GEOINT, IMINT…), los tres niveles de accesibilidad de una fuente y el sistema Admiralty de fiabilidad"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - Pasivo, semipasivo y activo - el gradiente de huella]]"
Nota siguiente: "[[04 - Marco legal y ético del OSINT]]"
Area: "[[Fundamentos OSINT.base|Fundamentos OSINT]]"
---
---

OSINT no es una técnica, es un **método aplicado a muchos tipos de fuente**. Para no perderse conviene un mapa en dos ejes: **qué disciplina** (según el tipo de fuente) y **qué nivel de accesibilidad** tiene esa fuente. Y, cruzándolo todo, un sistema para calificar **cuánto te puedes fiar** de lo que encuentres.

# Disciplinas: el tipo de fuente

En la jerga de inteligencia, cada "INT" es una disciplina según el origen del dato (`HUMINT` humana, `SIGINT` de señales…). <mark style="background: #ADCCFFA6;">El OSINT es la que usa fuentes abiertas, y dentro de ella hay sub-disciplinas según el tipo de material</mark>. Son las dimensiones que estructuran esta área:

| Disciplina | Qué fuente explota | Sub-área del vault |
| - | - | - |
| **Personas / identidades** | Nombres, alias, usernames, correos, teléfonos | [[OSINT de personas.base\|Personas]] |
| **`SOCMINT`** | Redes sociales (perfiles, posts, relaciones) | [[SOCMINT.base\|SOCMINT]] |
| **Corporativo** | Registros mercantiles, finanzas, empleados | [[OSINT corporativo.base\|Empresas]] |
| **Infraestructura** | Dominios, IPs, certificados, servicios | [[OSINT de infraestructura.base\|Infraestructura]] |
| **Brechas / credenciales** | Filtraciones, *stealer logs*, secretos | [[Brechas y filtraciones.base\|Brechas]] |
| **`GEOINT` / `IMINT`** | Geolocalización, imágenes, vídeo, metadatos | [[GEOINT e imágenes.base\|GEOINT]] |
| **Dark web** | Mercados, foros, *pastes* | [[Dark web OSINT.base\|Dark web]] |

Un investigador real **salta entre disciplinas**: de un username (personas) llega a una cuenta de GitHub (infraestructura/código), de ahí a un correo (personas), y ese correo lo busca en brechas (credenciales). <mark style="background: #8000E1A6;">El valor está en el *pivote* de una disciplina a otra</mark> — de ahí que el OSINT de personas sea el pegamento de casi todo.

# Accesibilidad: cómo de "abierta" es la fuente

"Fuente abierta" tiene matices legales y prácticos. Tres niveles:

- <mark style="background: #ADCCFFA6;">**Abierta / pública**</mark>: indexada y accesible sin barrera — la web indexada, registros públicos, un post público de RRSS, los `CT logs`. Es OSINT en estado puro.
- <mark style="background: #FFB8EBA6;">**Semiabierta / *gated***</mark>: pública pero tras una barrera — registro gratuito, API con clave, rate-limiting, o un muro de pago legítimo (registros judiciales de pago, informes mercantiles). Sigue siendo lícita, pero la barrera (y sus `ToS`) condiciona cómo accedes.
- <mark style="background: #FFB86CA6;">**Cerrada / filtrada**</mark>: no debería ser pública pero lo es por una brecha — bases de credenciales filtradas, *stealer logs*, documentos robados en foros. Aquí hay un debate: los puristas del OSINT **excluyen** la *breach data* de la definición porque el dato fue sustraído, no publicado. El red teamer la usa igualmente porque el atacante real la usa, pero con todas las salvaguardas legales de [[04 - Marco legal y ético del OSINT]]. <mark style="background: #FF5582A6;">Tratar *breach data* como si fuera "abierta" es el error ético y legal más grave del OSINT ofensivo</mark>.

# Fiabilidad: el sistema Admiralty

No todo lo que encuentras es cierto. El estándar para calificar un hallazgo es el **código Admiralty** (o "sistema NATO", `STANAG 2511`), que puntúa por separado dos cosas con un par letra+número:

- **Fiabilidad de la *fuente*** (A–F): `A` completamente fiable … `E` poco fiable … `F` no evaluable.
- **Credibilidad de la *información*** (1–6): `1` confirmada por otras fuentes … `3` posiblemente cierta … `6` no evaluable.

Un dato se anota entonces como `B2`, `C3`, etc. Ejemplos aplicados al OSINT:

- Un registro `WHOIS` oficial de un `TLD` serio → fuente **A**, y si lo corrobora el DNS, información **1** → `A1`.
- Un post anónimo en un foro afirmando que ACME usa cierto EDR → fuente **E/F**, información **5** → `E5`: una **pista**, no un hecho. Hay que corroborarla antes de actuar.
- Una oferta de empleo de ACME pidiendo "experiencia con CrowdStrike Falcon" → fuente **B** (la propia empresa, aunque puede estar desactualizada), información **2** → `B2`: muy aprovechable.

<mark style="background: #8000E1A6;">La regla operativa: un hallazgo de una sola fuente no corroborada es una hipótesis, no inteligencia</mark>. Antes de construir un ataque sobre él, confírmalo con una segunda fuente independiente. El detalle de verificación, corroboración y los sesgos que lo sabotean, en [[05 - Verificación, fiabilidad de fuentes y sesgos]].

> [!info]+ Fuentes
> Disciplinas de inteligencia y definición de OSINT: `ATP 2-22.9` (US Army). Sistema de calificación de fuentes: `STANAG 2511` (NATO), conocido como *Admiralty Code*. El debate sobre si la *breach data* es OSINT "legítimo" es recurrente en la comunidad (Bazzell, *OSINT Techniques*, y la escuela IntelTechniques).
