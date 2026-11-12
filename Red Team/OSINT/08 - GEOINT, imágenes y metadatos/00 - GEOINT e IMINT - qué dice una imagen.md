---
tags:
  - OSINT/Geoint
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Por qué una imagen es la fuente OSINT más rica: las dos preguntas (qué contiene / dónde y cuándo) y el valor para el red teamer"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Metadatos y EXIF]]"
Area: "[[GEOINT e imágenes.base|GEOINT e imágenes]]"
---
---

Una imagen suele decir más que mil posts. <mark style="background: #ADCCFFA6;">El `GEOINT` (inteligencia geoespacial) y el `IMINT` (inteligencia de imágenes) extraen de fotos y vídeos **dónde**, **cuándo** y **qué** — a menudo sin que el autor lo pretendiera</mark>. En redes sociales, cada foto de un empleado es un candidato a geolocalización y a fuga de metadatos.

# Las dos preguntas de una imagen

Toda imagen se interroga en dos planos, con técnicas distintas:

1. <mark style="background: #FFB8EBA6;">**¿Qué contiene?**</mark> (IMINT) — qué se ve: una pantalla con datos, un badge de acceso, una matrícula, una pizarra, un uniforme, el interior de una oficina. Observación directa + [[02 - Búsqueda inversa de imágenes y geolocalización por IA|búsqueda inversa]] para identificar objetos, personas o la fuente original.
2. <mark style="background: #FFB8EBA6;">**¿Dónde y cuándo se tomó?**</mark> (GEOINT) — por los **metadatos** ([[01 - Metadatos y EXIF|EXIF/GPS]]) si están, y si no, por el **contenido** ([[03 - Geolocalización - el método Bellingcat|geolocalización]]) y las **sombras/clima** ([[04 - Cronolocalización, vídeo y satélite|cronolocalización]]).

# Por qué importa para el red teamer

<mark style="background: #FFB86CA6;">Las imágenes filtran lo que ninguna política de seguridad controla</mark>:

- **Metadatos**: una foto subida sin limpiar lleva GPS (la casa del CEO), el modelo de cámara/móvil y el software.
- **Fondo**: una foto "de equipo" en la oficina expone pantallas con datos, pizarras con arquitectura, badges (clonables), la distribución física (recon para acceso físico).
- **Geolocalización de personal**: las fotos de Strava/Instagram sitúan domicilios y rutinas ([[05 - De un dato a la persona - pivote completo|pivote de personas]]).
- **Verificación**: confirmar si una imagen es real, antigua o generada por IA ([[05 - Verificación, fiabilidad de fuentes y sesgos|verificación]]).

# El cambio de 2026: la geolocalización por IA

<mark style="background: #8000E1A6;">Hasta hace poco, geolocalizar una foto sin GPS era trabajo manual de horas</mark>. En 2026 una nueva generación de modelos (GeoSpy, GeoSeer) **lee los píxeles** —terreno, arquitectura, vegetación— y propone una región candidata **en segundos, sin EXIF**. No son infalibles, pero cambian el flujo: ahora la IA da la hipótesis y el método manual la **confirma**. Se trata en [[02 - Búsqueda inversa de imágenes y geolocalización por IA]].

# El orden de trabajo

1. **Metadatos primero** (si los hay, es la respuesta directa) → [[01 - Metadatos y EXIF]].
2. **Si no hay EXIF**: IA de geolocalización + búsqueda inversa para una hipótesis rápida → [[02 - Búsqueda inversa de imágenes y geolocalización por IA]].
3. **Confirmar manualmente** con el método de geolocalización (landmarks, OSM, street view) → [[03 - Geolocalización - el método Bellingcat]].
4. **Datar** la imagen por sombras, clima y contexto → [[04 - Cronolocalización, vídeo y satélite]].

> [!warning]+ Una imagen se puede falsificar
> El EXIF se altera, y en 2026 una imagen entera puede ser sintética. Ninguna pista aislada prueba nada: GEOINT serio es **corroboración** de varias señales (metadatos + contenido + sombras + fuente). Aplica siempre el [[05 - Verificación, fiabilidad de fuentes y sesgos|método de verificación]].

> [!info]+ Fuentes
> Metodología de verificación de imagen/vídeo y geolocalización: [Bellingcat toolkit](https://bellingcat.gitbook.io/toolkit); [Authentic8, *Geolocation 101*](https://authentic8.com/blog/geolocation-101-image-based-tips-and-inspiration-unlikely-places/). IA de geolocalización 2026: [geoinfer, *AI OSINT Geolocation Tools 2026*](https://geoinfer.com/en/blog/ai-osint-geolocation-tools-2026).
