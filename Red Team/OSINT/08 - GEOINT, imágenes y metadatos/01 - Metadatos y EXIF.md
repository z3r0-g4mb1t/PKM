---
tags:
  - OSINT/Geoint
  - Pentesting/Enumeracion
Descripción: "Extraer GPS, dispositivo y fechas de una imagen con exiftool, qué metadatos llevan los documentos, y limpiarlos antes de subir algo tú"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - GEOINT e IMINT - qué dice una imagen]]"
Nota siguiente: "[[02 - Búsqueda inversa de imágenes y geolocalización por IA]]"
Area: "[[GEOINT e imágenes.base|GEOINT e imágenes]]"
---
---

Los metadatos son la respuesta directa al "¿dónde y cuándo?" — cuando existen. Una foto con GPS incrustado te da las coordenadas exactas sin más trabajo. Y los documentos que publica el objetivo llevan su propia carga de metadatos reveladores.

# exiftool: la navaja suiza de los metadatos

<mark style="background: #ADCCFFA6;">`exiftool` (de Phil Harvey) lee y escribe metadatos de casi cualquier formato</mark> —imágenes, PDF, Office, vídeo—. Es el estándar. Rama **13.x** en 2026 (verifica la versión en exiftool.org).

```shell-session
# Volcar TODOS los metadatos de una imagen
$ exiftool foto.jpg

# Solo coordenadas GPS y fecha de captura (lo que más importa)
$ exiftool -gps:all -createdate foto.jpg
$ exiftool -gpslatitude -gpslongitude foto.jpg

# Procesar un directorio entero (fotos descargadas de un perfil)
$ exiftool -gps:all -filename -createdate -r ./descargas/
```

<mark style="background: #FFB86CA6;">Si aparece `GPS Position`, cópialo a Google Maps y tienes el lugar exacto</mark>. De una foto de Instagram de un empleado puedes sacar su domicilio; de una foto corporativa, la sede o una ubicación interna.

Más allá del GPS, el EXIF revela:

- **Dispositivo**: marca y modelo de cámara/móvil (`Make`, `Model`) → perfila a la persona.
- **Software**: con qué se editó (`Software`) → pistas del entorno.
- **Fechas**: `CreateDate`, `ModifyDate` → cuándo se tomó y editó.
- **Miniatura incrustada**: a veces la *thumbnail* conserva una versión **anterior** al recorte/edición — puede mostrar lo que se quiso ocultar.

# Metadatos en documentos

<mark style="background: #ADCCFFA6;">Los PDF y documentos de Office que publica el objetivo son una mina</mark>. `exiftool informe.pdf` o `documento.docx` suele revelar:

- **Autor y organización** (`Author`, `Creator`) → nombres de usuario de red, nombres reales.
- **Rutas internas** incrustadas → estructura de carpetas, nombres de servidor.
- **Software y versiones** → pila ofimática, a veces el SO.

Esto conecta con el [[02 - Dorks por objetivo#Documentos y sus metadatos|dorking de documentos]]: dorkeas los PDF/XLS del dominio, los descargas y les sacas los metadatos. Una herramienta clásica para esto a escala es **FOCA** (extrae y agrega metadatos de todos los documentos de un dominio), y `exiftool` en bucle hace lo mismo desde consola.

# El filo para ti: limpia antes de subir

<mark style="background: #FF5582A6;">Lo mismo que explotas en el objetivo te delata a ti</mark>. Cualquier imagen o documento que subas (a un sock puppet, a un informe, a un repo) lleva **tus** metadatos. Límpialos:

```shell-session
# Borrar TODOS los metadatos de un fichero (in-place, con backup *_original)
$ exiftool -all= foto.jpg
# Alternativa dedicada a la limpieza (respeta mejor algunos formatos)
$ mat2 foto.jpg          # Metadata Anonymisation Toolkit
```

Es parte del [[03 - Cómo no quemar la operación - fugas y errores comunes|OPSEC]]: una captura subida con GPS de tu casa arruina una operación entera.

> [!warning]+ El EXIF miente, y las redes lo borran
> Dos límites: (1) el EXIF se puede **fabricar o alterar** trivialmente — trátalo como **evidencia de apoyo**, no como prueba, y corrobóralo con el contenido. (2) <mark style="background: #FFB8EBA6;">La mayoría de las redes sociales **eliminan el EXIF al subir**</mark> (varía según la vía de subida). Por eso, para fotos de RRSS, el GPS casi nunca estará — y hay que recurrir a la [[03 - Geolocalización - el método Bellingcat|geolocalización por contenido]].

> [!info]+ Fuentes
> [ExifTool](https://exiftool.org) (Phil Harvey; rama 13.x 2026) y su [ficha en Bellingcat toolkit](https://bellingcat.gitbook.io/toolkit/more/all-tools/exiftool). Limpieza: [MAT2](https://0xacab.org/jvoisin/mat2). Agregación de metadatos de documentos: FOCA.
