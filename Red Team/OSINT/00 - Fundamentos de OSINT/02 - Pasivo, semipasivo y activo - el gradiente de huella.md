---
tags:
  - OSINT/Fundamentos
  - Pentesting/Enumeracion
Descripción: "El gradiente pasivo-semipasivo-activo, y por qué 'pasivo respecto al objetivo' no significa invisible ni sin rastro"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - El ciclo de inteligencia aplicado al OSINT]]"
Nota siguiente: "[[03 - Taxonomía de fuentes y disciplinas OSINT]]"
Area: "[[Fundamentos OSINT.base|Fundamentos OSINT]]"
---
---

La distinción pasivo/activo es la que gobierna **cuánto rastro dejas** y **cuándo necesitas autorización**. El recon web ya trae [[00 - Reconocimiento web#Reconocimiento activo vs pasivo|la tabla clásica activo vs pasivo]]; esta nota no la repite, la afina con dos matices que el OSINT real obliga a entender: el escalón **semipasivo** y el hecho de que <mark style="background: #FF5582A6;">"pasivo respecto al objetivo" no es lo mismo que "invisible" ni "sin rastro"</mark>.

# El gradiente, no la pared

En la práctica no hay dos categorías sino un gradiente de tres escalones según **quién ve tu actividad**:

| Escalón | Quién te ve | Ejemplos |
| - | - | - |
| **Pasivo** | Solo terceros (nunca el objetivo) | `WHOIS`, `crt.sh`, Shodan/Censys, buscadores, LinkedIn, bases de brechas, Wayback |
| **Semipasivo** | Infraestructura pública compartida; tráfico indistinguible del normal | Resolver DNS de subdominios en un resolver público, visitar la web como un usuario cualquiera, cargar la home vía CDN |
| **Activo** | El objetivo, en sus logs | `port scan`, fuerza bruta de subdominios/directorios, `nuclei`, login en la app, *probing* de endpoints |

<mark style="background: #ADCCFFA6;">**Pasivo**: obtienes la información de un tercero que ya la tiene; el objetivo no recibe ni un paquete tuyo</mark>. Preguntar a `crt.sh` qué certificados ha emitido una CA para `*.acme.com` te da subdominios sin que ACME se entere.

<mark style="background: #ADCCFFA6;">**Semipasivo**: tocas infraestructura pública o al propio objetivo de forma indistinguible del tráfico legítimo</mark>. Resolver `vpn.acme.com` contra `8.8.8.8` consulta al DNS, no a ACME (salvo que ACME aloje su propio autoritativo, y aun así es una query DNS normal). Cargar `https://www.acme.com` una vez, con un navegador real, es una visita más entre miles. No es "sin tocar", pero es ruido de fondo.

<mark style="background: #ADCCFFA6;">**Activo**: generas tráfico anómalo o dirigido que el objetivo puede registrar y atribuir</mark>. Un `nmap -sV` contra sus IPs, un `ffuf` descubriendo directorios o un intento de login quedan en sus logs y pueden disparar un `IDS`/`WAF`.

> [!example]+ El mismo subdominio, tres niveles de huella
> 1. **Pasivo** — `crt.sh` y `Shodan` te revelan que existe `jenkins.acme.com`. ACME no ve nada.
> 2. **Semipasivo** — resuelves `jenkins.acme.com` → `10.0.0.5` y cargas la home una vez con el navegador. Una petición HTTP normal.
> 3. **Activo** — lanzas `nuclei` con las plantillas de Jenkins y pruebas credenciales por defecto. Ahora estás atacando, y ACME lo ve.
>
> El flujo real **encadena los tres de abajo arriba**: construyes el inventario completo en pasivo, confirmas lo vivo en semipasivo, y solo escalas a activo sobre lo que ya sabes que merece la pena —y, en un engagement, sobre lo que está **dentro de alcance**.

# "Pasivo" no es "invisible"

El matiz que más se malinterpreta: <mark style="background: #8000E1A6;">pasivo describe la relación con el **objetivo**, no con el resto del mundo</mark>. Cada consulta pasiva deja rastro **en el tercero** que consultas, y a veces ese rastro llega al objetivo:

- **El tercero te logea.** Shodan y Censys registran tus consultas de API (con tu clave). El proveedor de *breach data* sabe qué dominios buscas. LinkedIn registra que tu perfil visitó 300 empleados de ACME —y, si tienes la visita de perfil activada, **se lo notifica a cada uno**—. De ahí el [[OPSEC OSINT.base|OPSEC del investigador]]: cuentas desechables, no las tuyas.
- **El objetivo puede vigilar al tercero.** Muchas empresas monitorizan los **`Certificate Transparency` logs** de sus dominios ([[07 - Certificate Transparency logs|CT logs]]) y reciben una alerta cuando se emite un certificado para un subdominio nuevo. Si levantas infraestructura de phishing con un certificado `login-acme-sso.com`, esa emisión es pública y el *blue team* de ACME puede verla **antes de que lances el phishing**.
- **Los *canary tokens* convierten lo pasivo en delator.** Un documento cebo con un *token* incrustado, o una entrada DNS trampa, avisan al defensor en cuanto alguien los abre o resuelve. Un bucket "olvidado" puede ser un honeypot.

> [!warning]+ Autorización: el eje no es pasivo/activo, es dentro/fuera de alcance
> La fase pasiva suele poder ejecutarse incluso sobre infraestructura de terceros sin tocar al cliente. Pero **lanzar técnicas activas** (escaneo, fuerza bruta, login) contra un host **fuera de alcance** puede violar el contrato o la política del programa de *bug bounty*, e incluso la ley. Antes de pasar a activo, confirma que el activo está en *scope*. El detalle legal, en [[04 - Marco legal y ético del OSINT]].

La conclusión operativa: trata cada acción por su huella real, no por la etiqueta. Un dork en Google es pasivo e invisible; una visita logueada a LinkedIn es pasiva pero rastreable; monitorizar los CT logs del objetivo es pasivo pero puede alertarle. Planifica en consecuencia.
