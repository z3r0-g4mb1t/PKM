---
tags:
  - OSINT/Personas
  - Pentesting/Enumeracion
Descripción: "Buscadores de personas, registros públicos y data brokers: qué dan, su sesgo geográfico y las fuentes equivalentes en España y la UE"
Fecha de actualización: 2026-11-08
Nota previa: "[[03 - OSINT de teléfono]]"
Nota siguiente: "[[05 - De un dato a la persona - pivote completo]]"
Area: "[[OSINT de personas.base|OSINT de personas]]"
---
---

Cuando tienes un **nombre** y quieres dirección, familiares, edad, teléfonos o historial, entran los buscadores de personas y los registros públicos. La gran advertencia: <mark style="background: #FF5582A6;">casi todo este ecosistema es **estadounidense**</mark>; para objetivos en España y la UE, el RGPD recorta a los *data brokers* y las fuentes son otras.

# Buscadores de personas (people search engines)

Agregan registros públicos en un perfil: direcciones, familiares, números, edad aproximada, antiguas ubicaciones.

| Servicio | Qué da | Nota |
| - | - | - |
| **TruePeopleSearch / FastPeopleSearch** | Dirección, teléfonos, familiares (EE. UU.) | Gratuitos |
| **Spokeo / BeenVerified** | Agregado amplio (EE. UU.) | De pago, afiliados |
| **Whitepages** | Teléfono y dirección (EE. UU.) | Freemium |
| **Pipl** | Agregador profundo | Ahora solo **B2B** (verificación de identidad) |

<mark style="background: #FFB8EBA6;">Útiles sobre todo para objetivos en EE. UU.</mark>; para Europa dan poco y a menudo datos obsoletos. Verifica siempre: estos agregadores mezclan a homónimos.

# España y la UE: las fuentes que sí sirven

El RGPD limita los *data brokers*, pero hay **registros oficiales** muy aprovechables:

- <mark style="background: #ADCCFFA6;">**BORME**</mark> (Boletín Oficial del Registro Mercantil): nombra a **administradores y apoderados** de empresas — conecta personas con sociedades. Clave para OSINT corporativo ([[OSINT corporativo.base|empresas]]).
- **Registro Mercantil** / **Catastro**: datos societarios y de propiedad (el Catastro, con acceso limitado a datos personales).
- **Boletines oficiales** (BOE, boletines autonómicos): nombramientos, subvenciones, oposiciones, sanciones — mucho nombre propio indexado.
- ⚠️ **El censo electoral NO es público en España** (a diferencia del *electoral roll* británico). No esperes un padrón consultable.

# Profesional, académico y legal

- **ORCID, Google Scholar, ResearchGate**: para objetivos académicos — publicaciones, afiliación, coautores.
- **Google Patents, Espacenet**: inventores y asignatarios — nombre + empresa + tecnología.
- **Registros judiciales**: en EE. UU., PACER (federal) y portales estatales; en España, acceso mucho más restringido.

# Familiares: genealogía

- **FamilySearch** (gratis) y **Ancestry**: árboles genealógicos que revelan **familiares**, apellidos de soltera, fechas — útil para respuestas de seguridad y para mapear el entorno de la persona.

# Data brokers y su reverso defensivo

Los *data brokers* (Acxiom, LexisNexis, Oracle Data Cloud y cientos más) compran y revenden perfiles. <mark style="background: #8000E1A6;">Su existencia es también la vía defensiva</mark>: reducir tu exposición personal pasa por los **opt-outs** de estos brokers — el reverso contra-OSINT se trata en [[Contra-OSINT.base|contra-OSINT]].

> [!warning]+ Sesgo, caducidad y verificación
> Estos servicios (a) están sesgados hacia EE. UU., (b) contienen datos **desactualizados** (direcciones de hace años), y (c) **mezclan homónimos** en un mismo perfil. Trátalos como fuente `C`: pistas a corroborar con una segunda fuente, nunca hechos. Y recuerda el marco [[04 - Marco legal y ético del OSINT|RGPD]]: perfilar personas en la UE exige base jurídica y minimización.

Con las piezas sueltas (username, correo, teléfono, nombre, registros), el valor está en **encadenarlas**. La siguiente nota lo hace de principio a fin.

> [!info]+ Fuentes
> Registros oficiales españoles: [BORME](https://www.boe.es/diario_borme/), Registro Mercantil, Sede del Catastro. People search de EE. UU. y su sesgo: guías OSINT 2026. Opt-out de data brokers: ver [[Contra-OSINT.base|contra-OSINT]].
