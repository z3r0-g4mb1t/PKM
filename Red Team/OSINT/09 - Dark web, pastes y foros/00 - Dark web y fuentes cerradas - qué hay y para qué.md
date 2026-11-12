---
tags:
  - OSINT/Dark-Web
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Qué contiene la dark web que importa al red team y la CTI (leaks, mercados, chatter, leak sites de ransomware) y por qué no siempre necesitas Tor"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Acceso y OPSEC en Tor]]"
Area: "[[Dark web OSINT.base|Dark web OSINT]]"
---
---

La dark web tiene fama de inaccesible y peligrosa; parte de eso es cierto, pero para el OSINT la realidad práctica es más matizada. <mark style="background: #ADCCFFA6;">La "dark web" son servicios ocultos accesibles solo por redes de anonimato (`.onion` sobre Tor, `.i2p` sobre I2P)</mark>, y lo relevante para un engagement es un subconjunto concreto.

# Qué hay que importe (y qué no)

Para red team y *threat intelligence*, lo valioso:

- <mark style="background: #FFB86CA6;">**Datos filtrados del objetivo**</mark>: credenciales, documentos, bases de datos a la venta o publicadas.
- <mark style="background: #FFB86CA6;">**Leak sites de ransomware**</mark>: los grupos modernos mantienen sitios `.onion` donde **nombran a sus víctimas** y publican muestras de datos robados. Si el cliente aparece ahí, es una emergencia.
- **Chatter de actores**: foros donde se discuten accesos, exploits y objetivos. Un *initial access broker* vendiendo acceso a "una empresa española de logística" puede ser tu cliente.
- **Mercados**: de credenciales, [[02 - Stealer logs e infostealers|stealer logs]], accesos.

Lo que **no** es trabajo del pentester: comprar, interactuar, ni acercarse a contenido ilegal. El OSINT aquí es **observacional y de inteligencia**, no transaccional.

# No siempre necesitas Tor

<mark style="background: #8000E1A6;">Buena parte de este OSINT se hace desde la clearnet</mark>, a través de pasarelas que indexan y monitorizan los servicios ocultos por ti:

- **Ahmia** indexa `.onion` y es accesible por web normal.
- **Intelligence X** indexa leaks, pastes y contenido onion (tiers de pago).
- **ransomware.live** consolida las víctimas de los leak sites de ransomware, en clearnet.

Esto reduce mucho la necesidad de entrar en Tor y, con ella, el riesgo. Entras en Tor solo cuando necesitas ver una fuente que las pasarelas no cubren — y entonces con [[01 - Acceso y OPSEC en Tor|OPSEC estricto]].

# Señal vs ruido vs riesgo

La dark web es ruidosa, llena de estafas, clones y contenido perturbador o ilegal. <mark style="background: #FF5582A6;">El coste/beneficio rara vez justifica "explorar" a ciegas</mark>. El enfoque profesional es **dirigido**: buscas el nombre del cliente, sus dominios, sus correos; monitorizas apariciones; no navegas por curiosidad.

# El flujo de esta sub-área

1. **Acceso y OPSEC** si necesitas Tor → [[01 - Acceso y OPSEC en Tor]].
2. **Descubrir y buscar** fuentes (motores, directorios, leak sites) → [[02 - Buscar en la dark web - motores y directorios]].
3. **Monitorizar** la exposición del objetivo (el uso más rentable) → [[03 - Monitorización y el ángulo CTI]].

> [!danger]+ El límite legal y ético, muy en serio
> Observar un leak site o buscar el dominio del cliente en Ahmia es OSINT legítimo. **Descargar** datos robados, **comprar** accesos o credenciales, **interactuar** con vendedores, o acceder a contenido ilegal **no lo es** — y parte de ese contenido es delito con solo poseerlo. Mantente estrictamente en lo observacional, dentro del alcance autorizado, y si encuentras datos del cliente, **repórtalo**; no los descargues "para analizarlos" sin cobertura legal explícita. En caso de material claramente delictivo, se escala, no se recolecta.

> [!info]+ Fuentes
> Panorama y pasarelas clearnet: [Max Intel, *Dark Web OSINT Guide 2026*](https://maxintel.org/dark-web-osint-guide-2026.html); [Web Asha, *21 Dark Web Resources*](https://www.webasha.com/blog/top-21-dark-web-resources-every-osint-professional-should-know). Leak sites: [ransomware.live](https://www.ransomware.live).
