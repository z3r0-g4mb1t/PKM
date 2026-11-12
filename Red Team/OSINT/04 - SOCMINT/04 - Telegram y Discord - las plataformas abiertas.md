---
tags:
  - OSINT/SOCMINT
  - Pentesting/Enumeracion
Descripción: "Telegram como la plataforma más abierta al OSINT: descubrimiento con dorks t.me y TGStat, análisis con Telethon, el ID inmutable, y Discord"
Fecha de actualización: 2026-11-08
Nota previa: "[[03 - Instagram, Facebook y TikTok]]"
Nota siguiente: "[[05 - Análisis de red, actividad y correlación]]"
Area: "[[SOCMINT.base|SOCMINT]]"
---
---

Mientras las grandes plataformas se cerraban, Telegram y Discord siguen relativamente **abiertas** — y Telegram es hoy la fuente SOCMINT más rica para *threat intel*, filtraciones y actores. También la más llena de contenido ilícito, lo que trae sus propias cautelas.

# Telegram: descubrimiento

La búsqueda nativa de posts es **Premium/de pago**, así que el descubrimiento gratuito va por fuera:

- <mark style="background: #ADCCFFA6;">**Dorks `site:t.me`** en Google</mark>: `site:t.me acme` encuentra canales y mensajes públicos indexados.
- **Directorios y estadísticas**: `TGStat` y `Telemetr.io` catalogan y analizan canales/grupos (crecimiento, actividad, temas); `Lyzem` busca en el contenido de canales públicos; `combot` da estadísticas de grupos.
- **Archivo**: Internet Archive mantiene búsqueda de texto completo sobre páginas web de Telegram archivadas.

# Telegram: análisis de canales y miembros

Para ir más allá de la lectura manual:

- **Telepathy** es el *toolkit* clásico (archivar chats, listar miembros, analizar los que más postean, mapear mensajes). <mark style="background: #FFB8EBA6;">Pero su versión OSS se quedó en 2024 y el repo indica que ya no se mantiene</mark> (empuja a un *Telepathy Pro* comercial) — verifícalo antes de depender de él.
- **Telethon** es la librería Python sobre la API de Telegram, **activamente mantenida** (v1.45.0, sep-2026). Es la base para construir tu propio recolector (listar miembros de un grupo, volcar el historial de un canal, seguir actividad):

```python
# Esqueleto con Telethon: volcar miembros de un grupo público
from telethon.sync import TelegramClient
with TelegramClient('session_sockpuppet', API_ID, API_HASH) as client:
    for user in client.iter_participants('nombre_del_grupo'):
        print(user.id, user.username, user.phone)  # phone solo si es visible
```

> [!important]+ Sigue el ID numérico, no el username
> <mark style="background: #FF5582A6;">El username de Telegram cambia de manos; el **ID numérico es inmutable**</mark>. Para rastrear a un actor a lo largo del tiempo, ancla al ID, no al `@alias` (que puede vender, soltar o cambiar). El username, además, suele reutilizarse en otras redes → [[01 - Enumeración de usernames cross-platform|pivote de username]].

# Discord

- **Servidores públicos**: con una cuenta sock puppet puedes unirte vía *invite link*, ver canales, roles, historial y miembros.
- **El ID (snowflake)**: los IDs de Discord **codifican la fecha de creación** de la cuenta/mensaje — un dato temporal gratis (cuentas muy nuevas = sospechosas).
- **Pivote**: los perfiles enlazan a otras redes; el username se reutiliza.

# La cautela imprescindible

> [!danger]+ Los bots de "lookup" de Telegram son datos robados
> Telegram está lleno de bots que ofrecen <mark style="background: #FF5582A6;">acceso a bases de datos de brechas, dumps de credenciales y exports de chats</mark> por teléfono, correo o nombre. Su legalidad y procedencia son **dudosas**, violan el `ToS` y usarlos para *doxxing*, acoso o fuera de un marco autorizado es ilegal. No son "herramientas OSINT": son mercados de datos robados. Si necesitas datos de brecha, usa las vías con [[Brechas y filtraciones.base|controles legales]], no un bot anónimo.

Telegram opera siempre desde un **sock puppet** con número desechable, nunca tu cuenta ([[01 - Sock puppets - crear y mantener identidades de investigación|OPSEC]]).

> [!info]+ Fuentes
> Descubrimiento y herramientas 2026: [Authentic8, *Telegram OSINT research*](https://authentic8.com/blog/telegram-osint-research/); [Max Intel, *Telegram OSINT 2026*](https://maxintel.org/telegram-osint.html). [Telethon](https://github.com/LonamiWebs/Telethon) (mantenida, v1.45.0 sep-2026); [Telepathy](https://github.com/jordanwildon/Telepathy) (OSS sin mantenimiento desde 2024). ID de Discord como snowflake con timestamp: documentación de desarrolladores de Discord.
