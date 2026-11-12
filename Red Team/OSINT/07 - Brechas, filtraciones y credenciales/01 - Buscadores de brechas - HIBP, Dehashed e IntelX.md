---
tags:
  - OSINT/Brechas
  - Pentesting/Enumeracion
Descripción: "Qué consultar y con qué: HIBP para exposición (y Pwned Passwords por k-anonymity), Dehashed para credenciales reales, IntelX y Hudson Rock"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - Brechas y credenciales - la economía del credencial robado]]"
Nota siguiente: "[[02 - Stealer logs e infostealers]]"
Area: "[[Brechas y filtraciones.base|Brechas y filtraciones]]"
---
---

Cada buscador de brechas responde a una pregunta distinta. Elegir mal te hace pagar por lo que otro da gratis, o buscar credenciales donde solo hay exposición. Esta es la guía de cuál usar para qué.

# Have I Been Pwned (HIBP) — exposición, no contraseñas

<mark style="background: #ADCCFFA6;">HIBP (de Troy Hunt) te dice **en qué brechas** aparece un correo, pero **no** te da la contraseña</mark>. Es la referencia para evaluar exposición:

- **Búsqueda por correo**: la web es gratis para una persona; la API (automatizar, por dominio del objetivo) requiere clave de pago (~4 $/mes).
- **Búsqueda por dominio**: gratuita **tras verificar que controlas el dominio** → lista todas las cuentas `@tu-dominio` en brechas. En un engagement, el cliente puede darte este acceso.
- **Stealer logs**: HIBP incorporó soporte de *stealer logs*, así que marca también exposición por infostealer.

El truco más elegante de HIBP es **Pwned Passwords con k-anonymity**: comprueba si una contraseña está en alguna brecha **sin enviarla**. Calculas el SHA-1, mandas solo los **5 primeros caracteres** del hash y HIBP devuelve todos los sufijos que coinciden; tú buscas el tuyo en la lista local:

```shell-session
# SHA-1 de "password" = 5BAA61E4C9B93F3F0682250B6CF8331B7EE68FD8
$ curl -s https://api.pwnedpasswords.com/range/5BAA6 | grep -i 1E4C9B93F3F0682250B6CF8331B7EE68FD8
# 1E4C9B93F3F0682250B6CF8331B7EE68FD8:<nº de veces visto>
```

<mark style="background: #FFB8EBA6;">Nunca viaja la contraseña ni su hash completo</mark> — por eso lo usan los gestores de contraseñas para avisarte. Útil para validar si una contraseña candidata ya está quemada.

# Dehashed — credenciales reales

<mark style="background: #FFB86CA6;">Cuando necesitas la **contraseña** (no solo saber que hubo brecha), Dehashed es el estándar</mark>. Es de pago y permite buscar por `email`, `username`, `name`, `phone`, `ip_address`, `domain` o `hash`, devolviendo el registro completo — incluida la contraseña en claro o hasheada:

```text
# Búsqueda por dominio: todas las credenciales filtradas de la organización
domain:acme.com
# Por correo concreto
email:jdoe@acme.com
# Por teléfono (pivote desde OSINT de teléfono)
phone:+34600112233
```

Para un red team, `domain:acme.com` es la consulta que convierte "sabemos que ACME ha tenido brechas" en "aquí están 340 pares correo:contraseña de empleados de ACME" — la munición directa para [[03 - Credential reuse, combolists y password spraying|spraying]].

# IntelX (Intelligence X) — dónde aparece el dato

**IntelX** es un buscador de leaks, *pastes* y dark web por **selector** (correo, dominio, IP, teléfono, dirección bitcoin, etc.). No se centra en credenciales sino en **dónde** aparece un identificador: en qué volcado, en qué paste, en qué documento filtrado. Útil para rastrear la huella de un dato y descubrir filtraciones que los otros no indexan. Parte es gratis; el histórico completo, de pago.

# Hudson Rock — el ángulo stealer log

<mark style="background: #ADCCFFA6;">**Hudson Rock** (Cavalier) comprueba gratis si un correo o dominio aparece en **stealer logs**</mark> (equipos infectados por infostealer). Es la vía rápida para saber si hay máquinas de la organización comprometidas — el dato más valioso, porque implica credenciales **vivas** y cookies ([[02 - Stealer logs e infostealers|siguiente nota]]).

# Cuál usar para qué

| Pregunta | Herramienta |
| - | - |
| ¿Está este correo/dominio expuesto? | HIBP, Hudson Rock |
| Dame las **contraseñas** del dominio | Dehashed |
| ¿Está esta contraseña quemada (sin enviarla)? | HIBP Pwned Passwords |
| ¿Dónde aparece este selector (pastes, dark web)? | IntelX |
| ¿Hay equipos de la org infectados? | Hudson Rock |

> [!danger]+ Recordatorio legal
> Recuperar credenciales de Dehashed es trivial; **usarlas** contra el objetivo solo es legítimo dentro del alcance activo autorizado. Fuera de eso, es [[04 - Marco legal y ético del OSINT|acceso ilícito]]. Documenta la fuente y trata estos datos (personales, robados) con las salvaguardas del RGPD.

> [!info]+ Fuentes
> [Have I Been Pwned](https://haveibeenpwned.com) y [Pwned Passwords (k-anonymity)](https://haveibeenpwned.com/API/v3#PwnedPasswords). [Dehashed](https://dehashed.com), [Intelligence X](https://intelx.io), [Hudson Rock](https://www.hudsonrock.com). Técnica k-anonymity: Cloudflare/Troy Hunt (diseño original de *Pwned Passwords*).
