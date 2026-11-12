---
tags:
  - OSINT/SOCMINT
  - Pentesting/Enumeracion
Descripción: "La capa de análisis del SOCMINT: mapear la red social, leer la actividad temporal y correlacionar identidades entre plataformas"
Fecha de actualización: 2026-11-08
Nota previa: "[[04 - Telegram y Discord - las plataformas abiertas]]"
Nota siguiente:
Area: "[[SOCMINT.base|SOCMINT]]"
---
---

Recolectar perfiles es la mitad del SOCMINT; la otra mitad es **analizarlos**. El valor para el engagement no está en el perfil suelto, sino en la red, el patrón temporal y la correlación entre plataformas. Esta es la capa que convierte datos sociales en un plan de ataque.

# Análisis de red: quién con quién

<mark style="background: #ADCCFFA6;">Mapear las relaciones de un objetivo revela el eslabón por el que atacar</mark>. De seguidores, seguidos, amigos, menciones y comentarios sale:

- **El círculo real** (no todos los "contactos" lo son): con quién interactúa de verdad.
- **El eslabón débil**: a veces no atacas al CEO, atacas a su **asistente** o a un becario con acceso y menos concienciación.
- **Las relaciones de confianza**: a quién creería un phishing. Un correo "de" un compañero cercano pasa filtros mentales que uno de un desconocido no.

Para mapear esto a escala, el estándar es el **análisis de enlaces** con **Maltego** (grafos de entidades y relaciones; CE gratuito limitado) — se desarrolla en [[Frameworks OSINT.base|frameworks]].

# Análisis de actividad temporal

Las marcas de tiempo son inteligencia pura. <mark style="background: #FFB86CA6;">El patrón de publicación de una persona revela su zona horaria, su horario laboral y cuándo duerme</mark>:

- **Zona horaria y rutina**: si tuitea consistentemente entre las 8:00 y las 23:00 UTC+1 con un hueco a mediodía, tienes su huso y su jornada.
- **Ventanas de ausencia**: un patrón que se rompe (viaje, vacaciones) señala **cuándo el objetivo está fuera** — oportunidad para acceso físico o para un phishing "urgente" que no puede verificar en persona.
- **El mejor momento para el phishing**: lanzar el correo cuando la persona está activa pero ocupada (lunes por la mañana) sube la tasa de clic.

Herramientas como el histórico de X (`from:` + fechas) o el análisis de TGStat permiten reconstruir estos patrones.

# Correlación entre plataformas

<mark style="background: #8000E1A6;">La misma persona deja la misma huella en varias redes</mark>, y correlacionarlas confirma identidad y multiplica la inteligencia:

- **Mismo avatar / bio / username**: el pivote más directo ([[01 - Enumeración de usernames cross-platform|usernames]]).
- **Mismo estilo de escritura**: muletillas, errores, emojis — el *stylometry* vincula cuentas "anónimas" con las reales.
- **Mismos horarios**: dos cuentas que postean en la misma ventana horaria y callan a la vez probablemente son la misma persona.
- **Mismos enlaces/intereses**: comparten las mismas URLs, los mismos temas.

> [!warning]+ La correlación corta en los dos sentidos
> Exactamente lo que haces para vincular las cuentas de un objetivo es lo que un defensor (o un objetivo capaz) hará para **desenmascararte a ti**. Por eso el [[03 - Cómo no quemar la operación - fugas y errores comunes|OPSEC]] insiste en no reutilizar avatar, username, horario ni estilo entre tus sock puppets. Si tú correlacionas por estos ejes, asume que te correlacionan por los mismos.

# El producto: del análisis al ataque

<mark style="background: #FF5582A6;">El SOCMINT analizado produce entregables concretos para el engagement</mark>:

- **El pretexto de phishing**: construido sobre intereses reales, relaciones de confianza y eventos recientes.
- **El objetivo a suplantar**: el compañero cercano, el jefe, el de IT.
- **El momento**: cuando el objetivo está activo u ocupado, o fuera y sin poder verificar.
- **El eslabón humano**: la persona con acceso y menos defensa.

Esto enlaza directamente con la ingeniería social y, en el flujo del vault, con la [[03 - Enumeración de personal|munición de acceso]] y los [[Ataques a contraseñas.base|ataques de contraseña]]. El SOCMINT es, al final, inteligencia **para** el ataque humano.

> [!info]+ Fuentes
> Análisis de red con grafos: Maltego (ver [[Frameworks OSINT.base|frameworks]]). Análisis temporal y *stylometry* como técnicas de correlación: metodología OSINT estándar (Bazzell; think-pol). Aplicación ofensiva (pretexto, timing): precedente del vault en [[03 - Enumeración de personal]].
