---
tags:
  - OSINT/Brechas
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Por qué las credenciales filtradas son un vector de acceso central en 2026, los tres tipos de fuente y dónde está la frontera legal del dato robado"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Buscadores de brechas - HIBP, Dehashed e IntelX]]"
Area: "[[Brechas y filtraciones.base|Brechas y filtraciones]]"
---
---

Las credenciales filtradas son una de las vías de acceso más fiables del red teamer moderno. No por volumen —que es brutal— sino porque <mark style="background: #FFB86CA6;">una credencial válida no necesita exploit: entras por la puerta</mark>. Esta sub-área es sensible: trata datos robados, y la frontera legal es estricta.

# Dónde ranquea en 2026 (con honestidad)

Conviene no exagerar. Los datos de la industria en 2026:

- <mark style="background: #FFB8EBA6;">El `DBIR 2026` sitúa el abuso de credenciales en el **13 %** como vector de acceso **inicial** (bajó), pero aparece en algún punto del **39 %** de las brechas</mark>. Mandiant *M-Trends 2026* pone los **exploits** como vector inicial nº1 (32 %).
- Flashpoint contó **1.700 millones** de credenciales robadas por *infostealers* solo en el primer semestre de 2026 (variantes dominantes: Vidar, StealC, Lumma). KELA habla de **2.860 millones** en circulación.

<mark style="background: #8000E1A6;">Conclusión matizada: no es el vector nº1, pero sí uno de los más **baratos, fiables y repetibles**</mark>. Un exploit requiere una vulnerabilidad sin parchear; una credencial válida + *spraying* funciona contra cualquiera que reutilice contraseñas y no tenga MFA. Para el red teamer, es de lo primero que se prueba.

# Tres tipos de fuente (no las confundas)

| Fuente | Qué es | Frescura / valor |
| - | - | - |
| **Brechas (*breaches*)** | Volcados de BBDD de un servicio hackeado (LinkedIn 2012, Collection #1…) | Antiguas; contraseñas a menudo hasheadas o caducadas |
| **Stealer logs** | Lo que un infostealer roba de **un equipo infectado** | <mark style="background: #FF5582A6;">Frescas, en claro, con cookies de sesión</mark> → lo más peligroso |
| **Combolists** | Listas `correo:contraseña` agregadas de varias fuentes | Mezcla; mucho ruido, útiles para *stuffing* |

<mark style="background: #ADCCFFA6;">La diferencia clave es la frescura</mark>: una brecha de 2012 da contraseñas probablemente ya cambiadas; un stealer log de hace dos semanas da credenciales **vivas** y, encima, **cookies de sesión que saltan el MFA** ([[02 - Stealer logs e infostealers|siguiente nota]]). Por eso los stealer logs han desplazado a las brechas clásicas como la fuente estrella.

# El flujo de esta sub-área

1. **Comprobar exposición y recuperar datos** con los buscadores de brechas → [[01 - Buscadores de brechas - HIBP, Dehashed e IntelX]].
2. **Entender los stealer logs** y por qué rompen el MFA → [[02 - Stealer logs e infostealers]].
3. **Convertir el dato en acceso**: reutilización, combolists, *spraying* vs *stuffing* → [[03 - Credential reuse, combolists y password spraying]].

> [!danger]+ La frontera legal del dato robado — léela antes de seguir
> <mark style="background: #FF5582A6;">**Consultar** si un correo aparece en una brecha es una cosa; **descargar** el dump, **crackear** los hashes y **probar** las contraseñas es otra muy distinta</mark>. La postura OSINT estricta: reportar la exposición, **no** usarla para loguearte. La postura red team: validar si la credencial da acceso **solo dentro del alcance activo y autorizado**. Usar datos robados fuera de un marco autorizado vulnera el [[04 - Marco legal y ético del OSINT|art. 197 CP y el RGPD]] (son datos personales sustraídos). Y **nunca** compres accesos a *initial access brokers* ni uses bots de "lookup" de [[04 - Telegram y Discord - las plataformas abiertas|Telegram]] de procedencia dudosa — eso ya es participar en el mercado criminal.

> [!info]+ Fuentes
> Cifras 2026: [Flashpoint vía Infosecurity Magazine](https://www.infosecurity-magazine.com/news/infostealers-17-billion/); DBIR 2026 y Mandiant *M-Trends 2026*. Catálogo ATT&CK: [T1589.001 *Credentials*](https://attack.mitre.org/techniques/T1589/001/). El vector de credenciales en el vault: [[Ataques a contraseñas.base|Ataques a contraseñas]].
