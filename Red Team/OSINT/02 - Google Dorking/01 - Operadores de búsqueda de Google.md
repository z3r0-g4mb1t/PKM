---
tags:
  - OSINT/Dorking
  - Pentesting/Enumeracion
Descripción: "Referencia de operadores de búsqueda de Google (2026): qué hace cada uno, ejemplos, cuáles ya no funcionan y cómo combinarlos"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - Qué es el Google Dorking]]"
Nota siguiente: "[[02 - Dorks por objetivo]]"
Area: "[[Google Dorking.base|Google Dorking]]"
---
---

Los operadores son el lenguaje de precisión del buscador. Dominarlos es la diferencia entre "buscar en Google" y hacer *dorking*. Esta es la referencia **actualizada a 2026** —con los que han muerto marcados— y cómo encadenarlos.

# Operadores de alcance (los más usados en recon)

| Operador | Qué hace | Ejemplo |
| - | - | - |
| `site:` | Limita a un dominio o subdominio | `site:acme.com` · `site:*.acme.com` |
| `inurl:` | Un término aparece en la URL | `inurl:admin` · `inurl:login.php` |
| `allinurl:` | **Todos** los términos en la URL | `allinurl:admin panel` |
| `intitle:` | Un término en el `<title>` de la página | `intitle:"index of"` |
| `allintitle:` | **Todos** los términos en el título | `allintitle:admin login` |
| `intext:` | Un término en el cuerpo | `intext:"password"` |
| `allintext:` | **Todos** los términos en el cuerpo | `allintext:username password` |
| `filetype:` / `ext:` | Tipo de fichero por extensión | `filetype:pdf` · `ext:sql` |

<mark style="background: #ADCCFFA6;">`site:` es el operador que convierte el dorking en recon dirigido</mark>: acota cualquier búsqueda a tu objetivo. Casi todos tus dorks empezarán por él.

# Operadores lógicos y de refinado

| Operador | Qué hace | Ejemplo |
| - | - | - |
| `" "` | Frase exacta | `"internal use only"` |
| `-` | Excluye un término | `site:acme.com -www` |
| `OR` / `\|` | Cualquiera de los términos | `inurl:login OR inurl:signin` |
| `( )` | Agrupa | `site:acme.com (inurl:admin OR inurl:login)` |
| `*` | Comodín: cualquier palabra | `"acme * password"` |
| `..` | Rango numérico | `"precio" 100..500` |
| `before:` / `after:` | Filtrado por fecha | `site:acme.com after:2025-01-01` |

<mark style="background: #FFB8EBA6;">`OR` debe ir en mayúsculas; `|` es su equivalente</mark>. El `-` pegado al término (`-www`, no `- www`) lo excluye. `before:`/`after:` son oro para encontrar contenido reciente o acotar una ventana temporal.

# Operadores muertos (no los uses)

> [!fail]+ Ya no funcionan en Google
> - `cache:` — **retirado en 2024**. Google eliminó el acceso a la versión cacheada. Para ver el pasado de un sitio, usa [[13 - Web Archives|Wayback Machine]].
> - `link:` — muerto hace años (buscaba páginas que enlazan a una URL).
> - `info:` — retirado (información sobre una URL).
> - `+` (forzar término) y `~` (sinónimos) — retirados hace tiempo; hoy se usa `" "` para forzar.

# Fiabilidad: Google no siempre te hace caso

<mark style="background: #FF5582A6;">Google **ignora silenciosamente** algunos operadores en parte de los resultados</mark>, sin avisar. En pruebas recientes (2026): `site:` e `intitle:` se respetan casi al 100 %, pero `filetype:` solo acierta en ~73 % y `inurl:` es inconsistente. Consecuencias prácticas:

- **No confíes en un solo dork**: si `filetype:sql` no da nada, reformula (`inurl:.sql`, `intitle:"index of" sql`).
- **Verifica a mano** los resultados: algunos no cumplirán el operador que pediste.
- **El orden y la especificidad importan**: cuanto más concreto el dork, menos ruido.

# Combinar: de operador a dork

El poder está en encadenar. Un ejemplo construido paso a paso para encontrar paneles de login del objetivo:

```text
# 1. Acotar al objetivo
site:acme.com
# 2. Buscar rutas de login en la URL
site:acme.com inurl:login
# 3. Ampliar a variantes con OR y agrupación
site:acme.com (inurl:login OR inurl:admin OR inurl:signin OR inurl:portal)
# 4. Excluir ruido conocido (la web pública de marketing)
site:acme.com (inurl:login OR inurl:admin) -inurl:blog -inurl:news
```

Otro, para ficheros expuestos:

```text
# Documentos y hojas de cálculo del objetivo
site:acme.com (filetype:pdf OR filetype:xlsx OR filetype:docx)
# Ficheros de configuración y backups
site:acme.com (ext:conf OR ext:cnf OR ext:env OR ext:sql OR ext:bak)
# Listados de directorio navegables
site:acme.com intitle:"index of"
```

<mark style="background: #8000E1A6;">La metodología: empieza amplio (`site:`), añade un eje (URL/título/texto/fichero), amplía con `OR`, y resta ruido con `-`</mark>. Itera. Las recetas completas por tipo de objetivo están en [[02 - Dorks por objetivo]].

> [!info]+ Fuentes
> Operadores vigentes y retirados verificados contra pruebas de 2026: [SEOmator](https://seomator.com/blog/google-search-operators-cheat-sheet), [Admin365](https://admin365.blog/2026/07/14/google-search-operators-that-still-work-in-2026-and-the-ones-that-dont/). Documentación oficial: Google Search Central (operadores soportados). Retirada de `cache:` (2024) ya documentada en [[12 - Search Engine Discovery]].
