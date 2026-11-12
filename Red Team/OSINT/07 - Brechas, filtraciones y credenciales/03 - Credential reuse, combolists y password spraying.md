---
tags:
  - OSINT/Brechas
  - Pentesting/Enumeracion
Descripción: "Cómo el dato de brecha se convierte en acceso: reutilización de contraseñas, combolists, credential stuffing vs password spraying y la defensa"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - Stealer logs e infostealers]]"
Nota siguiente:
Area: "[[Brechas y filtraciones.base|Brechas y filtraciones]]"
---
---

Tener credenciales de brecha no es tener acceso: hay que convertir el dato en entrada. El puente es un hecho humano —la gente **reutiliza contraseñas**— y dos técnicas que lo explotan. La ejecución del ataque vive en [[Ataques a contraseñas.base|Password Attacks]]; aquí se ve **cómo el OSINT de brechas lo alimenta**.

# La premisa: la reutilización de contraseñas

<mark style="background: #ADCCFFA6;">Todo esto funciona porque la gente reutiliza la misma contraseña (o variantes mínimas) en muchos servicios</mark>. Si `jdoe` usó `Verano2023!` en un foro que sufrió una brecha, hay una probabilidad real de que la misma —o `Verano2024!`— abra su correo corporativo. El OSINT de brechas te da el punto de partida; la reutilización hace el resto.

# Combolists

<mark style="background: #FFB8EBA6;">Una *combolist* es una lista de pares `correo:contraseña` agregada de muchas brechas</mark>, el formato estándar para los ataques automatizados. Mucho ruido (credenciales muertas, duplicados), pero a volumen, un porcentaje sigue vivo. Para un objetivo concreto, en vez de una combolist genérica, construyes la tuya: las credenciales de `domain:acme.com` de [[01 - Buscadores de brechas - HIBP, Dehashed e IntelX#Dehashed - credenciales reales|Dehashed]].

# Las dos técnicas: stuffing vs spraying

| | **Credential stuffing** | **Password spraying** |
| - | - | - |
| Qué pruebas | Pares `user:pass` **conocidos** (de brecha) | **Pocas** contraseñas comunes contra **muchos** usuarios |
| Cuándo | Tienes credenciales reales del objetivo | Tienes la lista de usuarios pero no contraseñas |
| Riesgo de bloqueo | Bajo por cuenta (1 intento con la pass correcta) | Controlado: 1 intento por usuario por ronda |
| Fuente OSINT | Dehashed/stealer logs | [[01 - LinkedIn - la mina del red teamer\|LinkedIn]]/CrossLinked + patrón |

<mark style="background: #8000E1A6;">La elección depende de qué te dio el OSINT</mark>:

- **Credential stuffing**: si tienes `jdoe@acme.com:Verano2023!` de una brecha, lo pruebas (y sus variantes) contra los portales de ACME. Alta tasa de acierto si hay reutilización.
- **Password spraying**: si tienes la lista de empleados ([[01 - LinkedIn - la mina del red teamer|LinkedIn]]) pero no contraseñas, pruebas **una** contraseña plausible (`Acme2026!`, `Otoño2026!`) contra **todos** los usuarios. Una ronda por contraseña, con espera entre rondas, evita el bloqueo por cuenta.

# El pivote del patrón

El hallazgo más fino lo viste en el [[05 - De un dato a la persona - pivote completo|pivote de personas]]: una contraseña antigua en claro revela el **patrón** de la persona (`Austin2016!` → ciudad + año + símbolo). Eso te deja generar candidatas actuales (`Austin2026!`) que no están en ninguna brecha pero siguen la misma lógica mental. <mark style="background: #FFB86CA6;">La contraseña filtrada no vale tanto por sí misma como por el **patrón** que delata</mark>.

# La defensa (y por qué el atacante la sortea)

> [!info]+ Qué para esto y qué no
> - **MFA**: para el *stuffing*/*spraying* clásico, pero **no** el [[02 - Stealer logs e infostealers#El pass-the-cookie: saltarse el MFA|pass-the-cookie]] (la cookie ya pasó el MFA). La defensa real es **MFA resistente a phishing** (FIDO2/passkeys) + **revocación de sesiones**.
> - **Contraseñas únicas** (gestor de contraseñas): rompe la premisa de la reutilización. Es la defensa de raíz.
> - **Detección de spraying**: alertar sobre muchos fallos de login de una misma IP contra muchos usuarios; *conditional access* que bloquea por geolocalización/imposible viaje.
> - **Cuentas canario**: usuarios señuelo que nadie usa; un intento de login contra ellos delata un *spray* en curso.

> [!danger]+ El salto a la explotación necesita alcance
> Construir la combolist del objetivo es OSINT. **Lanzar** el stuffing/spraying contra sus portales es un ataque activo: solo dentro del alcance autorizado ([[04 - Marco legal y ético del OSINT|legal]]). La ejecución técnica (herramientas, tempo, evasión de bloqueo) está en [[Ataques a contraseñas.base|Password Attacks]] y en el [[00 - Introducción al brute forcing|brute forcing web]].

> [!info]+ Fuentes
> Stuffing vs spraying y reutilización: MITRE ATT&CK [T1110.003 *Password Spraying*](https://attack.mitre.org/techniques/T1110/003/) y [T1110.004 *Credential Stuffing*](https://attack.mitre.org/techniques/T1110/004/). Defensas: guía de MFA resistente a phishing (FIDO2/passkeys) y revocación de sesión.
