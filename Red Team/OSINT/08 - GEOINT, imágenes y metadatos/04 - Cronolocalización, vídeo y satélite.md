---
tags:
  - OSINT/Geoint
  - Pentesting/Enumeracion
Descripción: "Datar una imagen por sombras (SunCalc, Shadow Finder) y clima, analizar vídeo frame a frame y usar imagen satélite histórica"
Fecha de actualización: 2026-11-08
Nota previa: "[[03 - Geolocalización - el método Bellingcat]]"
Nota siguiente:
Area: "[[GEOINT e imágenes.base|GEOINT e imágenes]]"
---
---

Geolocalizar responde al "dónde"; la **cronolocalización** responde al "cuándo", y el vídeo y la imagen satélite amplían ambos. Es lo que convierte "una foto en esta oficina" en "una foto en esta oficina, un martes de marzo por la tarde".

# Cronolocalización: datar por las sombras

<mark style="background: #ADCCFFA6;">La posición del sol en una fecha y lugar es física predecible: las sombras delatan la hora</mark>, y su longitud, la latitud.

- **SunCalc.org**: modela la posición del sol para cualquier lugar y fecha. Si ya tienes el **sitio** (de la [[03 - Geolocalización - el método Bellingcat|geolocalización]]), ajustas la fecha/hora hasta que la **dirección de la sombra** en SunCalc coincide con la de la foto → tienes la hora.
- **Shadow Finder** (Bellingcat): el problema inverso — dada la **longitud de una sombra** respecto al objeto y la fecha, calcula las **latitudes posibles**. Útil cuando no sabes el lugar pero sí la fecha.
- **Vegetación y clima en la imagen**: hojas/nieve/follaje → estación; sirve de comprobación cruzada.

<mark style="background: #FFB8EBA6;">La clave: sombra + fecha + lugar son tres variables ligadas</mark> — con dos, deduces la tercera.

# Datar por el clima

<mark style="background: #ADCCFFA6;">El tiempo que hacía es un registro histórico consultable</mark>. Si una foto muestra lluvia, niebla o nieve y tienes el lugar, cruza con el histórico meteorológico (Meteostat, Wolfram Alpha, registros de aeropuerto METAR) para confirmar o descartar una fecha. Una foto "de ayer" que muestra sol cuando ese día diluvió en esa ciudad es sospechosa.

# Análisis de vídeo

El vídeo es muchas imágenes más audio — más superficie:

- **Frame a frame**: extrae fotogramas y geolocaliza/data los más informativos.

```shell-session
# Extraer un fotograma por segundo de un vídeo para analizarlos
$ ffmpeg -i video.mp4 -vf fps=1 frame_%04d.png
```

- **Audio**: idioma, acento, campanas, megafonía, himnos, tráfico → país, tipo de lugar, evento.
- **Reflejos y detalles**: en escaparates, gafas, pantallas — a veces revelan lo que la cámara no enfocaba.
- **Verificación de vídeo**: la extensión **InVID/WeVerify** extrae *keyframes*, hace búsqueda inversa de cada uno, lee metadatos y detecta reedición — el estándar para verificar y rastrear un vídeo.

# Imagen satélite (y su histórico)

Para confirmar una ubicación desde arriba y **datar cambios**:

- **Google Earth**: tiene un **control de histórico** que muestra la misma zona en distintas fechas — ves cuándo se construyó un edificio, se pavimentó un parking o apareció/desapareció algo. Clave para datar.
- **Copernicus / Sentinel Hub** (gratis, UE): imágenes recientes y periódicas, buenas para cambios a gran escala.
- **Planet** (comercial): alta resolución y cadencia diaria.

<mark style="background: #8000E1A6;">El histórico satélite data construcciones e infraestructura</mark>: útil para recon físico (¿cuándo se amplió la sede?, ¿hay obra nueva sin vigilancia?) y para verificar la antigüedad de una imagen.

# Cierre de GEOINT

Con las cuatro notas tienes el ciclo completo de una imagen: [[01 - Metadatos y EXIF|metadatos]] → [[02 - Búsqueda inversa de imágenes y geolocalización por IA|inversa + IA]] → [[03 - Geolocalización - el método Bellingcat|geolocalización por contenido]] → cronolocalización. Para el red teamer, esto sitúa personas e instalaciones en el espacio y el tiempo — inteligencia para recon físico, ingeniería social y verificación.

> [!info]+ Fuentes
> Cronolocalización por sombras: [SunCalc.org](https://www.suncalc.org) y [Bellingcat ShadowFinder](https://github.com/bellingcat/ShadowFinder). Verificación de vídeo: [InVID/WeVerify](https://www.invid-project.eu/tools-and-services/invid-verification-plugin/). Satélite: Google Earth (histórico), [Copernicus/Sentinel](https://www.copernicus.eu). Clima histórico: Meteostat, METAR.
