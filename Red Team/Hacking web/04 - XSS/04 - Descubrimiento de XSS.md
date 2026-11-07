---
tags:
  - Web/Red-Team
  - Pentesting/Enumeracion
  - XSS
Descripción: "Detectar una XSS puede ser tan difícil como explotarla"
Fecha de actualización: 2026-06-08
Nota previa: "[[03 - XSS basado en DOM]]"
Nota siguiente: "[[05 - Evasión y ofuscación de XSS]]"
Area: "[[XSS.base|XSS]]"
---
Detectar una XSS puede ser tan difícil como explotarla. Por suerte, al ser tan común, hay muchas herramientas y técnicas para encontrarla. **Tres enfoques**: <mark style="background: #FFB8EBA6;">automatizado, manual y revisión de código</mark>.

# Descubrimiento automatizado

Casi todos los escáneres de vulnerabilidades web (`Nessus`, `Burp Pro`, `ZAP`) detectan los tres tipos de *XSS* con dos modos: <mark style="background: #ADCCFFA6;">un passive scan que revisa el código cliente buscando DOM XSS, y un active scan que inyecta payloads para intentar disparar la ejecución</mark>. Las herramientas de pago aciertan más (sobre todo cuando hace falta evadir filtros), pero <mark style="background: #FF5582A6;">hay open-source útil</mark>: `XSStrike`, `BruteXSS`, `XSSer`.

```shell-session
$ python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"

[~] Checking for DOM vulnerabilities
[!] Testing parameter: task
[!] Reflections found: 1
[+] Payload: <HtMl%09onPoIntERENTER+=+confirm()>
[!] Efficiency: 100
```

> [!warning]+ Verifica siempre a mano
> Estas herramientas <mark style="background: #FFB86CA6;">inyectan payloads y comparan el código renderizado</mark> para ver si el payload aparece. <mark style="background: #FF5582A6;">Que el payload se refleje no garantiza que se ejecute</mark> (puede colarse en el *HTML* sin dispararse por mil razones). <mark style="background: #ADCCFFA6;">Un escáner genera falsos positivos</mark>; **confirma cada hallazgo manualmente antes de reportarlo**.

# Descubrimiento manual

Lo más básico es <mark style="background: #FFB86CA6;">probar payloads de listas conocidas</mark> ([PayloadsAllTheThings](https://github.com/swisskyrepo/payloadsallthethings), [Payload-Box](https://github.com/payload-box)) contra cada campo. Pero verás que la mayoría **no funcionan** aunque la app sea vulnerable, porque <mark style="background: #ADCCFFA6;">cada payload está escrito para un contexto de inyección concreto</mark> (romper tras una comilla, evadir un filtro) o un vector distinto (`<script>`, atributos `<img>`, estilos `CSS`).

> [!important]+ El contexto de inyección lo es todo
> La clave del descubrimiento manual no es la lista de payloads, sino <mark style="background: #FFB86CA6;">dónde aterriza tu entrada en la respuesta</mark>: en el cuerpo *HTML*, dentro de un atributo (`value="AQUÍ"`), dentro de un string de *JavaScript* (`var x = 'AQUÍ'`), en una URL… 
> Cada ubicación exige un payload que **rompa** ese contexto: cerrar el atributo con `">`, cerrar el string con `';`, etc. Un *polyglot* (un <mark style="background: #FFB8EBA6;">payload que funciona en varios contextos a la vez</mark>) ahorra trabajo. 
> Y recuerda: <mark style="background: #FF5582A6;">la XSS no vive solo en campos de formulario</mark> — <mark style="background: #ADCCFFA6;">cualquier entrada que se refleje vale</mark>, incluidas cabeceras HTTP como `Cookie` o `User-Agent` si su valor acaba en la página. El detalle de cada contexto y cómo romperlo se desarrolla en [[05 - Evasión y ofuscación de XSS]].

# Revisión de código

<mark style="background: #ADCCFFA6;">El método más fiable es la revisión manual de código</mark>, *front-end* y *back-end*. Si entiendes exactamente cómo se trata tu entrada hasta que llega al navegador (el [[03 - XSS basado en DOM|source y sink]]), escribes un payload a medida con alta confianza. En apps muy comunes (ya pasadas por escáneres antes de publicarse) la revisión de código suele ser la única vía de encontrar XSS que sobrevivieron al lanzamiento.

> [!info]+ Detección a escala en bug bounty
> Auditar un programa grande no se hace campo por campo, sino con un *pipeline*: **reunir parámetros** (`gau`/`waybackurls` sobre URLs históricas (ver [[13 - Web Archives|Web Archives]]) y [[19 - Fuzzing de parámetros y valores|fuzzing de parámetros]] para los no documentados), **filtrar reflejos** (`kxss`/`Gxss`), **confirmar ejecución** (`Dalfox`), y cubrir aparte **DOM** (`DOM Invader`) y **blind XSS** (`ezXSS`, sembrado en campos que vea personal interno —ver [[01 - XSS Almacenado]]). El instrumental completo y cuándo usar cada pieza está en [[06 - Herramientas para XSS]].

Localizar la vulnerabilidad rara vez basta: en producción el payload choca con filtros de entrada y WAFs que hay que **evadir** antes de que llegue a ejecutarse. Ese es el siguiente paso: [[05 - Evasión y ofuscación de XSS]].
