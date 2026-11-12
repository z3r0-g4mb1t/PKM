---
tags:
  - OSINT/Frameworks
  - Pentesting/Enumeracion
Descripción: "El agregador OSINT autónomo: de una semilla a un grafo correlacionado con 200+ módulos, cómo lanzarlo, el modo pasivo y su estado tras Intel 471"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - Automatización y frameworks - cuándo y por qué]]"
Nota siguiente: "[[02 - recon-ng y Maltego]]"
Area: "[[Frameworks OSINT.base|Frameworks OSINT]]"
---
---

<mark style="background: #ADCCFFA6;">SpiderFoot es el agregador OSINT autónomo de referencia: le das una **semilla** (dominio, IP, correo, nombre, teléfono) y, con más de 200 módulos, recolecta de cientos de fuentes y **correlaciona los resultados solo** en un grafo de entidades</mark>. Es la forma más rápida de pasar de "tengo un dominio" a "tengo el mapa completo de su superficie".

# Cómo funciona

Cada módulo consulta una fuente (DNS, WHOIS, Shodan, HIBP, redes sociales, brechas, CT logs…). Lo potente es que SpiderFoot **encadena** solo: un subdominio que encuentra un módulo lo pasa a los demás, que lo resuelven, lo buscan en Shodan, extraen sus certificados, etc. El resultado es un grafo donde los datos están **relacionados**, no en listas sueltas. Su motor de **correlación** marca lo interesante (credenciales expuestas, hosts vulnerables, coincidencias en brechas).

# Cómo lanzarlo

SpiderFoot se usa por su interfaz web (lo habitual) o por CLI:

```shell-session
# Interfaz web local (navegas a http://127.0.0.1:5001)
$ python3 sf.py -l 127.0.0.1:5001

# Escaneo por CLI: semilla + tipos de dato a buscar
$ python3 sf.py -s acme.com -t EMAILADDR,DOMAIN_NAME,IP_ADDRESS

# Listar módulos disponibles
$ python3 sf.py -M
```

En la web, al crear un escaneo eliges el **caso de uso**:

- **Footprint**: mapear toda la superficie de la organización (lo más completo, también lo más ruidoso).
- **Investigate**: centrado en una entidad concreta (una persona, un correo).
- **Passive**: <mark style="background: #FF5582A6;">solo fuentes que **no tocan al objetivo**</mark> — el modo para recon sigiloso y para no salirte de alcance.

# API keys: el multiplicador

<mark style="background: #FFB86CA6;">Muchos de los mejores módulos necesitan API key</mark> (Shodan, Censys, Hunter, HIBP, VirusTotal…). SpiderFoot funciona sin ellas, pero con las claves configuradas (en *Settings*) multiplica lo que ve. Configúralas con cuentas desechables, no las tuyas — SpiderFoot consulta todas esas fuentes **desde tu infraestructura** ([[02 - Infraestructura de investigación - VM, red y navegador|OPSEC]]).

# Interpretar la salida

<mark style="background: #8000E1A6;">La salida es materia prima, no inteligencia</mark>. SpiderFoot genera mucho, incluido ruido y falsos positivos. El flujo: filtra por tipo de dato, usa las **correlaciones** que marca (son lo que vale la pena mirar primero), y **verifica** a mano lo accionable antes de reportarlo. Exporta a CSV/JSON/GEXF para cruzar con otras herramientas (p. ej. visualizar en [[02 - recon-ng y Maltego|Maltego]]).

# Estado: Intel 471 (2022)

> [!info]+ SpiderFoot hoy
> **Intel 471 adquirió SpiderFoot en noviembre de 2022** (su fundador, Steve Micallef, pasó a Intel 471). El **proyecto open source sigue en GitHub** (`smicallef/spiderfoot`, licencia MIT) y es el que usas gratis; el producto comercial **SpiderFoot HX** se integró en la plataforma **TITAN** de Intel 471. Posible efecto: las contribuciones comunitarias al OSS pueden ralentizarse al desplazarse el foco del mantenedor. Comprueba la actividad del repo si te importa la frescura de un módulo.

# Cuándo usarlo

Para **footprint amplio** de una organización y para **monitorización** periódica (reejecutas el escaneo y ves qué cambió). Para el pivote fino sobre una persona, las herramientas dedicadas de [[OSINT de personas.base|personas]] dan más control. SpiderFoot es la apisonadora de amplitud; no sustituye el trabajo manual de profundidad.

> [!info]+ Fuentes
> [SpiderFoot (OSS)](https://github.com/smicallef/spiderfoot) (MIT). Adquisición por Intel 471 (nov-2022): [comunicado oficial](https://www.thomabravo.com/press-releases/intel-471-acquires-spiderfoot); integración en TITAN.
