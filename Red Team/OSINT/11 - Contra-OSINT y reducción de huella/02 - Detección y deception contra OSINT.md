---
tags:
  - OSINT/Contra-OSINT
  - Tipo/Deteccion
Descripción: "Las pocas señales que deja el OSINT y las trampas proactivas: monitorización de CT logs, canary tokens, honeypots y detección de typosquatting"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Reducción de huella personal y corporativa]]"
Nota siguiente:
Area: "[[Contra-OSINT.base|Contra-OSINT]]"
---
---

Si el OSINT pasivo es invisible, la defensa proactiva consiste en **vigilar los pocos momentos en que el atacante sí deja rastro** y en **plantar cebos** que lo delaten cuando pasa de mirar a actuar. Es la parte más ingeniosa del contra-OSINT.

# Monitorizar tus Certificate Transparency logs

<mark style="background: #ADCCFFA6;">Cada certificado TLS emitido es público ([[07 - Certificate Transparency logs|CT logs]]), y eso te da una ventana al atacante</mark>: cuando registra un dominio de phishing (`login-acme-sso.com`) y le saca un certificado, **esa emisión es visible para ti antes de que lance el ataque**.

- Monitoriza los CT logs buscando tu marca y variantes: **crt.sh**, **Cert Spotter** (SSLMate), o un *stream* en vivo con **CertStream**.
- Una alerta por un certificado `acme-support.net` que tú no emitiste = alguien preparando una suplantación. Lo detectas en la fase de **montaje**, no cuando ya está robando credenciales.

<mark style="background: #8000E1A6;">Es de las pocas señales tempranas que da un atacante</mark> — y nace de su propia infraestructura, no de tocarte a ti.

# Canary tokens: trampas que avisan

<mark style="background: #FFB86CA6;">Un *canary token* es un cebo que, al ser **usado**, te avisa</mark> — convierte la recolección invisible en una alerta. **canarytokens.org** (Thinkst) genera gratis tokens de muchos tipos:

- Una **URL/web bug** que alerta al cargarse.
- Un **registro DNS** que alerta al resolverse.
- Una **clave AWS** falsa que alerta al usarse (API call).
- Un **documento Word/PDF** o una **carpeta** que alertan al abrirse.

La táctica contra-OSINT: <mark style="background: #FF5582A6;">plantar los tokens donde un atacante que hace OSINT los **encontrará y usará**</mark>. Un `config.php` con una "clave AWS" canary en un repo, un documento "confidencial" cebo indexable, un subdominio trampa. Cuando el atacante —que te dorkeó y encontró el cebo— lo abre o lo usa, te delata su interés y, a menudo, su IP y su herramienta.

> [!example]+ El cebo en acción
> Siembras `backup_credentials.docx` con una clave AWS canary en un recurso indexable. Un atacante lo encuentra dorkeando, abre el documento y prueba la clave. Recibes una alerta con la hora, la IP y el `User-Agent` → sabes que te están investigando, desde dónde y con qué. La recolección "invisible" acaba de hacerse visible.

# Honeypots y cuentas señuelo

- **Cuentas de empleado falsas**: usuarios señuelo que nadie usa; un intento de login contra ellos (tras un [[01 - LinkedIn - la mina del red teamer|OSINT de personal]] + [[03 - Credential reuse, combolists y password spraying|spraying]]) delata el ataque.
- **Servicios expuestos trampa**: un "RDP" o un panel que parece jugoso en Shodan pero es un honeypot — la interacción revela al atacante.

# Detectar suplantación: typosquatting

<mark style="background: #ADCCFFA6;">Vigila los dominios parecidos al tuyo</mark>: **dnstwist** genera las variantes *typosquat* de tu dominio y comprueba cuáles están **registradas** y activas — las que un atacante usaría para phishing o para suplantarte. Monitorizarlo detecta la infraestructura de ataque antes de que se use.

# El cierre: la asimetría, de los dos lados

<mark style="background: #8000E1A6;">El OSINT pasivo es invisible; la deception es cómo cazas la **transición a la acción**</mark>. Para el defensor, es la respuesta a "no puedo detectar que me investigan": no detectes la mirada, detecta la mano cuando toca el cebo. Para el red teamer, esta nota es el mapa de lo que **te delata** — canary tokens, certificados que emites, logins que pruebas, cebos demasiado fáciles. Tenerlo presente es lo que separa una operación limpia de una quemada ([[05 - Verificación, fiabilidad de fuentes y sesgos#Datos envenenados: cuando la fuente te tiende una trampa|datos envenenados]], [[03 - Cómo no quemar la operación - fugas y errores comunes|fugas]]).

> [!info]+ Fuentes
> Monitorización de CT logs: [crt.sh](https://crt.sh), [Cert Spotter (SSLMate)](https://sslmate.com/certspotter/), CertStream. Canary tokens: [canarytokens.org](https://canarytokens.org) (Thinkst). Detección de typosquatting: [dnstwist](https://github.com/elceef/dnstwist).
