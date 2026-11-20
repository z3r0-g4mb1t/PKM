---
tags:
  - PKM/Decisiones
Fecha de actualización: 2026-11-20
Estado: Aceptada
---

## Contexto

Samuel pidió, para el área AD, (1) **base conceptual con más mimo**: varios ataques del área asumen conceptos low-level (Kerberos/PAC, identidad/SIDs/tokens, almacenes de credenciales, NTLM) que solo existían como resumen atacante en `Pentesting/05`, dejándolo sin entender tras "un párrafo y medio"; y (2) **profundizar la evasión**, porque su objetivo tras CPTS es **CRTO**, donde AD y detección/evasión pesan mucho. Interrogatorio previo (`pkm-design-grilling`): 4 decisiones fijadas con el usuario.

## Decisión

1. **Fundamentos como nuevo Level-2 `00 - Fundamentos de AD`** (`Fundamentos AD.base`), 4 notas profundas y net-new: `00` modelo de identidad (SIDs/RIDs/tokens/security descriptors), `01` dónde viven las credenciales (SAM/LSASS/NTDS/DPAPI/caché + formatos), `02` NTLM por dentro, `03` Kerberos por dentro (con el **PAC**). No duplican los resúmenes de `Pentesting/05`: los **sostienen** (cross-link bidireccional) y van más abajo.
2. **Segundo renumerado del área (+4)** vía `app.fileManager.renameFile`: fundamentos ocupan 00-03 y el resto pasa de 00-34 a 04-38, manteniendo la secuencia global monótona. Cadena empalmada por el frente (fundamentos → `04 - Introducción`).
3. **Integración in situ**: puntero "empieza por Fundamentos" en la intro + backlinks desde las notas de ataque a su fundamento (kerberoasting→Kerberos, ACLs→identidad, LLMNR→NTLM, DCSync→Kerberos).
4. **Evasión AD a fondo (foco CRTO)**: nuevo Level-2 `08 - Evasión y OPSEC` (`Evasión y OPSEC.base`), 6 notas 39-44 (modelo de detección/mindset, recon sigiloso, OPSEC de Kerberos, forjado de tickets y su detección, OPSEC de credential access, movimiento lateral/ejecución/deception). La nota `37` queda como referencia rápida "ataque→telemetría" y apunta al sub-tema. AMSI/ETW/inyección/C2 quedan como cross-link al área `Evasión de defensas` (en pausa), fuera de alcance. **Hecho.**
5. **CRTO registrada** en CLAUDE.md (misión) y Temario (anillo en la Home), como cert con temario mapeado al vault (no es path HTB).

## Alternativas descartadas

- **Fundamentos en `Redes/Protocolos` (Kerberos/NTLM)** siguiendo la regla *fundamentos ≠ ofensiva* — descartada: dejaría sin hogar al modelo de identidad y a los almacenes de credenciales (no son protocolos de red), y el usuario quería la base **accesible dentro del área AD**. Un sub-tema de fundamentos AD-céntrico cumple el espíritu de la regla (explica, no ataca) sin dispersar.
- **Solo ampliar in situ** las notas existentes — descartada: dispararía su longitud (límite ~1500) y mezclaría "qué es" con "cómo se ataca".
- **No renumerar (fundamentos al final, 35-38)** — descartada por el usuario a favor de la secuencia global limpia, asumiendo el churn de un segundo renumerado.
- **Evasión extendida a endpoint/C2 (AMSI/ETW/inyección)** — descartada de este trabajo: es de otra área (`Evasión de defensas`, en pausa); se cross-linkea como pendiente.

## Consecuencias

- **Mejora**: el área gana la base conceptual que faltaba, con secuencia limpia 00-38; los ataques dejan de ser recetas porque cada uno enlaza su fundamento. CRTO visible en la Home.
- **Hecho completo**: ambas mitades del encargo (fundamentos + evasión). El área pasó de 6 a **9 Level-2** (00-44, 45 notas). Pendiente menor: extender los backlinks a fundamentos a más notas de ataque (hecho en las 4 clave: kerberoasting, ACLs, LLMNR, DCSync).
- **Gotcha reconfirmado (2º renumerado)**: `renameFile` **no** reescribe wikilinks en bloques de código — el `[[… - Kerberoasting]]` del comentario `Get-DomainUser -SPN` hubo que corregirlo a mano otra vez. Anotado en CLAUDE.md.
- **CLAUDE.md**: nueva sección "Estructura del área Active Directory" (sustituye al bloque de renumerado pendiente de ADR 027), CRTO en la misión, rangos de notas LDAP actualizados (06-13 → 10-17).
