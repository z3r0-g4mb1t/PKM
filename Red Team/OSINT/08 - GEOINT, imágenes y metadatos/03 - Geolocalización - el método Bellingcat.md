---
tags:
  - OSINT/Geoint
  - Pentesting/Enumeracion
Descripción: "Geolocalizar una foto sin GPS por su contenido: extraer pistas, acotar la región, consultar OSM con Overpass Turbo y confirmar con street view"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - Búsqueda inversa de imágenes y geolocalización por IA]]"
Nota siguiente: "[[04 - Cronolocalización, vídeo y satélite]]"
Area: "[[GEOINT e imágenes.base|GEOINT e imágenes]]"
---
---

Cuando no hay EXIF (la mayoría de fotos de RRSS) y la IA solo da una región, toca geolocalizar **por el contenido**. Es el método que popularizó Bellingcat: observación sistemática + bases de datos abiertas + confirmación visual. Parece magia; es método.

# Paso 1: extraer todas las pistas

<mark style="background: #ADCCFFA6;">Exprime la imagen buscando señales que acoten el lugar</mark>:

- **Texto y señales**: carteles, nombres de comercios, matrículas, números de teléfono (prefijo → país/región), **idioma** (el discriminador más fuerte).
- **Arquitectura**: estilo de edificios, materiales, tejados, postes de luz, bolardos, señales de tráfico (varían por país).
- **Vegetación y clima**: tipo de árboles, aridez, nieve → latitud y estación.
- **Infraestructura**: marca de gasolinera, mobiliario urbano, modelo de autobús, lado de circulación (izquierda/derecha).
- **Topografía**: montañas, costa, ríos al fondo → rasgos que casar con el mapa.
- **Sol y sombras**: dirección → orientación (y, con la fecha, [[04 - Cronolocalización, vídeo y satélite|hora]]).

# Paso 2: acotar la región

Combina las pistas para estrechar: idioma español + arquitectura mediterránea + matrícula con formato UE + vegetación seca → sur de España, por ejemplo. <mark style="background: #FFB8EBA6;">Cada pista elimina territorio</mark>. No necesitas identificar el punto aún, solo reducir el área a algo buscable.

# Paso 3: consultar OpenStreetMap con Overpass Turbo

<mark style="background: #ADCCFFA6;">`Overpass Turbo` consulta la base de datos de OpenStreetMap por **características concretas**</mark>. Si en la foto ves un rasgo singular (una iglesia de dos torres, un puente concreto, una marca de gasolinera específica), lo buscas en el área acotada:

```text
// Overpass Turbo: todas las gasolineras de una marca en un área (bbox)
node["amenity"="fuel"]["brand"="Repsol"]({{bbox}});
out;
```

Esto convierte "una gasolinera Repsol junto a una rotonda" en una lista finita de candidatos sobre el mapa. Igual con hospitales, iglesias, torres, estaciones — cualquier `tag` de OSM.

# Paso 4: confirmar visualmente

Con los candidatos, confirma sobre el terreno virtual:

- **Vista satélite** (Google Earth, Bing Maps): casa la disposición de edificios, carreteras y topografía desde arriba.
- **Vista de calle**: **Google Street View**, **Mapillary** (imágenes de calle colaborativas, cubre sitios que Google no) y **Yandex Panoramas** (mejor en Europa del Este/Rusia). Comparas la foto con la calle real hasta que **todo encaja**: el edificio, la farola, el bordillo.

<mark style="background: #8000E1A6;">La confirmación es cuando varios elementos independientes coinciden</mark> — no uno. Un edificio parecido no basta; el edificio + la farola + la señal + la montaña del fondo, sí.

> [!example]+ Mini-flujo
> Foto de un empleado "en la pausa". Pistas: cartel en alemán, tranvía amarillo, torre de iglesia barroca al fondo. → región: Baviera/Austria. → Overpass Turbo: iglesias barrocas + red de tranvía → pocas ciudades. → Street View en los candidatos hasta casar la torre y el cruce → **coordenadas exactas de la oficina**. Tiempo: minutos, no horas, gracias a acotar bien.

# Para el red teamer

Geolocalizar la sede, un centro de datos o el domicilio de un directivo alimenta el **recon físico** y la ingeniería social ("vi que estuviste en [sitio]..."). Y geolocalizar a partir de fotos de RRSS de empleados sitúa domicilios y rutinas — con todas las [[04 - Marco legal y ético del OSINT|cautelas legales]] sobre datos personales.

> [!info]+ Fuentes
> Método de geolocalización: [Bellingcat toolkit](https://bellingcat.gitbook.io/toolkit) y su guía de geolocalización. Herramientas: [Overpass Turbo](https://overpass-turbo.eu) sobre OpenStreetMap, Google Earth/Street View, [Mapillary](https://www.mapillary.com).
