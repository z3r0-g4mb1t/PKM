---
tags:
  - OSINT/Contra-OSINT
  - Tipo/Defensa
Descripción: "Cómo reducir lo que el OSINT encuentra: opt-outs de data brokers y DELETE Act, derecho de supresión RGPD, y hardening corporativo de huella"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - Contra-OSINT - por qué casi no se detecta y qué hacer]]"
Nota siguiente: "[[02 - Detección y deception contra OSINT]]"
Area: "[[Contra-OSINT.base|Contra-OSINT]]"
---
---

La defensa de raíz: si no hay dato expuesto, no hay OSINT. Reducir huella es tedioso y nunca total (lo público reaparece), pero recorta mucho lo que un atacante encuentra. Dos planos: el personal y el corporativo.

# Huella personal

## Data brokers: opt-outs

<mark style="background: #ADCCFFA6;">Los *data brokers* ([[04 - Buscadores de personas y registros públicos|people search]]) son la mayor fuente de exposición personal</mark>, y se pueden limitar:

- **Opt-out manual**, broker a broker. Eficaz pero agotador: <mark style="background: #FFB8EBA6;">hay cientos de brokers, y si te borras en uno, te reponen cuando te encuentran en otro</mark> — hay que repetirlo.
- **Servicios de borrado** (DeleteMe, Incogni): automatizan los opt-outs de forma continua, por suscripción. Compran tiempo a cambio de dinero.
- **Derechos legales**: en la UE, el **derecho de supresión** del [[04 - Marco legal y ético del OSINT|RGPD (art. 17)]] obliga a borrar tus datos a quien los trate. En California, la **DELETE Act (SB 362)** creó **DROP**, un portal **único** de borrado: operativo para consumidores desde el 1 de enero de 2026, y desde el 1 de agosto de 2026 los brokers (545 registrados) deben consultarlo cada 45 días y borrar en ese plazo. Es el primer "borra en todos a la vez" — California por ahora.

## Configuración y disciplina

- **Privacidad en RRSS**: perfiles no públicos, listas de amigos/seguidores ocultas, ubicación desactivada.
- **Metadatos**: quita el GPS de las fotos que publiques ([[01 - Metadatos y EXIF|strip con exiftool/MAT2]]).
- **Reutilización**: no reuses el mismo username/avatar/correo entre contextos que no quieras vincular — es lo que explota la [[01 - Enumeración de usernames cross-platform|enumeración de usernames]].
- **Contraseñas únicas + MFA resistente a phishing**: rompe la premisa de las [[03 - Credential reuse, combolists y password spraying|brechas]].

# Huella corporativa

Para una organización, reducir huella es gobernar lo que expone sin querer:

- <mark style="background: #FFB86CA6;">**Arreglar DMARC/SPF**</mark>: pasar de `p=none` a `p=reject` para que el dominio **no sea** [[02 - Tech stack y activos digitales#La joya: spoofability por SPF y DMARC|spoofable]] en phishing. De los cambios de más impacto y menos coste.
- **Quitar lo indexado sensible**: configs, backups, documentos internos fuera del índice ([[05 - Defensa y detección|noindex + removals]], no `robots.txt`).
- **Controlar los metadatos de documentos públicos**: limpiar autor, rutas y software de los PDF/Office antes de publicarlos ([[01 - Metadatos y EXIF#Metadatos en documentos|FOCA/exiftool]] es lo que el atacante usará).
- **Moderar las ofertas de empleo**: no detallar el stack exacto ("experiencia con [EDR concreto], [versión]") — es un [[03 - Enumeración de personal#Las ofertas de empleo son un regalo|inventario firmado]] para el atacante.
- **Concienciación de empleados**: lo que publican en LinkedIn (tech stack, proyectos), las fotos de oficina (pantallas, badges), el correo corporativo en registros públicos. La huella de la empresa es la suma de la de su gente.
- **Reducir activos huérfanos**: dominios caducados, subdominios de *staging*, [[03 - Cadena de suministro, filiales y M&A|filiales olvidadas]] — inventariarlos y retirarlos.

> [!important]+ La reducción nunca es total
> <mark style="background: #FF5582A6;">Lo público reaparece</mark>: los brokers reponen datos, los archivos conservan copias ([[13 - Web Archives|Wayback]]), lo filtrado en una brecha ya no se "desfiltra". La reducción de huella **recorta y encarece** el OSINT del atacante, no lo elimina. Por eso se combina con la [[02 - Detección y deception contra OSINT|deception y la monitorización]].

> [!info]+ Fuentes
> California DELETE Act (SB 362) y portal DROP: [BCLP, *The DELETE Act*](https://www.bclplaw.com/en-US/events-insights-news/the-delete-act-a-first-of-its-kind-data-broker-law.html); [DataGrail, *DELETE Act and DROP*](https://www.datagrail.io/blog/regulations/the-delete-act-and-drop-what-you-need-to-know/). RGPD art. 17 (derecho de supresión). Servicios de borrado: DeleteMe, Incogni.
