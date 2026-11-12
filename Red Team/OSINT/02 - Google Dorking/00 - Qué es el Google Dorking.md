---
tags:
  - OSINT/Dorking
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Usar el buscador como arma de reconocimiento: qué es el Google Dorking, por qué funciona y qué puede destapar de un objetivo"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Operadores de búsqueda de Google]]"
Area: "[[Google Dorking.base|Google Dorking]]"
---
---

<mark style="background: #ADCCFFA6;">El `Google Dorking` (o `Google Hacking`) es el uso de operadores de búsqueda avanzados para destapar información sensible, vulnerabilidades o contenido oculto que un buscador ya ha indexado</mark>. El término viene del investigador Johnny Long, que a principios de los 2000 empezó a catalogar estas búsquedas en la **Google Hacking Database** (GHDB).

La idea es sencilla y potente: <mark style="background: #8000E1A6;">Google ya ha rastreado e indexado una porción enorme de la web por ti, incluida mucha que nadie pretendía exponer</mark>. En vez de escanear al objetivo (ruidoso, activo, detectable), le preguntas al índice de Google —que no toca al objetivo— por lo que ya sabe.

> [!info]+ Contexto en el vault
> El recon web ya introduce el *search engine discovery* en [[12 - Search Engine Discovery]]; esta sub-área lo desarrolla a fondo como técnica propia: operadores, recetas por objetivo, automatización, otros motores y defensa. Si vienes de allí, esta es la referencia completa.

# Por qué es tan rentable

- <mark style="background: #FFB8EBA6;">**Pasivo e invisible**</mark>: la consulta va a Google, no al objetivo. No deja rastro en los logs del objetivo (ver [[02 - Pasivo, semipasivo y activo - el gradiente de huella|gradiente de huella]]).
- **Coste cero, barrera cero**: no necesitas herramientas ni credenciales. Solo saber preguntar.
- **Encuentra lo que un escáner no ve**: un escáner prueba rutas que conoce; un dork encuentra lo que **alguien expuso y olvidó** —un backup, un `.env`, un panel interno— porque Google lo indexó al seguir un enlace.

<mark style="background: #FF5582A6;">Un `.sql` de backup, un `.env`, un `config.php` o un listado de directorio indexados son, a menudo, credenciales o lógica interna servidas en bandeja</mark>. Ahí está el valor: una sola consulta bien construida puede valer más que horas de escaneo.

# Qué se puede destapar

Las categorías clásicas (las mismas que organizan la GHDB):

- **Ficheros con credenciales** — configs, backups, volcados de BBDD, claves.
- **Páginas de login y paneles** — interfaces de administración expuestas.
- **Mensajes de error** — revelan rutas, versiones, consultas SQL, pila tecnológica.
- **Directorios navegables** — *directory listing* que expone la estructura del servidor.
- **Dispositivos conectados** — cámaras, impresoras, paneles IoT con interfaz web.
- **Documentos sensibles** — PDF/XLS/DOCX con datos internos (y sus metadatos).

El detalle operativo de cada categoría, con dorks listos, está en [[02 - Dorks por objetivo]].

# El matiz que define su alcance

<mark style="background: #FFB8EBA6;">Ves lo que Google **indexó**, no la realidad actual</mark>. Eso tiene dos caras:

- **Puede estar desfasado**: un resultado puede apuntar a algo ya retirado (aunque a menudo sigue en [[13 - Web Archives|Wayback]]).
- **No está todo**: lo protegido por `robots.txt`, `noindex`, autenticación o simplemente no enlazado no aparece. El dorking complementa, no sustituye, al [[00 - Reconocimiento web|recon activo]] y al [[Google Dorking.base|dorking en otros motores]] (cada uno indexa cosas distintas).

> [!warning]+ Dorkear es legal; actuar sobre el hallazgo puede no serlo
> Construir y lanzar dorks es una búsqueda normal: legal. Pero **acceder** a lo que encuentres cambia la ecuación: abrir un panel ajeno, usar credenciales indexadas o descargar una BBDD expuesta de un tercero puede ser acceso ilícito ([[04 - Marco legal y ético del OSINT|art. 197 bis CP]]). En un engagement, actúa solo dentro del alcance. Encontrar ≠ explotar.

La herramienta del dorking es su lenguaje: los **operadores**. Empezamos por ahí.

> [!info]+ Fuentes
> Origen del término y GHDB: Johnny Long, *Google Hacking for Penetration Testers*. Catálogo vivo: [Google Hacking Database (Exploit-DB / OffSec)](https://www.exploit-db.com/google-hacking-database).
