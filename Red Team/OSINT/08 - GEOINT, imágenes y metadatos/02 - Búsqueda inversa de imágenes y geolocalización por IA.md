---
tags:
  - OSINT/Geoint
  - Pentesting/Enumeracion
Descripción: "Qué motor de búsqueda inversa usar (Yandex, Lens, TinEye), reconocimiento facial y la nueva geolocalización por IA como generador de hipótesis"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Metadatos y EXIF]]"
Nota siguiente: "[[03 - Geolocalización - el método Bellingcat]]"
Area: "[[GEOINT e imágenes.base|GEOINT e imágenes]]"
---
---

Sin metadatos, el primer paso rápido es preguntar a las máquinas: ¿ha visto alguien esta imagen antes, o puede la IA decirme dónde se tomó? Es la **hipótesis**; la confirmación manual viene después ([[03 - Geolocalización - el método Bellingcat|método Bellingcat]]).

# Búsqueda inversa: qué motor para qué

Cada motor tiene fortalezas distintas. <mark style="background: #ADCCFFA6;">Pásale la imagen a varios: sus índices no coinciden</mark>.

| Motor | Fuerte en | Uso |
| - | - | - |
| **Yandex** | Caras, lugares, arquitectura, contenido no anglosajón | <mark style="background: #FFB86CA6;">El mejor para personas y geolocalización</mark> |
| **Google Lens** | Objetos, texto (OCR), *landmarks* famosos | Identificar qué hay en la foto |
| **TinEye** | Coincidencias **exactas**, imagen **original** y más antigua | Rastrear dónde y desde cuándo circula |
| **Bing Visual Search** | Objetos y productos | Segunda opinión |

<mark style="background: #8000E1A6;">Yandex es, desde hace años, sorprendentemente superior para caras y lugares</mark> — si solo vas a usar uno para geolocalizar una persona o un sitio, es ese. TinEye no geolocaliza, pero te dice si una foto de perfil "nueva" en realidad lleva años circulando (señal de sock puppet ajeno o de imagen robada).

# Reconocimiento facial

<mark style="background: #FFB86CA6;">`PimEyes` y `FaceCheck.ID` buscan **otras fotos de la misma cara** por toda la web</mark> — no por nombre, por rostro. Dada una foto de un empleado, encuentran sus demás apariciones (otros perfiles, fotos de eventos, prensa), un pivote potentísimo hacia identidades que el objetivo creía separadas.

> [!danger]+ Reconocimiento facial: zona roja legal y ética
> Buscar una cara es tratar un **dato biométrico**, la categoría más protegida del [[04 - Marco legal y ético del OSINT|RGPD]] (art. 9). El caso Clearview AI (multas en toda la UE) marca el límite: raspar y buscar caras de personas sin base jurídica es sancionable. En un engagement, úsalo solo con autorización explícita y para los objetivos en alcance. Fuera de eso, no lo toques.

# Geolocalización por IA: la novedad de 2026

<mark style="background: #ADCCFFA6;">Una nueva clase de herramientas **lee los píxeles** (terreno, arquitectura, vegetación, estilo) y propone una **región candidata en segundos, sin necesidad de EXIF**</mark>:

- **GeoSpy / GeoSpyAI**: predice la ubicación probable por los rasgos del paisaje y la construcción.
- **GeoSeer** (finales de 2025): afirma devolver coordenadas, ciudad, país y a veces precisión a nivel de calle desde una sola imagen.
- **Picarta**: en la misma línea.

<mark style="background: #FF5582A6;">Trátalas como **generador de hipótesis**, no como respuesta</mark>: pueden acertar la región en segundos, pero también equivocarse con aplomo. El flujo correcto en 2026:

```text
imagen sin EXIF
   → IA de geolocalización  → "probablemente sur de Alemania, casco antiguo"
   → búsqueda inversa        → ¿aparece ya identificada?
   → confirmación manual      → landmarks + OSM + street view  → coordenadas exactas
```

La IA y la búsqueda inversa reducen el espacio de búsqueda; el [[03 - Geolocalización - el método Bellingcat|método manual]] confirma. Nunca reportes una ubicación solo porque lo dijo la IA.

> [!info]+ Fuentes
> Motores de búsqueda inversa y su comparativa: [Bellingcat toolkit](https://bellingcat.gitbook.io/toolkit); [Authentic8, *Geolocation 101*](https://authentic8.com/blog/geolocation-101-image-based-tips-and-inspiration-unlikely-places/). IA de geolocalización (GeoSpy, GeoSeer) 2026: [geoinfer](https://geoinfer.com/en/blog/ai-osint-geolocation-tools-2026) (claims sin verificación independiente). Reconocimiento facial y RGPD: caso Clearview AI.
