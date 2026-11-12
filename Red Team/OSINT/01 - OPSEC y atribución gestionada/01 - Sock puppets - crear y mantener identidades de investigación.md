---
tags:
  - OSINT/OPSEC
  - Pentesting/Enumeracion
Descripción: "Guía paso a paso para crear, envejecer y mantener identidades de investigación creíbles sin quemarlas ni quemarte"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - El OPSEC del investigador OSINT]]"
Nota siguiente: "[[02 - Infraestructura de investigación - VM, red y navegador]]"
Area: "[[OPSEC OSINT.base|OPSEC OSINT]]"
---
---

<mark style="background: #ADCCFFA6;">Un *sock puppet* es una identidad de investigación falsa, construida y mantenida para que tu actividad OSINT no apunte a ti</mark>. No es una cuenta vacía creada en cinco minutos: una cuenta sin historia la detecta cualquiera (y las plataformas la banean sola). Un sock puppet útil es **creíble y envejecido**. Aquí va el proceso completo.

# 1. Construir la identidad base

Empieza por una **persona coherente**, no por la cuenta. Define y **apunta en una ficha** (la llevarás durante toda la vida del puppet):

- **Nombre y datos**: usa un generador como `fakenamegenerator.com`, que produce un conjunto consistente (nombre, fecha de nacimiento, dirección plausible para un país/ciudad reales, etc.). La coherencia importa: un nombre hispano con dirección en Ohio y zona horaria de Madrid se cae solo.
- **Rostro**: evita fotos reales de terceros (robo de identidad). Los rostros generados por IA (`thispersondoesnotexist.com`, StyleGAN) sirven, con dos avisos: <mark style="background: #FFB8EBA6;">tienen *tells*</mark> (ojos siempre centrados, fondos deformes, pendientes asimétricos) y, aunque son únicos, un rostro "de catálogo" puede delatarse. Para muchos puppets de bajo perfil, **mejor sin foto de cara** (un avatar neutro) que una cara sospechosa.
- **Biografía**: un relato mínimo coherente con la plataforma donde vivirá el puppet (si va a investigar un sector, dale un perfil de ese sector).

# 2. Correo y teléfono: el cuello de botella

- **Correo**: crea uno **dedicado** (ProtonMail, o un Gmail nuevo generado **desde el entorno aislado**, nunca desde tu navegador real). <mark style="background: #FF5582A6;">Jamás uses tu correo personal ni tu teléfono real como recuperación</mark> — ese es el fallo que vincula el puppet contigo de forma permanente.
- **Teléfono**: es la parte más difícil. Muchas plataformas exigen verificación por SMS y **marcan los números VoIP** (Google Voice, TextNow suelen rechazarse). Los números virtuales de un solo uso están reutilizados y quemados. Para una operación seria, un **SIM de prepago** comprado en efectivo y dedicado es lo más fiable; para trabajo ligero, asume que no todas las plataformas te dejarán registrarte. <mark style="background: #FFB86CA6;">El teléfono es donde mueren la mayoría de los sock puppets</mark>.

# 3. Registrar sin que te marquen

El registro es el momento de máximo escrutinio antifraude:

- **No uses VPN ni Tor al crear la cuenta**. Las plataformas marcan los rangos de VPN/Tor y disparan verificación extra o baneo inmediato. Regístrate desde una **IP residencial limpia** (una IP móvil o residencial que no uses para nada más). Una vez creada y asentada la cuenta, ya compartimentas con VPN/proxy para el uso diario.
- Rellena el perfil **poco a poco**, como lo haría una persona: no completes todo al segundo de crear la cuenta.

# 4. Envejecer y dar vida

<mark style="background: #8000E1A6;">La credibilidad de un puppet es función de su edad y su actividad</mark>. Un perfil creado ayer con 2.000 contactos es tan sospechoso como uno vacío.

- **Créalo con antelación** —semanas o meses antes de necesitarlo— y déjalo reposar.
- Añade actividad orgánica: alguna conexión, algún post o *like*, una foto de perfil. Lo justo para que tenga historia.
- No lo uses para la operación hasta que parezca real.

# 5. Redundancia y disciplina

- **Dos cuentas por persona, como mínimo**: si una cae (baneo), la otra sigue. Una sola cuenta es un punto único de fallo.
- **Una persona, un contexto**: no cruces el mismo puppet entre investigaciones que no quieras vincular. Cada reutilización es una oportunidad de correlación.
- **Nunca mezcles** la actividad del puppet con la real en el mismo navegador/sesión/dispositivo — de eso va [[03 - Cómo no quemar la operación - fugas y errores comunes|la cuarta capa]].

> [!warning]+ 2025-2026: la verificación de edad complica los puppets
> Las leyes de *age assurance* (Reino Unido desde julio de 2025, Australia desde diciembre de 2025) obligan a varias plataformas a pedir **escaneo facial o documento** para ciertas cuentas. Un sock puppet no puede pasar un *face scan* legítimo, así que algunas plataformas quedan efectivamente cerradas a identidades falsas. Es una tendencia creciente: cuenta con que crear puppets será más difícil, no menos.

> [!danger]+ Dónde está el límite legal
> Crear una identidad ficticia viola los `ToS` de casi todas las plataformas (riesgo de baneo, no penal en sí). Pero **suplantar a una persona real** (usar su nombre, su foto, hacerte pasar por ella) cruza a la usurpación de identidad y puede ser delito. El puppet es **inventado**, no robado. Y en un engagement, su uso va **documentado y dentro del alcance** — ver [[04 - Marco legal y ético del OSINT]].

> [!info]+ Fuentes
> Craft de sock puppets: [SANS, *What Are Sock Puppets in OSINT*](https://www.sans.org/blog/what-are-sock-puppets-in-osint); [CTI Academy, *Sock Puppet Accounts*](https://ctiacademy.io/blog/sock-puppet-accounts-osint). Constricciones 2025-2026 (age assurance, flagging de VPN en registro): [Max Intel, *OSINT OpSec Guide 2026*](https://maxintel.org/opsec-osint-guide-2026.html).
