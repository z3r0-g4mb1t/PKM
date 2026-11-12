---
tags:
  - OSINT/Dorking
  - Pentesting/Enumeracion
  - Tipo/Defensa
Descripción: "Cómo evitar que te dorkeen y por qué el dorking es casi indetectable: hardening del índice, eliminación de contenido y auto-monitorización"
Fecha de actualización: 2026-11-08
Nota previa: "[[04 - Dorking más allá de Google]]"
Nota siguiente:
Area: "[[Google Dorking.base|Google Dorking]]"
---
---

El eje defensivo del dorking tiene una asimetría incómoda para el *blue team*: <mark style="background: #FF5582A6;">el dorking es casi **indetectable**, porque la consulta la recibe Google, no tú</mark>. No hay un log en tu servidor que diga "alguien me está dorkeando". La defensa, por tanto, no es detectar al atacante: es **no tener nada que encontrar** y **vigilarte a ti mismo**.

# Por qué no puedes detectar que te dorkean

Cuando alguien busca `site:acme.com inurl:admin`, la petición va a los servidores de Google. ACME no ve nada: ni una IP, ni un user-agent, ni una entrada de log. <mark style="background: #8000E1A6;">Solo verás tráfico cuando el atacante **visite** el resultado</mark> —y entonces es una visita HTTP normal, indistinguible—. La consecuencia operativa es clara: la defensa es **preventiva**, no reactiva.

# Hardening: que no haya nada que indexar

El objetivo es que Google nunca indexe lo sensible:

- <mark style="background: #FFB86CA6;">**No expongas lo que no debe ser público**</mark>: configs, backups, `.env`, `.git`, volcados de BBDD y paneles internos no deberían ser alcanzables desde Internet. Esto es higiene de despliegue, no SEO.
- **`noindex` de verdad**: para sacar algo del índice, usa la etiqueta `<meta name="robots" content="noindex">` o la cabecera `X-Robots-Tag: noindex`. Google la respeta y **retira** la página del índice en el siguiente rastreo.
- **Autenticación**: lo que está tras login no se indexa. Protege los paneles.

> [!danger]+ La trampa de robots.txt
> `robots.txt` **no oculta nada**: es público y solo *pide* a los crawlers que no rastreen. Peor aún, <mark style="background: #FF5582A6;">listar `Disallow: /admin/` en `robots.txt` le dice al atacante exactamente dónde mirar</mark> — es un mapa de tus rutas sensibles servido en `acme.com/robots.txt`. No lo uses para esconder; usa `noindex` y autenticación.

# Eliminar lo ya indexado

Si algo sensible ya está en el índice:

- **Google Search Console → herramienta de Eliminaciones (*Removals*)**: retira una URL del índice de Google (temporal); combínalo con quitar el contenido del servidor o ponerle `noindex` para que sea permanente.
- **Quita el contenido del origen**: si el fichero sigue vivo, volverá. Elimínalo o protégelo, **y** rota cualquier secreto que haya estado expuesto (asume que ya se ha filtrado).
- **Recuerda Wayback**: aunque lo saques de Google, puede seguir en [[13 - Web Archives|archivos históricos]]. Solicita su retirada aparte si es crítico.

# Auto-monitorización: dorkéate antes que ellos

La mejor defensa es <mark style="background: #ADCCFFA6;">correr los mismos dorks contra ti mismo, de forma periódica</mark>, y arreglar lo que aparezca:

- **Auto-dorking regular**: lanza las categorías de la [[03 - La Google Hacking Database y la automatización|GHDB]] contra tu dominio (vía Custom Search API) y revisa hallazgos.
- **Google Alerts**: crea alertas para `site:acme.com` combinado con términos sensibles (`password`, `confidential`, `index of`) — te avisa cuando aparezca algo nuevo.
- **Monitorización de superficie (ASM)**: las plataformas de *Attack Surface Management* incluyen detección de exposición por buscadores de forma continua.
- **Secretos en código**: audita tus repos con las herramientas de [[04 - Dorking más allá de Google|GitHub dorking]] (trufflehog, gitleaks) antes de que lo haga un atacante.

> [!success]+ El lado ofensivo de esto
> Para un red teamer, esta nota también es inteligencia: si el objetivo **no** hace nada de esto (y la mayoría no lo hace bien), el dorking seguirá siendo productivo. Y "dorkéate a ti mismo" es exactamente lo que harás **sobre el cliente** en la fase de recon — reportar la exposición indexada es un hallazgo de informe de alto valor y bajo esfuerzo.

> [!info]+ Fuentes
> `noindex` vs `robots.txt` y eliminación de URLs: documentación de Google Search Central (control de indexación, herramienta de Removals). Riesgo de `robots.txt` como mapa de rutas: consenso OWASP / práctica de pentest.
