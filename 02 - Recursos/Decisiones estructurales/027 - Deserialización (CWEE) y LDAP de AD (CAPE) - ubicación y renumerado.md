---
tags:
  - PKM/Decisiones
Fecha de actualización: 2026-11-20
Estado: Aceptada
---

## Contexto

Dos módulos nuevos desbloqueados en HTB Academy: *Introduction to Deserialization Attacks* (CWEE #11, módulo 169) y *Active Directory LDAP* (CAPE #2, módulo 22). El primero necesitaba carpeta en `Hacking web/`; el segundo introduce **CAPE** como path nuevo en el vault (id 417, verificado contra la API de HTB) y su contenido pisa el área AD ya existente, que tiene una numeración de notas corrida 00-26 entre 6 Level-2.

## Decisión

1. **Deserialización → `Hacking web/36 - Deserialización/`** (15 notas 00-14 + `.base`), siguiendo el patrón de Level-2 hermanos del ADR 016 (no anidar `Deserialization/Intro` + `/Advanced`). Esto desplaza en el mapeo: *Whitebox Attacks* (CWEE #12) de `36`→`37`, y *Advanced Deserialization* (CWEE #14) pasa a `38 - Deserialización avanzada`, hermano Level-2 de 36.
2. **LDAP de AD → `Red Team/Active Directory/02 - LDAP y enumeración/`** (8 notas + `AD LDAP.base`), **insertado como Level-2 `02`** entre `01 - Enumeración` y el antiguo `02 - Ataques de credenciales`, con **renumerado** del resto del área (carpetas 02-06→03-07, notas 06-26→14-34).
3. **CAPE dado de alta** completo: tabla de 15 módulos en `CLAUDE.md` con mapeo y `Estado`, y sección `## CAPE · …` en `📋 Temario.md` (la Home le crea el anillo sola).
4. **Las 4 secciones del módulo 22 ya cubiertas** (AD Overview, Why Enumerate, Rights & Privileges in AD, NT AUTHORITY\SYSTEM) se **enriquecen + cross-linkean**, no se duplican.

## Alternativas descartadas

- **Deserialización anidada (`Deserialization/Intro` + `/Advanced`)** — lo que decía el mapeo literal. Descartada: contradice el patrón de hermanos Level-2 del ADR 016 (como `35`/`36` whitebox, `XSS`/`XSS Avanzado`) y rompe la numeración plana del área web.
- **Deserialización en `37`, dejando `36` a Whitebox Attacks** — descartada: el siguiente número libre es `36`, y dejar un hueco reservado a un módulo aún sin extraer es peor que renumerar su destino tentativo.
- **LDAP como `07` al final del área AD** (coste cero de rotura) — descartada por el usuario a favor del orden lógico (LDAP justo tras Enumeración), asumiendo el coste del renumerado de ~21 notas.
- **LDAP fundido en `01 - Enumeración`** — descartada: mezcla "enumeración general" con "mecánica de LDAP" y choca con la numeración.
- **Notas nuevas para las 4 secciones solapadas** — descartada: duplicaría material ya escrito en dos áreas y crearía las incoherencias que la auditoría marca como deuda.

## Consecuencias

- **Mejora**: el área AD gana un sub-tema LDAP coherente y bien ubicado; CAPE queda visible en la Home con su progreso; la deserialización sigue el patrón de hermanos del resto del vault.
- **Se complica / pendiente**: el **renumerado del área AD está pendiente de ejecutar con Obsidian abierto** (receta ADR 017: `app.fileManager.renameFile`, de mayor a menor número, + empalme de cadena `05→06 LDAP` y `13 LDAP→14`, + caché de Dataview). Las notas LDAP (nº 06-13) **conviven sin romper nada** con la numeración antigua hasta entonces, porque los nombres de nota completos difieren y los `.base` Level-2 filtran por `Area==link(base)`, no por número. Verificado que Obsidian estaba cerrado en la sesión de creación (la CLI intentaba lanzar la app, no conectar).
- **`CLAUDE.md`**: tabla CWEE (#11 completado, #12→37, #14→38), path CAPE nuevo con su tabla y el bloque de renumerado pendiente. **Temario**: #11 marcado, sección CAPE añadida.
- **Modernización 2026** integrada en ambos módulos (investigación contra fuente primaria): deserialización → OWASP A08:2025, CWE 5.0, `yaml.load` TypeError (PyYAML 6.0), `jsonpickle safe=True` no asegura, PHAR vía `getMetadata()`, PEAS muerto→`fickling`, `gASV`→`gAWV`; LDAP → signing 2026 + `CVE-2026-70229`, ADWS/SOAPy, `windapsearch`/PowerView muertos→ldeep/NetExec/BloodHound CE, bind anónimo no común, 4 matching-rule OIDs, `MaxValRange` 1500.
