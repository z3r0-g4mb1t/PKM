---
tags:
  - OSINT/Corporativo
  - Pentesting/Enumeracion
Descripción: "Identificar la tecnología del objetivo sin tocarlo: BuiltWith/Wappalyzer, el proveedor de correo por MX y la spoofability por SPF/DMARC"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Registros mercantiles y financieros]]"
Nota siguiente: "[[03 - Cadena de suministro, filiales y M&A]]"
Area: "[[OSINT corporativo.base|OSINT corporativo]]"
---
---

Saber **qué tecnología** usa el objetivo define qué CVEs probar, qué portal atacar y cómo construir un phishing creíble — todo antes de tocar un solo sistema. Tres fuentes pasivas dan casi todo: el perfilado web, los registros de correo y la política antispoofing.

# Perfilado web: BuiltWith y Wappalyzer

<mark style="background: #ADCCFFA6;">`BuiltWith` y `Wappalyzer` identifican la pila tecnológica de un sitio</mark> — CMS, frameworks, analítica, servidor, CDN, pasarelas de pago, herramientas de marketing — a partir de las huellas en el HTML, las cabeceras y los recursos que carga:

- **Wappalyzer**: extensión de navegador (perfila al vuelo la web que visitas) o su CLI/API. Gratuito para uso básico.
- **BuiltWith**: web + API; además da histórico (qué tecnología usaban antes) y *relationships* (otros dominios con el mismo ID de Analytics o de anuncios → activos del mismo dueño).

<mark style="background: #FFB86CA6;">Lo que detectan es superficie de ataque y munición de phishing a la vez</mark>: si usan WordPress 6.3 con cierto plugin, tienes CVEs que probar; si usan Salesforce, Workday o Concur, tienes marcas creíbles para suplantar en un correo.

> [!warning]+ Wappalyzer toca el objetivo (levemente)
> Visitar la web para perfilarla es [[02 - Pasivo, semipasivo y activo - el gradiente de huella|semipasivo]] (una petición HTTP normal). BuiltWith, consultado desde su web, es pasivo (preguntas a BuiltWith, no al objetivo). Para recon totalmente pasivo, usa BuiltWith; para la foto más fresca, Wappalyzer sobre la web.

# El proveedor de correo: registros MX

<mark style="background: #ADCCFFA6;">Los registros `MX` del dominio revelan quién gestiona su correo</mark>, y eso te dice qué portal de login atacar y cómo pretextar:

```shell-session
$ dig +short MX acme.com
# 0 acme-com.mail.protection.outlook.com   → Microsoft 365
# (o aspmx.l.google.com                      → Google Workspace)
```

<mark style="background: #8000E1A6;">`*.protection.outlook.com` = Microsoft 365</mark>: el objetivo tiene OWA/Azure AD, susceptible de *password spraying* contra el portal, de *device code phishing* y de enumeración de usuarios vía O365. `*.google.com` = Workspace: otro conjunto de técnicas. Es un dato que orienta toda la fase de acceso.

# La joya: spoofability por SPF y DMARC

Si vas a hacer phishing, lo primero es saber **si puedes suplantar el dominio del objetivo**. Lo dicen sus registros `TXT`:

```shell-session
# Política SPF (qué servidores pueden enviar en su nombre)
$ dig +short TXT acme.com | grep spf
# v=spf1 include:spf.protection.outlook.com -all

# Política DMARC (qué hacer con el correo que falla la verificación)
$ dig +short TXT _dmarc.acme.com
# v=DMARC1; p=none; rua=mailto:dmarc@acme.com
```

Interpretación crítica para el red teamer:

- <mark style="background: #FF5582A6;">**`p=none`** (o DMARC ausente): el dominio es **spoofable**</mark> — puedes enviar correo "desde" `@acme.com` y lo más probable es que entregue. Es el hallazgo de phishing de más valor.
- **`p=quarantine`**: el correo falso va a spam (sorteable, pero más difícil).
- **`p=reject`**: no puedes suplantar el dominio directamente; tocará *lookalike* (`acme-corp.com`) o suplantar a un tercero.
- **SPF `-all`** (hard fail) vs **`~all`** (soft fail): el `~all` es más laxo y ayuda al atacante.

Un `p=none` en el DMARC de un dominio corporativo es, además, un hallazgo de informe por derecho propio.

# De la tecnología a los activos

El perfilado tecnológico se solapa con el [[OSINT de infraestructura.base|OSINT de infraestructura]]: `SecurityTrails` da el histórico DNS y los subdominios; los certificados revelan hostnames; el ASN agrupa los rangos IP del objetivo. El **tech stack** responde al "qué software"; la **infraestructura**, al "qué hosts e IPs". Se desarrollan juntos en la sub-área de infraestructura. Las **ofertas de empleo** ([[03 - Enumeración de personal#Las ofertas de empleo son un regalo|job postings]]) confirman y amplían todo esto, firmado por la propia empresa.

> [!info]+ Fuentes
> [BuiltWith](https://builtwith.com), [Wappalyzer](https://www.wappalyzer.com). SPF/DMARC: `RFC 7208` (SPF) y `RFC 7489` (DMARC); interpretación ofensiva de `p=none` como spoofable es práctica estándar de phishing. Histórico DNS: [SecurityTrails](https://securitytrails.com).
