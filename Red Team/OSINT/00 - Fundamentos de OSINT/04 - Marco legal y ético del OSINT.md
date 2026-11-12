---
tags:
  - OSINT/Fundamentos
  - Pentesting/Enumeracion
  - Tipo/Defensa
Descripción: "Qué es legal y qué no al hacer OSINT: Código Penal español, RGPD, ToS y jurisprudencia de scraping, y la autorización en un engagement"
Fecha de actualización: 2026-11-08
Nota previa: "[[03 - Taxonomía de fuentes y disciplinas OSINT]]"
Nota siguiente: "[[05 - Verificación, fiabilidad de fuentes y sesgos]]"
Area: "[[Fundamentos OSINT.base|Fundamentos OSINT]]"
---
---

"Es información pública, no es ilegal mirarla" es media verdad y una trampa. <mark style="background: #FF5582A6;">Mirar casi siempre es legal; **recopilar, almacenar y procesar** datos personales está regulado aunque sean públicos, y **acceder** a lo que está tras una barrera puede ser delito</mark>. El red teamer trabaja en el filo, así que conviene conocerlo.

# Código Penal español

Las fronteras relevantes para el OSINT ofensivo:

- <mark style="background: #ADCCFFA6;">**Art. 197 CP** (descubrimiento y revelación de secretos)</mark>: apoderarse de datos personales reservados o interceptar comunicaciones ajenas. Recopilar lo que alguien publicó voluntariamente no encaja; **usar datos sustraídos en una brecha** (art. 197.2, datos en ficheros) sí entra en terreno peligroso.
- <mark style="background: #ADCCFFA6;">**Art. 197 bis CP** (acceso ilícito a sistemas)</mark>: acceder sin autorización a un sistema de información vulnerando medidas de seguridad. El OSINT puro no accede a sistemas; pero **loguearse con credenciales filtradas**, entrar en un panel "oculto" o sortear un *login* **deja de ser OSINT y es acceso ilícito**.
- **Art. 197 ter CP**: fabricar o procurarse herramientas para lo anterior. **Art. 264 y ss.**: daños informáticos (no aplican al OSINT pasivo, pero marcan dónde acaba "solo mirar").

La línea mental: <mark style="background: #8000E1A6;">leer lo publicado = OSINT; franquear una barrera de seguridad o usar datos robados = delito</mark>.

# RGPD: recopilar datos personales es tratarlos

El punto que casi nadie interioriza: <mark style="background: #FFB86CA6;">bajo el `RGPD` (y la `LOPDGDD` española), recopilar datos personales —aunque sean públicos— es un *tratamiento*</mark>, y un tratamiento necesita base jurídica (art. 6 RGPD; en un pentest, el *interés legítimo* del cliente, documentado en el contrato). Que un correo esté en LinkedIn no lo convierte en libre: sigue siendo dato personal. En la UE esto es independiente del análisis estadounidense de *scraping*: el caso **Clearview AI** (multas de las APDs europeas en 2024-2025 por raspar fotos faciales públicas) confirma que **lo público no es "gratis"** cuando hay datos personales.

Consecuencia práctica en un engagement: **minimiza** (recoge solo lo que un PIR necesita), **documenta** la base jurídica y el alcance, y **destruye** el material OSINT al cerrar según el contrato ([[11 - Post-engagement - cierre, retest y retención|retención y destrucción]]).

# Scraping y Términos de Servicio: la jurisprudencia

El *scraping* automatizado choca con los `ToS` de las plataformas. La jurisprudencia (de EE. UU., pero marca tendencia) deja un cuadro matizado:

| Caso | Resolución | Lectura para el OSINT |
| - | - | - |
| **hiQ v. LinkedIn** (9º Circuito, 2022) | Raspar páginas **públicas** no viola la `CFAA`… pero hiQ **perdió** por incumplimiento de contrato y *trespass*: acabó con sentencia de 500.000 $, orden de cese y destrucción de datos | Público ≠ sin consecuencias: el `ToS` sigue siendo un contrato |
| **Meta v. Bright Data** (N.D. Cal., ene. 2024) | Raspar datos públicos **sin estar logueado** no infringe los `ToS` de Meta | Lo decisivo es **no autenticarse**: deslogueado y público pesa a favor |
| **X v. Bright Data** (2024) | Las demandas estatales por raspar posts públicos quedaron **desplazadas por el copyright** federal | El scraping de contenido público resiste; el *copyright* del contenido es otro eje |

<mark style="background: #FFB8EBA6;">El patrón: raspar lo **público** y **deslogueado** es defendible; raspar tras un *login* (aceptando el `ToS`) o datos no públicos es donde se pierde</mark>. Son sentencias de instancia o de un circuito, no doctrina universal — y en la UE manda el RGPD por encima de todo esto.

# En un engagement: autorización y alcance

El marco que convierte el OSINT en legítimo es el **contrato**:

- **Autorización escrita** del cliente, con el OSINT explícitamente dentro del alcance (`Rules of Engagement`). Ver [[03 - Pre-engagement I - contratos, NDA y scoping|pre-engagement y scoping]].
- **Alcance también geográfico y de terceros**: el OSINT arrastra datos de personas y empresas **fuera** del cliente (empleados, proveedores, familiares). Acótalo y evita recolectar sobre quien no autorizó.
- **Deconfliction**: si tu actividad pudiera confundirse con un ataque real o cruzarse con otra operación, hay un canal para aclararlo ([[03 - Coordinación de operadores y deconflicting|deconfliction]]).
- **La falacia del "solo estoy mirando"**: aunque no toques un sistema, acumular dossieres de personas es tratamiento de datos y, mal hecho, acoso. El propósito autorizado y la minimización son lo que te separan de eso.

> [!danger]+ La frontera que no se cruza
> El OSINT acaba donde empieza el acceso no autorizado. Probar una credencial filtrada contra el `login` del cliente **solo** si está en el alcance activo; usar datos de una brecha **solo** con base jurídica y para el objetivo pactado; nunca pivotar a sistemas, personas o empresas fuera del *scope*. En duda, pregunta al cliente antes de recolectar — no después.

> [!info]+ Fuentes
> `Código Penal` español (arts. 197, 197 bis, 197 ter, 264). `RGPD` (Reglamento UE 2016/679, art. 6) y `LOPDGDD` (LO 3/2018). Jurisprudencia: *hiQ v. LinkedIn* (9th Cir. 2022, acuerdo 2022), *Meta Platforms v. Bright Data* (N.D. Cal., 23-ene-2024), *X Corp. v. Bright Data* (N.D. Cal., 2024). Sanciones Clearview AI por APDs europeas (CNIL, Garante, HmbBfDI), 2022-2025.
