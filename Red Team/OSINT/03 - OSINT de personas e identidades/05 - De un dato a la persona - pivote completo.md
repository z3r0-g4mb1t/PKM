---
tags:
  - OSINT/Personas
  - Pentesting/Enumeracion
Descripción: "Caso práctico de principio a fin: de un alias de foro a la persona completa y a un patrón de contraseña, encadenando y verificando cada salto"
Fecha de actualización: 2026-11-08
Nota previa: "[[04 - Buscadores de personas y registros públicos]]"
Nota siguiente:
Area: "[[OSINT de personas.base|OSINT de personas]]"
---
---

Las notas anteriores dan las piezas; el valor está en **encadenarlas**. Este es un pivote completo, ficticio pero realista, de un alias suelto a un perfil cerrado y a munición de acceso. Cada salto **se verifica** antes de seguir.

> [!example]+ Escenario
> Durante el recon de ACME encuentras, en un foro técnico, un post firmado por `n3onf0x` recomendando una librería que ACME usa. Hipótesis: es un desarrollador de ACME. Objetivo: identificar a la persona y evaluar si da acceso.

# Paso 1 — Alias → cuentas

Enumeras el alias en cientos de plataformas:

```shell-session
$ maigret n3onf0x --html --extract
```

Resultado (verificado abriendo cada perfil con el [[02 - Infraestructura de investigación - VM, red y navegador|navegador compartimentado]]): `github.com/n3onf0x`, un Reddit y un Keybase, **los tres con la misma foto de avatar** → misma persona (corroboración por imagen). Dos cuentas más con avatar distinto se descartan (homonimia).

# Paso 2 — GitHub → correo real

El perfil de GitHub es el salto de oro. <mark style="background: #FFB86CA6;">Los commits filtran el correo del autor</mark> en sus metadatos:

```shell-session
$ git clone https://github.com/n3onf0x/dotfiles && cd dotfiles
$ git log --format='%an <%ae>' | sort -u
# Alex Rivera <alex.rivera@gmail.com>
# n3onf0x <n3onf0x@users.noreply.github.com>
```

Tienes **nombre real y correo personal**. (De paso, en otro repo encuentras un `.env` comiteado por error → pivote a [[Brechas y filtraciones.base|secretos]], fuera de este hilo.)

# Paso 3 — Correo → servicios y cuenta Google

```shell-session
$ holehe alex.rivera@gmail.com
# exists: Spotify, Twitter, Strava, ...
$ ghunt email alex.rivera@gmail.com
# Nombre: Alex Rivera · foto · reseñas de Google Maps
```

<mark style="background: #FF5582A6;">Las reseñas de Maps lo sitúan</mark>: reseña un gimnasio y una taquería en **Austin, TX** → ciudad de residencia (`B2`, corroborado por dos reseñas). `holehe` confirma Strava, que guardamos para el siguiente paso.

# Paso 4 — Cuentas → geolocalización y entorno

- **Strava** (de `holehe`): si sus rutas son públicas, el punto donde empiezan y acaban revela su **barrio** (ver [[GEOINT e imágenes.base|GEOINT]]).
- **Twitter/Reddit**: horarios de publicación (zona horaria), intereses (pretextos de phishing), menciones de ACME.

# Paso 5 — Correo → brechas → patrón de contraseña

```shell-session
# ¿Aparece el correo en filtraciones? (ver sub-área de brechas)
$ # HIBP / Dehashed: alex.rivera@gmail.com
# → brecha de 2019, contraseña en claro: "Austin2016!"
```

<mark style="background: #8000E1A6;">De aquí sale un **patrón** de contraseña</mark> (ciudad + año + símbolo). Si Alex trabaja en ACME, variantes como `Austin2026!` son candidatas de alto valor para un *password spraying* contra el portal de ACME — **siempre dentro del alcance activo** ([[Ataques a contraseñas.base|password attacks]]).

# El perfil cerrado

```text
n3onf0x  →  Alex Rivera
  ├── GitHub, Reddit, Keybase (misma foto)
  ├── alex.rivera@gmail.com  (de los commits)
  ├── Spotify, Twitter, Strava  (holehe)
  ├── Austin, TX  (reseñas de Maps + Strava)
  └── patrón de contraseña  (brecha 2019)  →  spraying
```

<mark style="background: #ADCCFFA6;">Partiendo de un alias suelto, en cinco saltos tienes identidad, ubicación, superficie social y un vector de acceso</mark> — todo pasivo, todo verificado.

> [!warning]+ Dónde parar
> Cada salto fue recolección pasiva y legal. **Actuar** sobre el patrón de contraseña (lanzar el *spray*) solo si el acceso activo a ACME está autorizado ([[04 - Marco legal y ético del OSINT|alcance]]). Y todo el dossier de Alex es dato personal bajo RGPD: justificado por el engagement, minimizado y destruido al cierre. Identificar ≠ atacar; recolectar ≠ retener para siempre.

> [!success]+ La lección del método
> El pivote funciona porque **cada identificador es la llave del siguiente** y porque **cada salto se corroboró** (misma foto, dos reseñas, correo en commits). Salta sin verificar y acabas construyendo el perfil de la persona equivocada. Este es el núcleo del OSINT de personas; las demás dimensiones (SOCMINT, brechas, GEOINT) son profundizaciones de pasos concretos de esta cadena.

> [!info]+ Fuentes
> Técnica de extracción de correo de commits: funcionamiento estándar de `git log`. Herramientas encadenadas: [[01 - Enumeración de usernames cross-platform|Maigret]], [[02 - OSINT de correo electrónico|holehe/GHunt]], [[Brechas y filtraciones.base|brechas]].
