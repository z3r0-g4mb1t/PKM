---
tags:
  - OSINT/Dark-Web
  - Pentesting/Enumeracion
Descripción: "El uso más rentable de la dark web: monitorizar la exposición del cliente de forma continua, con qué plataformas y qué reportar"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - Buscar en la dark web - motores y directorios]]"
Nota siguiente:
Area: "[[Dark web OSINT.base|Dark web OSINT]]"
---
---

La búsqueda puntual en la dark web da una foto; la **monitorización** da una película. Para un cliente, lo valioso no es "¿aparecía ACME el día que miré?" sino "avísame en cuanto ACME aparezca". Es el solapamiento del OSINT ofensivo con la *Cyber Threat Intelligence* (CTI).

# Qué monitorizar

<mark style="background: #ADCCFFA6;">Defines **selectores** del cliente y vigilas su aparición</mark> en leaks, pastes, foros y leak sites:

- **Dominios y correos** (`@acme.com`) → credenciales filtradas, [[02 - Stealer logs e infostealers|stealer logs]].
- **Nombre y marcas** → menciones en foros, datos a la venta, suplantación.
- **Directivos y VIP** → *whaling*, extorsión, exposición personal.
- **Rangos IP / ASN** ([[02 - ASN, BGP y mapeo de rangos IP|infra]]) → accesos a la venta.
- **Código fuente / nombres de repos internos** → fugas de propiedad intelectual.

<mark style="background: #FFB86CA6;">La aparición de cualquiera de estos es un hallazgo accionable</mark>: desde "rota estas credenciales" hasta "activa el plan de respuesta a incidentes" si es un leak site de ransomware.

# Con qué monitorizar

- **Intelligence X**: indexa leaks, pastes, contenido onion e histórico; permite alertas por selector. Buena base.
- **Solapamiento con brechas**: [[01 - Buscadores de brechas - HIBP, Dehashed e IntelX|Hudson Rock, HIBP, Dehashed]] cubren la parte de credenciales/stealer logs — esta sub-área y la de brechas se tocan aquí.
- **Plataformas CTI comerciales** (Flare, DarkOwl, KELA, Recorded Future): acceden a **foros cerrados** que un investigador individual no alcanza, mantienen personas infiltradas y dan alertas y contexto. Es la vía para cobertura seria y continua — de pago y orientada a empresa.
- **DIY**: scripts que consultan [[02 - Buscar en la dark web - motores y directorios|Ahmia/OnionSearch]] y sitios de pastes periódicamente por tus selectores, con alertas. Barato, pero no llega a los foros cerrados.

# El entregable

Para el red teamer, la monitorización produce dos cosas:

1. <mark style="background: #FF5582A6;">**Hallazgos de informe** de alto impacto</mark>: "340 credenciales de ACME en stealer logs recientes", "ACME listada en el leak site de [grupo]", "acceso VPN a ACME a la venta en un foro". Son de los hallazgos que más mueven la aguja en un informe ejecutivo.
2. **Vectores de acceso** (en alcance): credenciales vivas, accesos, contexto para phishing.

# El ángulo defensivo

Esta nota es también el puente al Blue Team: la monitorización de dark web es una capacidad **defensiva** estándar (la hace un SOC o un servicio de *Digital Risk Protection*). <mark style="background: #8000E1A6;">El red teamer que entiende cómo se monitoriza sabe también qué deja rastro y cómo lo detecta el defensor</mark> — lo que conecta con [[Contra-OSINT.base|contra-OSINT]].

> [!warning]+ Monitorizar, no coleccionar
> Monitorizar la exposición del cliente es legítimo y de alto valor. Pero sigue aplicando el límite: si aparece una base de datos robada del cliente, **se reporta y se escala**; no se descarga "para analizarla" sin cobertura legal explícita, porque es dato robado ([[04 - Marco legal y ético del OSINT|legal]]). La CTI seria tiene procesos para manejar esto; improvisar es arriesgado.

> [!info]+ Fuentes
> Monitorización de leaks/pastes/onion: [Intelligence X](https://intelx.io). Consolidado de ransomware: [ransomware.live](https://www.ransomware.live). Plataformas CTI comerciales (Flare, DarkOwl, KELA, Recorded Future): categoría de *Digital Risk Protection* / *Dark Web Monitoring*.
