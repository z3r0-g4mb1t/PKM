---
tags:
  - OSINT/OPSEC
  - Pentesting/Enumeracion
Descripción: "El catálogo de fugas que vinculan tu identidad real o tus personas entre sí, cómo probarlas y una checklist previa a cada operación"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - Infraestructura de investigación - VM, red y navegador]]"
Nota siguiente:
Area: "[[OPSEC OSINT.base|OPSEC OSINT]]"
---
---

La cuarta capa —la conducta— es la que más operaciones quema. <mark style="background: #FF5582A6;">No te delata la plataforma, te delatas tú</mark>: una fuga técnica del navegador, un username reutilizado, una búsqueda con tu cuenta real. Este es el catálogo de los fallos y cómo cerrarlos.

# Fugas técnicas del navegador

Aunque uses VPN, el navegador puede escupir tu IP o tu localización reales:

- <mark style="background: #FFB86CA6;">**WebRTC**</mark>: filtra tu IP real (incluso tras VPN) vía el *stack* de WebRTC. Desactívalo y verifica.
- **Fugas de DNS**: si tu DNS no va por el túnel, tu resolver (y tu ISP) delatan los dominios que visitas.
- **Zona horaria e idioma**: tu navegador anuncia `Accept-Language` y tu `timezone` por JavaScript. Un puppet que dice vivir en Texas pero manda `es-ES` y hora de Madrid se cae. **Ajústalos a la persona.**
- **Fingerprint**: la combinación de user-agent, fuentes, resolución, canvas, etc. te identifica entre sesiones. Mullvad Browser lo mitiga.

Pruébalo todo **antes** de operar, con páginas de test:

```shell-session
# Comprobar IP, DNS, WebRTC, zona horaria y fingerprint que expone tu navegador
# (abrir desde la VM, con la VPN/proxy ya puesta)
https://browserleaks.com     # fingerprint, WebRTC, canvas, zona horaria
https://ipleak.net           # IP, fugas DNS y WebRTC
```

Si `browserleaks` muestra tu IP real o tu hora local, tienes una fuga que cerrar antes de tocar el objetivo.

# Correlación de identidades

Lo que vincula dos cuentas (o una cuenta contigo) rara vez es la IP; suele ser un dato reutilizado:

- <mark style="background: #FF5582A6;">**Username / avatar / bio repetidos**</mark>: el mismo alias en Twitter, GitHub y Reddit los une en segundos (es, literalmente, lo que hace el [[OSINT de personas.base|OSINT de personas]] con los objetivos — no se lo pongas fácil a quien te investigue a ti).
- **Correo o teléfono de recuperación compartido**: si dos cuentas comparten recuperación, están unidas. Cada puppet, su correo y su teléfono.
- **"Personas que quizá conozcas"**: Google y LinkedIn sugieren contactos cruzando agendas y teléfonos. <mark style="background: #FFB8EBA6;">Si el dispositivo o la cuenta del puppet toca tus contactos reales, la plataforma os vincula</mark> y puede sugerirte a ti a los objetivos (¡o al revés!). Nunca importes contactos ni des permiso de agenda.
- **Estilo de escritura y horarios**: escribir siempre a tu hora local real, o con tus muletillas, es correlacionable. En objetivos muy sensibles, hasta esto cuenta.

# Errores clásicos que delatan

- **Buscar al objetivo con tu Google logueado**: personaliza resultados y **registra** tu actividad asociada a tu cuenta real. Investiga deslogueado o desde el puppet.
- **LinkedIn "quién ha visto tu perfil"**: tu visita se le notifica al objetivo. Navega en modo privado/anónimo (ajuste de LinkedIn) o con un puppet con la visibilidad desactivada.
- **Subir tus propios archivos con metadatos**: una imagen o PDF que subas lleva `EXIF` (GPS, dispositivo, software). **Limpia los metadatos** antes de subir nada (ver [[GEOINT e imágenes.base|metadatos]]).
- **Rastro de pago**: pagar el proxy, el SIM o una herramienta con tu tarjeta real vincula la infraestructura contigo. Para operaciones sensibles, medios de pago separados.
- **Descargar el cebo**: abrir un *canary token* o resolver un subdominio trampa te delata (ver [[05 - Verificación, fiabilidad de fuentes y sesgos#Datos envenenados: cuando la fuente te tiende una trampa|datos envenenados]]).

# Checklist antes de cada operación

> [!important]+ Pre-flight OPSEC
> - [ ] VM dedicada, revertida a snapshot limpio.
> - [ ] Red correcta para la tarea (VPN / proxy residencial / Tor) y **activa**.
> - [ ] `browserleaks` + `ipleak`: sin fuga de IP real, DNS, WebRTC; zona horaria e idioma coherentes con la persona.
> - [ ] Navegador/perfil del puppet, **separado** del real; sin sesiones reales abiertas.
> - [ ] Google/LinkedIn **deslogueados** de tu cuenta real; visibilidad de perfil desactivada en el puppet.
> - [ ] Ficha de la persona a mano; dos cuentas por persona disponibles.
> - [ ] Autorización y alcance confirmados.

<mark style="background: #8000E1A6;">El OPSEC no es un estado, es un hábito de cada sesión</mark>. La checklist de treinta segundos evita la fuga que cuesta la operación. Con la identidad protegida, ya se puede recolectar en serio — empezando por la dimensión que lo conecta todo, el [[OSINT de personas.base|OSINT de personas]], o por la técnica más universal, el [[Google Dorking.base|Google Dorking]].

> [!info]+ Fuentes
> Fugas de navegador y pruebas: [browserleaks.com](https://browserleaks.com), [ipleak.net](https://ipleak.net). Correlación de identidades y disciplina de conducta: [Max Intel, *OSINT OpSec Guide 2026*](https://maxintel.org/opsec-osint-guide-2026.html); [SANS, *Sock Puppets in OSINT*](https://www.sans.org/blog/what-are-sock-puppets-in-osint).
