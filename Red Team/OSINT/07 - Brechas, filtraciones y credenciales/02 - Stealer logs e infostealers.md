---
tags:
  - OSINT/Brechas
  - Pentesting/Enumeracion
Descripción: "Qué roba un infostealer, por qué las cookies de sesión saltan el MFA (pass-the-cookie) y cómo detectar si el objetivo tiene máquinas comprometidas"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Buscadores de brechas - HIBP, Dehashed e IntelX]]"
Nota siguiente: "[[03 - Credential reuse, combolists y password spraying]]"
Area: "[[Brechas y filtraciones.base|Brechas y filtraciones]]"
---
---

Los *stealer logs* son la evolución que ha cambiado el juego del acceso por credenciales. Entender **qué contienen** y **por qué rompen el MFA** es lo que separa el OSINT de brechas moderno del de hace cinco años.

# Qué es un infostealer y qué roba

<mark style="background: #ADCCFFA6;">Un *infostealer* es malware que, al infectar un equipo, vuelca todo lo aprovechable de los navegadores y el sistema en un "log"</mark>. Un solo log (= una máquina infectada) contiene típicamente:

- **Credenciales guardadas** de todos los navegadores (en claro).
- <mark style="background: #FF5582A6;">**Cookies de sesión y tokens** activos</mark> — la pieza clave.
- **Autofill** (direcciones, tarjetas, formularios).
- **Fingerprint** del equipo y del navegador.
- **Wallets de criptomonedas**, datos de correo/mensajería.
- **Información del sistema** y a menudo una **captura del escritorio** en el momento de la infección.

Variantes dominantes en 2026: **Lumma, Vidar, StealC** (RedLine fue histórico). Flashpoint contó 1.700 M de credenciales robadas así solo en el primer semestre de 2026.

# Por qué un stealer log vale más que una brecha

Dos razones: <mark style="background: #FFB86CA6;">**frescura** (la credencial es de hace días, no de 2012, así que probablemente sigue viva) y **cookies de sesión**</mark>. Lo segundo es lo grave.

# El pass-the-cookie: saltarse el MFA

<mark style="background: #8000E1A6;">Una cookie de sesión se emite **después** de un login correcto — así que lleva incorporada la aprobación del MFA de ese login</mark>. El mecanismo del abuso:

```text
Víctima se loguea en portal.acme.com  →  pasa MFA  →  recibe cookie de sesión
   infostealer roba la cookie
      atacante la importa en su navegador
         → reanuda la sesión de la víctima
            → SIN contraseña, SIN reto MFA
```

Es el **`pass-the-cookie`**. Consecuencias que hay que entender:

- <mark style="background: #FF5582A6;">El MFA **no protege** contra esto</mark>: la cookie ya pasó el MFA. Por eso el stealer log es tan valioso.
- **Sobrevive al cambio de contraseña**: resetear la contraseña **no** invalida la cookie robada. Solo **revocar las sesiones** (cerrar sesión en todos los dispositivos) la mata. Por eso, ante un compromiso, revocar sesiones importa tanto como cambiar la contraseña.

# El ecosistema

Los logs se venden en foros clandestinos y **canales de Telegram**, muchos buscables por cookie de alto valor. Los **IAB** (*initial access brokers*) peinan los logs buscando accesos corporativos —portales **VPN**, **SSO**, **RDP**— y revenden ese acceso a afiliados de ransomware. Un log con la cookie de SSO de un empleado de ACME es, literalmente, acceso a ACME en venta.

# El ángulo OSINT: ¿tiene el objetivo máquinas infectadas?

Para el red teamer, lo accionable es saber si el objetivo aparece en logs:

- <mark style="background: #ADCCFFA6;">**Hudson Rock (Cavalier)** comprueba gratis si un dominio/correo está en stealer logs</mark> → equipos de la org comprometidos.
- **HIBP** marca exposición por stealer log; **SpyCloud** y **Flare** (empresariales) indexan logs y permiten alertas.

Que un empleado de ACME aparezca en un log reciente significa: sus credenciales corporativas y, posiblemente, sus **cookies de sesión** están comprometidas → acceso directo, saltándose el MFA.

> [!danger]+ El límite — y lo que NO se hace
> Consultar Hudson Rock/HIBP por exposición es OSINT legítimo. **Comprar logs** en mercados clandestinos, usar bots de Telegram de procedencia dudosa o importar cookies robadas para acceder **fuera del alcance** es participar en el mercado criminal y es delito ([[04 - Marco legal y ético del OSINT|art. 197/197 bis CP]]). En un engagement, el *pass-the-cookie* solo se valida con material y alcance autorizados por el cliente. Repórtalo como exposición; no cruces a la explotación sin permiso explícito.

> [!info]+ Fuentes
> Contenido de un log y pass-the-cookie: [SpyCloud, *Infostealer Malware*](https://spycloud.com/glossary/infostealer-malware/); [The Hacker News, *Infostealer logs & replayable tokens*](https://thehackernews.com/2026/09/infostealer-logs-expose-replayable-ai.html). Escala 2026: [Infosecurity Magazine](https://www.infosecurity-magazine.com/news/infostealers-17-billion/). Detección de exposición: [Hudson Rock](https://www.hudsonrock.com).
