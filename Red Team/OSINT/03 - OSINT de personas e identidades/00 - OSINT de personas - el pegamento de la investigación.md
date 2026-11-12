---
tags:
  - OSINT/Personas
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Por qué las identidades son el pivote de toda investigación: el grafo nombre-username-correo-teléfono-cuentas y el flujo para recorrerlo"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Enumeración de usernames cross-platform]]"
Area: "[[OSINT de personas.base|OSINT de personas]]"
---
---

El OSINT de personas es <mark style="background: #8000E1A6;">el pegamento que conecta todas las demás dimensiones</mark>. Casi cualquier investigación entra o sale por una persona: de un username llegas a una cuenta de GitHub (código), de ahí a un correo (personas), ese correo lo buscas en brechas (credenciales) y el teléfono asociado te lleva a WhatsApp (SOCMINT). <mark style="background: #ADCCFFA6;">Dominar el pivote entre identificadores de una persona es la habilidad OSINT más transferible que existe</mark>.

Para el red teamer, además, es munición directa: el OSINT de personal ([[03 - Enumeración de personal]]) ya mostró que de los empleados salen las listas de usuarios y correos que alimentan *phishing* y *password spraying*. Esta sub-área desarrolla **cómo** se obtienen y se cruzan esos identificadores.

# El grafo de identidad

Piensa en la identidad de una persona como un grafo de identificadores conectados. Tienes **uno** y quieres los demás:

```text
            ┌─────────────┐
            │   NOMBRE    │
            └──────┬──────┘
         ┌─────────┼─────────┐
         ▼         ▼         ▼
    ┌────────┐ ┌───────┐ ┌─────────┐
    │USERNAME│ │CORREO │ │TELÉFONO │
    └───┬────┘ └───┬───┘ └────┬────┘
        │          │          │
        ▼          ▼          ▼
   cuentas en   cuentas,    WhatsApp,
   N plataformas brechas,   Telegram,
   (GitHub,     Google      Truecaller,
   Reddit...)   (GHunt)     brechas
```

<mark style="background: #FFB86CA6;">Cada identificador es a la vez un hallazgo y una llave para el siguiente</mark>. El arte está en recorrer el grafo sin perder el rastro ni saltar a conclusiones (un username común puede pertenecer a varias personas — ver verificación).

# El flujo de pivote

Un recorrido típico, que las siguientes notas desarrollan herramienta a herramienta:

1. **Semilla**: parte de lo que tengas (un nombre de LinkedIn, un alias de un foro, un correo de una fuga).
2. **Username → cuentas**: enumera ese alias en cientos de plataformas → [[01 - Enumeración de usernames cross-platform]].
3. **Correo → cuentas y servicios**: descubre en qué sitios está registrado ese correo y qué cuenta de Google/Microsoft tiene → [[02 - OSINT de correo electrónico]].
4. **Teléfono → mensajería e identidad** → [[03 - OSINT de teléfono]].
5. **Nombre → registros públicos y buscadores de personas** → [[04 - Buscadores de personas y registros públicos]].
6. **Cruzar y verificar**: una cuenta nueva revela un username nuevo, una bio revela un correo, una foto revela una localización. Se vuelve al paso 2 hasta cerrar el perfil → [[05 - De un dato a la persona - pivote completo]].

> [!warning]+ La trampa de la homonimia
> <mark style="background: #FF5582A6;">El mismo username o nombre no es la misma persona</mark>. `jsmith` en GitHub y `jsmith` en Reddit pueden ser dos individuos distintos. Antes de fusionar identificadores en un mismo perfil, **corrobora** (misma foto, misma bio, mismo correo, enlaces cruzados entre cuentas). Fusionar dos personas en una arruina la inteligencia — y en un informe, es un error grave. Aplica el [[05 - Verificación, fiabilidad de fuentes y sesgos|método de verificación]].

> [!danger]+ Esto es tratamiento de datos personales
> Construir el perfil de una persona **es** tratar datos personales bajo [[04 - Marco legal y ético del OSINT|RGPD]]. En un engagement: base jurídica, minimización (solo lo que un PIR necesite), alcance acotado a quien lo autorizó, y destrucción al cierre. Fuera de un marco autorizado, acumular dossieres de personas puede ser acoso.

Empezamos por el identificador más productivo para pivotar: el **username**.

> [!info]+ Fuentes
> El concepto de pivote de identidad es central en la metodología OSINT (Bazzell, *OSINT Techniques*; soxoj, [OSINT namecheckers list](https://github.com/soxoj/osint-namecheckers-list)). Munición para acceso: [[03 - Enumeración de personal]].
