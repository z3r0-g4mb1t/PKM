---
tags:
  - PKM/Decisiones
Fecha de actualización: 2026-11-12
Estado: Aceptada
---

## Contexto
`02 - Recursos/🛠️ Tools/` creció a **41 herramientas** en lista plana. Encontrar una en el explorador de archivos es tedioso y empeora con cada alta. Además, la fórmula `categoría` de `Tools.base` era un `containsAny` con los **41 nombres hardcodeados** (deuda señalada desde ADR 019 / 022: cada herramienta nueva obligaba a editar la fórmula a mano).

## Decisión
Mover las 41 carpetas a **10 subcarpetas numeradas por función** dentro de `🛠️ Tools/`:

`01 - Recon y escaneo` · `02 - Evaluación de vulnerabilidades` · `03 - Web` · `04 - Red y MITM` · `05 - Credenciales` · `06 - Explotación` · `07 - Wi-Fi` · `08 - Reversing y binario` · `09 - SAST` · `10 - Seguridad de IA`

- Movimiento con `app.fileManager.renameFile` (API de Obsidian), **no `mv`** (ADR 017): mantiene coherente el `metadataCache` y los wikilinks.
- `Tools.base`: la fórmula `categoría` pasa a **derivarse de `file.folder`** (`contains("/NN - …")`); el filtro `inFolder` de la vista "Todas las notas" pasa a `startsWith` para seguir viendo notas dos niveles abajo.

## Alternativas descartadas
- **Solo mejorar el índice `Tools.base`** (sin mover carpetas) — no arregla el dolor real, que es el explorador plano.
- **Taxonomía por objetivo** (web/red/wireless/…) — "Red" se convierte en un cajón de sastre de ~16 herramientas.
- **Taxonomía por fase del pentest** — muchas herramientas cruzan varias fases (Metasploit, Scapy), encaje ambiguo.

## Consecuencias
- Explorador navegable: 10 buckets ordenados en vez de 41 carpetas alfabéticas.
- **Deuda ADR 019 resuelta**: añadir una herramienta a una categoría existente ya **no toca** la fórmula (la hereda por vivir en la carpeta); solo una categoría nueva añade una línea.
- **Nada se rompe**: los `.base` por-herramienta (`Area == link(...)`), los `[[X.base|X]]`/`Area` de las notas (por *basename*) y el filtro `startsWith("02 - Recursos/🛠️ Tools")` son *path-independent*. Verificado: `md=153`, `base=42` (41 + `Tools.base`), 0 carpetas sueltas, fórmula con paréntesis balanceados (10 `if` / 10 `)`).
- Tras el *rename* hay que tirar la caché de Dataview (`dataview:dataview-drop-cache`) — hecho.
- **Deuda preexistente NO tocada**: casing `John The Ripper` (carpeta) ↔ `John the Ripper.base`; nombres con emoji de `🔨📦 Scapy.md` y las notas de Firewalk. Las referencias *inline* a rutas de herramientas en `CLAUDE.md` quedan sin el subnivel de categoría (cosméticas; barrido pendiente opcional).
