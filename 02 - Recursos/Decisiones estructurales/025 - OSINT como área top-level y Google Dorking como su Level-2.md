---
tags:
  - PKM/Decisiones
Fecha de actualización: 2026-11-08
Estado: Aceptada
---

## Contexto

El vault no tenía un área de **OSINT como disciplina**. Lo que había era **recon técnico disperso** por su fase/área: `Hacking web/02 - Reconocimiento Web` (WHOIS, DNS, subdominios, CT logs, fingerprinting, un apartado de Google Dorking dentro de `12 - Search Engine Discovery`), `Pentesting/01 - Footprinting` (dominio, cloud, enumeración de personal — ya tagueada `OSINT`), más piezas sueltas (cliente Shodan en Go, harvesting de correos). Faltaba justo lo que da ventaja a un red teamer y los junior no controlan: **metodología/ciclo de inteligencia, OPSEC del investigador, OSINT de personas/identidades, SOCMINT, brechas y stealer logs, GEOINT/imágenes/metadatos, dark web y los frameworks** (SpiderFoot/recon-ng/Maltego). El tag `OSINT` existía **plano** en 4 notas.

Google Dorking solo existía como **subsección** de una nota de recon web, insuficiente para un tema que merece desarrollo propio.

## Decisión

1. **OSINT = área top-level** en `Red Team/OSINT/` (Level-1 `OSINT.base`), con **13 sub-temas Level-2** (cada uno su `.base` y su cadena Zettelkasten 00→N independiente, como el resto de áreas multi-carpeta).
2. **Google Dorking = Level-2 dentro de OSINT** (`02 - Google Dorking`), no área hermana ni sub-sección de recon web. La subsección de `12 - Search Engine Discovery` se **recorta a puntero** con cross-link al área nueva.
3. **La capa de infraestructura/dominios de OSINT es un puente**: cross-linkea a `Reconocimiento Web` y `Footprinting` en vez de reexplicar DNS/subdominios/WHOIS. OSINT **posee** la disciplina y las dimensiones humana/social/brechas/geo/imágenes; el recon técnico sigue donde está.
4. **Tag jerárquico `OSINT/…`** (paraguas `OSINT` + `OSINT/Personas`, `OSINT/SOCMINT`, `OSINT/Brechas`, `OSINT/Geoint`, `OSINT/Dorking`, …), siguiendo el patrón `Wi-Fi/…` e `IA/…` (ADR 014). Las 4 notas con el tag plano se migran al esquema.
5. Se aplican los **3 ejes del vault**: `11 - Contra-OSINT` (eje 2, detección/reducción de huella) y `12 - Arsenal` (eje 3), con fuentes actuales (2026) y atribución por-fuente.

## Alternativas descartadas

- **Meter OSINT dentro de `Reconocimiento Web`** — no: OSINT es transversal a web/infra/personas/social; subordinarlo a "web" rompe su alcance y lo deja invisible para pentest de red/AD.
- **Google Dorking como área top-level propia** — descartado por el usuario: es OSINT de buscadores, subordinado, no un área hermana.
- **Duplicar el recon de dominios/subdominios dentro de OSINT** — rompe DRY y crea dos verdades divergentes; se usa el patrón puente (Footprinting ↔ Redes/Protocolos ya sienta el precedente).
- **Mantener el tag plano `OSINT`** — pierde la capacidad de filtrar por dimensión en los `.base` y en el grafo, justo cuando el área va a tener 13 dimensiones.

## Consecuencias

- **Red-Team.base (Level 0) la recoge sola**: la vista "Áreas" filtra por regex de profundidad (`/^Red Team\/[^\/]+$/`), así que `OSINT.base` aparece sin tocar el Level 0. La fórmula `certificación` la clasifica en **"Sin certificación asociada"** (correcto: es auto-dirigida, como Evasión de perímetro o los libros).
- **Migración**: 4 notas (`Footprinting/01,02,03`, `Fases/05`) cambian su tag plano `OSINT` al esquema jerárquico; `Search Engine Discovery` se recorta; se añaden **cross-links entrantes** desde recon web, footprinting, password attacks, AD externa y bug bounty hacia OSINT.
- **CLAUDE.md**: se añade OSINT (con Google Dorking) al mapa de estructura de `Red Team/` y a una tabla de área nueva; la familia de tags `OSINT/…` se documenta.
- **Temario**: OSINT puede entrar como **bloque de estudio** (no es certificación) — pendiente de decidir con el usuario.
- Build grande (~55-75 notas) ejecutado por cortes verticales sub-área a sub-área.
