---
tags:
  - OSINT/Frameworks
  - Pentesting/Enumeracion
Descripción: "Encadenar herramientas CLI en un pipeline de recon, usar IA para correlacionar sin que alucine, y la deconfliction y el reporting del OSINT"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - recon-ng y Maltego]]"
Nota siguiente:
Area: "[[Frameworks OSINT.base|Frameworks OSINT]]"
---
---

Cierra la automatización lo que la une todo: encadenar herramientas en un pipeline, apoyarse en IA sin que te traicione, y convertir el volcado en inteligencia reportable sin pisar otra operación.

# El pipeline de CLI

<mark style="background: #ADCCFFA6;">El patrón moderno de recon no es un framework monolítico, sino herramientas Unix especializadas encadenadas por tuberías</mark> — cada una hace una cosa y pasa su salida a la siguiente. Es el modelo de la suite [[Tools.base|ProjectDiscovery]]:

```shell-session
# Subdominios → resolver vivos → sondear HTTP → escanear con plantillas
$ subfinder -d acme.com -silent \
    | dnsx -silent \
    | httpx -silent \
    | nuclei -t exposures/
```

<mark style="background: #FFB86CA6;">Cada etapa filtra y enriquece</mark>: `subfinder` enumera subdominios (pasivo), `dnsx` se queda con los que resuelven, `httpx` con los que sirven web, `nuclei` busca exposiciones conocidas. El mismo patrón sirve para ASN→IPs ([[02 - ASN, BGP y mapeo de rangos IP|asnmap | naabu]]) o para búsqueda en motores de dispositivos (`uncover`). Es componible, scriptable y reejecutable para **monitorización continua** (combinado con `notify` para alertas).

> [!info]+ El detalle de cada herramienta, en Tools/
> `subfinder`, `dnsx`, `httpx`, `naabu`, `asnmap`, `uncover`, `notify` tienen su referencia en [[Tools.base|ProjectDiscovery]]. Aquí importa el **patrón de orquestación**: el pipeline, no la herramienta suelta.

# OSINT asistido por IA (2026)

Los LLMs han entrado en el flujo OSINT para lo que antes era trabajo manual tedioso:

- **Resumir** grandes volúmenes (cientos de posts, un leak grande) a lo relevante.
- **Extraer entidades** (nombres, correos, organizaciones) de texto no estructurado.
- **Correlacionar y traducir** contenido en otros idiomas.
- **Agentes de IA** que orquestan varias herramientas y razonan sobre los resultados (la tendencia emergente de 2026).

> [!danger]+ La IA alucina: nunca es fuente
> <mark style="background: #FF5582A6;">Un LLM puede inventar un dato con total aplomo</mark> — un correo, una relación, una ubicación que no existen. **La salida de IA es una hipótesis a verificar, jamás un hecho reportable.** Dos reglas: (1) todo lo que diga la IA se corrobora contra una [[00 - Qué es OSINT y por qué da ventaja|fuente real]] antes de usarlo; (2) **no metas datos del objetivo en LLMs de terceros** sin pensar en la privacidad — estás enviando datos personales (RGPD) y, posiblemente, material del cliente a un proveedor externo. Usa modelos locales o con garantías para material sensible.

# Deconfliction

<mark style="background: #ADCCFFA6;">Un recon automatizado a gran escala puede confundirse con un ataque real</mark> o cruzarse con otra operación (otro equipo, un incidente en curso). La *deconfliction* es el canal acordado para aclarar "esto soy yo, autorizado" y evitar una respuesta a incidentes innecesaria. En un engagement va en las reglas de enfrentamiento → [[03 - Coordinación de operadores y deconflicting|deconflicting]].

# Reporting: de volcado a inteligencia

El output de toda esta maquinaria no es un informe. Convertirlo en inteligencia reportable:

- **Deduplicar y normalizar**: los frameworks repiten hallazgos; consolida.
- **Atribuir la fuente** de cada dato (dónde, cuándo) — sin procedencia, no es verificable ([[01 - El ciclo de inteligencia aplicado al OSINT|ciclo de inteligencia]]).
- **Priorizar por impacto**: credenciales filtradas y activos expuestos arriba; curiosidades, abajo.
- **Enmarcar como exposición**: el entregable OSINT en un informe es "esto es lo que un atacante ve de ti y por dónde entraría". Conecta con el [[06 - Cómo redactar un hallazgo|reporting del pentest]].

<mark style="background: #8000E1A6;">La automatización recolecta; tú produces la inteligencia</mark>. Esa última milla —verificar, correlacionar, priorizar, reportar— es lo que no se automatiza y lo que distingue al profesional.

> [!info]+ Fuentes
> Patrón de pipeline: [ProjectDiscovery](https://github.com/projectdiscovery) (subfinder/dnsx/httpx/nuclei/notify). OSINT con IA 2026: [geoseeer, *The AI Agent Era of OSINT*](https://geoseeer.com/blog/emerging-osint-tools-ai-agent-era) (tratar los claims con cautela). Deconfliction: [[03 - Coordinación de operadores y deconflicting]].
