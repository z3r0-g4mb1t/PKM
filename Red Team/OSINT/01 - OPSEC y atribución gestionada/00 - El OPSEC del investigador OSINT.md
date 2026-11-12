---
tags:
  - OSINT/OPSEC
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Por qué investigar deja rastro y cómo la atribución gestionada en cuatro capas protege tu identidad y tu operación"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Sock puppets - crear y mantener identidades de investigación]]"
Area: "[[OPSEC OSINT.base|OPSEC OSINT]]"
---
---

El OSINT parece de sentido único —tú miras, el objetivo no— pero no lo es. <mark style="background: #FF5582A6;">Cada plataforma que consultas te ve consultarla</mark>, y algunas se lo cuentan al objetivo. Si investigas a ACME con tu cuenta real de LinkedIn, dejas un patrón —tu perfil visitando a 40 empleados de la misma empresa— que su equipo puede ver y atribuirte. Peor aún en bug bounty o red team contra un adversario capaz: quemar tu identidad real puede costarte la operación, la cuenta o la seguridad personal.

<mark style="background: #ADCCFFA6;">El `OPSEC` del investigador (o **atribución gestionada**, *managed attribution*) es el conjunto de medidas para que tu actividad de recolección no se pueda vincular ni a tu identidad real ni entre sí</mark>. No es paranoia: es la diferencia entre un investigador profesional y un aficionado que se delata en la primera búsqueda.

# El modelo de las cuatro capas

La atribución gestionada se construye en cuatro capas independientes. Fallar en una compromete todo el conjunto —la cadena es tan fuerte como su eslabón más débil—:

| Capa | Qué protege | Se desarrolla en |
| - | - | - |
| **Identidad separada** | Que la actividad no apunte a ti | [[01 - Sock puppets - crear y mantener identidades de investigación\|Sock puppets]] |
| **Máquina aislada** | Que un enlace/pixel malicioso no infecte tu equipo real | [[02 - Infraestructura de investigación - VM, red y navegador\|VM e infra]] |
| **Red anonimizada** | Que tu IP real no quede en los logs | [[02 - Infraestructura de investigación - VM, red y navegador\|VM e infra]] |
| **Conducta disciplinada** | Que no te delates tú mismo (fugas, correlación) | [[03 - Cómo no quemar la operación - fugas y errores comunes\|Fugas y errores]] |

<mark style="background: #8000E1A6;">La capa que más se descuida y más delata es la cuarta</mark>: la mejor VM y el mejor sock puppet no sirven de nada si buscas al objetivo con tu Google logueado, reutilizas tu avatar de siempre o tu navegador filtra tu zona horaria real.

# Autorización primero

Antes de crear un solo sock puppet: **permiso por escrito** que fije propósito, alcance y fecha de caducidad. Un marco de referencia (el *OSINT Foundation policy framework*, 2026) insiste en algo sensato: <mark style="background: #FFB8EBA6;">las autorizaciones abiertas se convierten en capacidades permanentes que nadie revisa</mark> — una cuenta falsa creada "por si acaso" y olvidada es un riesgo legal y operativo. En un engagement, esto vive en las [[03 - Pre-engagement I - contratos, NDA y scoping|Rules of Engagement]].

# Proporcionalidad: no toda investigación necesita Tails

El OPSEC tiene coste (tiempo, fricción). Calíbralo al riesgo:

- **Consulta trivial** (un `WHOIS`, un dork anónimo): basta un navegador limpio y una VPN. No vincula nada a ti.
- **Trabajo con cuentas** (navegar LinkedIn, interactuar en foros): exige **sock puppet** + navegador/perfil aislado + IP que no te delate.
- **Objetivo sensible o capaz** (un APT, un grupo criminal, dark web): máquina desechable o **Tails**, red vía **Tor**, y disciplina estricta. Aquí un error se paga caro.

> [!warning]+ El sock puppet no es un disfraz para hacer cualquier cosa
> La atribución gestionada te protege a **ti**, pero no te autoriza a **actuar**. Sigue sin poder: suplantar a una persona **real**, acceder a sistemas, usar las cuentas falsas para estafar o acosar, ni salirte del alcance. Es OPSEC defensivo del investigador, no una licencia. El marco legal, en [[04 - Marco legal y ético del OSINT]].

> [!info]+ Fuentes
> Modelo de cuatro capas y constricciones 2026: [Max Intel, *OSINT OpSec & Managed Attribution Guide 2026*](https://maxintel.org/opsec-osint-guide-2026.html). Sock puppets y política de autorización: [SANS, *What Are Sock Puppets in OSINT*](https://www.sans.org/blog/what-are-sock-puppets-in-osint); OSINT Foundation policy framework (2026).
