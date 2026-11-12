---
tags:
  - OSINT
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Qué es realmente OSINT, su lugar en el engagement y por qué es la ventaja que los red teamers junior infravaloran"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - El ciclo de inteligencia aplicado al OSINT]]"
Area: "[[Fundamentos OSINT.base|Fundamentos OSINT]]"
---
---

<mark style="background: #ADCCFFA6;">`OSINT` (*Open-Source Intelligence*) es la inteligencia producida a partir de información de fuentes públicamente disponibles, recopilada, evaluada y difundida para responder a un requisito concreto</mark>. La definición doctrinal estadounidense (recogida en la *National Defense Authorization Act* de 2006 y en el manual del ejército `ATP 2-22.9 Open-Source Intelligence`) insiste en ese final: **para responder a un requisito**. No es acumular datos, es producir inteligencia.

Esa palabra —*intelligence*— es la que separa el OSINT del "buscar en Google". <mark style="background: #8000E1A6;">Un dato público no es inteligencia; se convierte en inteligencia cuando se recopila con un propósito, se verifica, se cruza con otros datos y se entrega en forma accionable</mark>. El perfil de LinkedIn de un administrador de sistemas es un dato. Saber que *esa* persona administra el Exchange de la empresa, deducir su correo corporativo por el patrón de la organización, y encontrar su contraseña en una filtración de 2023 que probablemente reutilice: eso es inteligencia.

# Por qué los junior lo infravaloran

El operador junior tiende a saltar directo al escaneo y la explotación: le resulta más "técnico" y da gratificación inmediata (un puerto abierto, un servicio vulnerable). El OSINT parece lento y poco glamuroso. Es un error de juicio con consecuencias directas sobre el resultado del engagement:

- <mark style="background: #FFB86CA6;">El vector de acceso inicial más rentable en 2026 rara vez es un 0-day</mark>. Es una credencial válida sacada de una filtración o un *stealer log* combinada con *password spraying*, o un *phishing* creíble construido sobre inteligencia de personas. Todo eso nace del OSINT, no del escáner.
- Cuanto mejor es el mapa de la superficie de ataque, más estrecho y sigiloso puede ser el ruido activo posterior. <mark style="background: #FFB8EBA6;">El OSINT bien hecho reduce la cantidad de escaneo activo que necesitas</mark> —y por tanto tu exposición a la detección—.
- En *bug bounty*, el activo que el objetivo expuso por descuido y olvidó (un subdominio de *staging*, un repo con secretos, un bucket abierto) es un punto de entrada que el resto de cazadores aún no ha mirado. Encontrarlo es puro OSINT.

# OSINT como fase del engagement

En cualquier modelo de ataque, el OSINT es la **primera fase** y la que condiciona todas las demás:

- En la metodología del pentest es la fase de `Information Gathering` / `Recopilación de información` → [[05 - Recopilación de información]].
- En MITRE ATT&CK es la táctica **`Reconnaissance` (TA0043)** del pre-ataque: técnicas como *Gather Victim Identity Information*, *Search Open Technical Databases* o *Search Open Websites/Domains* son literalmente OSINT catalogado.
- En la *Cyber Kill Chain* de Lockheed Martin es el paso 1, *Reconnaissance*.

<mark style="background: #ADCCFFA6;">La característica que define el OSINT frente al resto del recon es que, bien ejecutado, **no toca al objetivo**</mark>: la información se obtiene de terceros (buscadores, registros públicos, redes sociales, brechas) que ya la tienen indexada. Eso lo hace sigiloso y, en la fase pasiva, ejecutable incluso antes de que el alcance activo esté autorizado — con los matices legales de [[04 - Marco legal y ético del OSINT|el marco legal]].

> [!important]+ El OSINT no sustituye al recon técnico: lo precede y lo potencia
> El vault ya cubre el recon técnico de web e infraestructura en [[00 - Reconocimiento web|Reconocimiento Web]] y [[00 - Principios y metodología de enumeración|Footprinting]]. Esta área **no lo reexplica**: se ocupa de lo que esos temas no tocan —metodología de inteligencia, personas, redes sociales, filtraciones, geolocalización, imágenes, dark web— y cross-linkea al recon técnico cuando el flujo lo pide. El OSINT de personal que alimenta los ataques de acceso ya empezó en [[03 - Enumeración de personal]]; aquí se desarrolla a fondo.

# Qué dimensiones cubre esta área

El OSINT profesional se organiza por **disciplinas** según el tipo de fuente, no por herramienta. Las que dan ventaja a un red teamer y que esta área desarrolla:

- **Personas e identidades** — nombres, alias, usernames cross-platform, correos, teléfonos.
- **`SOCMINT`** — inteligencia de redes sociales (relaciones, actividad, metadatos).
- **Empresas y organizaciones** — estructura, finanzas, empleados, *tech stack*.
- **Infraestructura** — dominios, IPs, certificados, servicios expuestos (puente al recon técnico).
- **Brechas y credenciales** — filtraciones, *stealer logs*, secretos en repos.
- **`GEOINT` / `IMINT`** — geolocalización, imágenes, metadatos.
- **Dark web, pastes y foros** — mercados, filtraciones, chatter.

La taxonomía completa, con el sistema de fiabilidad de fuentes, está en [[03 - Taxonomía de fuentes y disciplinas OSINT]]. Antes conviene entender **cómo** se trabaja: el ciclo de inteligencia.

> [!info]+ Fuentes
> Definición doctrinal: `ATP 2-22.9 Open-Source Intelligence` (US Army) y `JP 2-0 Joint Intelligence`. Catálogo de técnicas OSINT como *Reconnaissance*: [MITRE ATT&CK TA0043](https://attack.mitre.org/tactics/TA0043/). Modelo de kill chain: Lockheed Martin *Cyber Kill Chain*.
