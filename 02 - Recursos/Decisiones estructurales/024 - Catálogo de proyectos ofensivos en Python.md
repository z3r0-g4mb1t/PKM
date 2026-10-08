---
tags:
  - PKM/Decisiones
Fecha de actualización: 2026-09-21
Estado: Aceptada
---
---

## Contexto

El [[021 - Catálogo de proyectos ofensivos en Go]] sentó el patrón de las notas-proyecto en `Red Team/Proyectos/`: notas de planteamiento (no de concepto ni de técnica), con frontmatter propio (`Estado`, `Dificultad`, `Esfuerzo`), indexadas por `Proyectos ofensivos.base` y pintadas en el BLOQUE 3 de la Home. Se añade ahora un **segundo catálogo de 14 proyectos pensados específicamente para Python** (16–29), 100% net-new respecto a los 16 de Go, que exprimen el ecosistema Python (estadística/`scipy`, `z3`, Impacket, pySigma, SDK de MCP, `ast`/`pickletools`, `Hypothesis`). La Home solo mostraba proyectos Go y hacía falta poder ver unos u otros.

## Decisión

1. **Misma carpeta, numeración continua 16–29** (no subcarpeta `Python/`, no reorganización en `Go/`+`Python/`). Un único catálogo, ordenado globalmente por dificultad vía el número de fichero.
2. **Cadena Zettelkasten única**: `15 → 16 → … → 29`. Un solo recorrido de 30 notas, una cabeza (`00`) y una cola (`29`).
3. **Distinción de lenguaje por tag** (`#Go` / `#Python`), ya presente en el frontmatter de cada nota. Se añade además la propiedad `Lenguaje` a las nuevas por autodocumentación, pero el filtro real casa por tag para no tocar las 16 notas de Go.
4. **Alternador de tres botones en la Home** (Todos / Go / Python): filtra las filas por CSS (`data-filter` en `.pkm-projs`) y recalcula los KPIs por lenguaje. Dos vistas nuevas en el `.base` (`Ruta Go`, `Ruta Python`) con filtro `file.hasTag(...)`.

## Alternativas descartadas

- **Subcarpeta `Python/` con `.base` propio y numeración 00-09** — catálogos separados, pero deja el árbol asimétrico (Go suelto en la raíz, Python anidado) y crea dos notas `00 - …`. Rechazada por asimetría y por partir un catálogo que conceptualmente es uno.
- **Reorganizar en `Go/` + `Python/` con `Proyectos ofensivos.base` ascendido a Level 1** — lo más simétrico a futuro, pero exige mover las 16 notas de Go con el procedimiento del [[017 - Nombres de carpeta sin emoji y numeración de dos dígitos|ADR 017]] (API de Obsidian, reescribir filtros, rutas del `dataviewjs`, tirar caché de Dataview). Coste alto para un beneficio estético; reversible más adelante si el catálogo crece mucho.
- **Distinguir por propiedad `Lenguaje` en vez de tag** — habría obligado a editar las 16 notas de Go para añadirles `Lenguaje: Go`. El tag `#Go`/`#Python` ya existe y basta; la propiedad queda como redundancia autodocumentada, no como fuente del filtro.
- **Toggle que reconsulta Dataview por lenguaje** — más "puro", pero innecesario: pintar todas las filas una vez y ocultar por CSS es instantáneo y evita el coste de re-render de Dataview (mismo criterio de pintado optimista del [[013 - Home como dashboard generado y CSS scoped por cssclass|ADR 013]]).

## Consecuencias

- La Home muestra ambos catálogos con el alternador; por defecto `Todos`. Sin cambios en las 16 notas de Go.
- `Proyectos ofensivos.base` pasa de 4 a 6 vistas (Ruta, Ruta Go, Ruta Python, Por estado, Portfolio, Siguiente a construir).
- El CSS del snippet `home-dashboard.css` gana el bloque `.pkm-langbar`/`.pkm-langbtn` y las reglas de ocultación por `data-filter`. **Verificar en modo claro y oscuro** (regla del ADR 013).
- Si el catálogo Python crece hasta descompensar la carpeta, la opción de reorganizar en `Go/`+`Python/` sigue disponible y este ADR quedaría reemplazado.
- No afecta a las tablas de path de `CLAUDE.md` (los proyectos no viven ahí, igual que los de Go).
