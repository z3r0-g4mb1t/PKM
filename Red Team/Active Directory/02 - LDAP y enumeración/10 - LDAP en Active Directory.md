---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: Qué es LDAP y su relación con AD, los tipos de bind, puertos y cifrado, y por qué el rootDSE da un mapa del dominio antes de autenticarse
Fecha de actualización: 2026-11-20
Nota previa: "[[09 - Living Off the Land]]"
Nota siguiente: "[[11 - Construir filtros de búsqueda LDAP]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

<mark style="background: #ADCCFFA6;">LDAP (Lightweight Directory Access Protocol) es el idioma con el que se consulta un servicio de directorio</mark>. La analogía que lo fija: Active Directory es a LDAP lo que Apache a HTTP. AD es el servidor de directorio; LDAP es el protocolo con el que se le habla. La especificación vigente es **LDAPv3**, publicada en [RFC 4511](https://www.rfc-editor.org/info/rfc4511) (junio 2006) — lleva 20 años en estado *Proposed Standard* y nada la ha sustituido.

Dominar LDAP importa a ambos lados: el atacante lo usa para inventariar el dominio entero desde una sola cuenta de usuario, y el defensor para entender qué consultas delatan ese inventario. No todo entorno con LDAP es AD —existe [OpenLDAP](https://www.openldap.org/) y los *appliances* Linux lo usan—, pero en una red corporativa Windows, LDAP **es** la puerta a AD.

# El inicio de sesión: la operación BIND

Una sesión LDAP empieza conectando al servidor —el `Directory System Agent`, que en AD es el Domain Controller— y fijando el estado de autenticación con una operación **BIND**. Hay dos familias, y la distinción tiene peso en la detección:

- **Simple authentication** — usuario y contraseña directos. Incluye el bind anónimo (sin credenciales) y el no autenticado. <mark style="background: #FF5582A6;">Un simple bind sobre el puerto 389 sin TLS envía usuario y contraseña en claro</mark>: es la definición exacta del evento 2889 de los DCs.
- **SASL authentication** — delega en un mecanismo externo (Kerberos vía GSSAPI, GSS-SPNEGO, EXTERNAL). El DC anuncia los que acepta en el atributo `supportedSASLMechanisms` del rootDSE. <mark style="background: #FFB8EBA6;">Un bind SASL nunca manda la contraseña en claro</mark>; lo que puede faltarle es *integridad* (firma), que es cosa distinta.

> [!warning]+ Errata de HTB matizada: «LDAP va en claro por defecto»
> El material de HTB (2020-2021) afirma que los mensajes de autenticación LDAP viajan en claro por defecto. Es **parcialmente cierto y hoy desfasado**. Lo cierto: hasta Windows Server 2022 inclusive, el DC acepta por defecto binds **sin firmar** y simple binds en claro sobre 389. Lo desfasado: <mark style="background: #FFB86CA6;">Windows Server 2025 exige firma LDAP por defecto en despliegues nuevos</mark> (política `Domain controller: LDAP server signing requirements enforcement`), y el cliente de 2025+ cifra por defecto. Matiz clave: «en claro» (simple bind sin TLS) y «sin firmar» (SASL sin integridad) **no son lo mismo**. Fuente: [Microsoft Learn — LDAP signing](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/ldap-signing) (actualizado 2026-10-02). Un *upgrade* a 2025 **conserva** la configuración previa; solo las instalaciones nuevas heredan el default seguro.

# Puertos y cifrado

| Puerto | Servicio | Uso |
| --- | --- | --- |
| `389/TCP` y `389/UDP` | LDAP | Consulta estándar. El UDP es la vía del *LDAP ping* / cLDAP |
| `636/TCP` | LDAPS | LDAP sobre TLS desde el handshake |
| `3268/TCP` | LDAP Global Catalog | Vista parcial de **todo el bosque** en una consulta |
| `3269/TCP` | LDAP GC sobre SSL | GC cifrado |
| `9389/TCP` | ADWS | Active Directory Web Services — lo usan los cmdlets de RSAT |

**LDAPS vs StartTLS** son dos formas distintas de cifrar:

- **LDAPS** (636/3269): el TLS ocurre *antes* de cualquier operación; la conexión va cifrada toda su vida.
- **StartTLS**: se abre una sesión normal en 389 y se eleva a TLS a mitad con la operación extendida `ldap_start_tls_s` (OID `1.3.6.1.4.1.1466.20037`). Se puede incluso volver a claro. Para saber si un DC la soporta, se mira el atributo operacional `supportedExtension` del rootDSE.

> [!info]+ El Global Catalog no sirve para la caza fina
> El GC (3268) es tentador porque ve el bosque entero, pero **no devuelve los atributos marcados `In Global Catalog: False`** — y ahí cae casi todo lo jugoso: `msDS-AllowedToDelegateTo`, `msDS-AllowedToActOnBehalfOfOtherIdentity` (RBCD), `msDS-ManagedPassword`, `adminCount`, `ms-DS-MachineAccountQuota`. El GC sirve para el **mapa** del bosque, no para la caza de delegación o credenciales (ver [[12 - Filtros de alto valor para el atacante]]).

# El rootDSE: el mapa gratis

El `rootDSE` es una entrada especial en la raíz del árbol que **se lee sin autenticar**. Es el hallazgo que de verdad cambia el enunciado de HTB sobre el bind anónimo: aunque desde Windows Server 2003 las operaciones anónimas estén restringidas, <mark style="background: #8000E1A6;">el rootDSE sigue siendo legible sin una sola credencial</mark>, y de él sale un mapa del dominio:

| Atributo | Qué revela |
| --- | --- |
| `defaultNamingContext` | El DN base del dominio sin conocer el FQDN |
| `rootDomainNamingContext` | La raíz del bosque |
| `configurationNamingContext` / `schemaNamingContext` | Base para enumerar ADCS, sitios, trusts |
| `supportedSASLMechanisms` | Qué binds acepta el DC |
| `domainFunctionality` / `forestFunctionality` | DFL/FFL — fija qué ataques están disponibles |
| `dnsHostName` / `ldapServiceName` | FQDN del DC y SPN del servicio LDAP |
| `currentTime` | El reloj del DC — crítico para el *skew* antes de Kerberos |
| `LDAPPoliciesEffective` | Los límites LDAP efectivos de ese DC (operacional) |

El último es un regalo que HTB no menciona: `LDAPPoliciesEffective` devuelve los límites del servidor —`MaxPageSize`, `MaxQueryDuration`— desde el propio rootDSE, sin `ntdsutil` ni acceso privilegiado al DC. Esos límites condicionan toda la enumeración, como se ve en [[11 - Construir filtros de búsqueda LDAP]].

Comprobar el bind anónimo con `ldap3` en Python es inmediato, y la conexión al rootDSE da el `naming context` de arranque:

```python
>>> from ldap3 import Server, Connection, ALL
>>> s = Server('10.129.1.207', get_info=ALL)
>>> c = Connection(s, '', '')
>>> c.bind()
True
>>> s.info.naming_contexts
['DC=INLANEFREIGHT,DC=LOCAL', 'CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL', ...]
```

Esa es la base para todo lo demás: con el `naming context` en mano, la enumeración se construye con filtros de búsqueda, que son el tema de la nota siguiente. La relación con la vulnerabilidad de inyección sobre estos mismos filtros está en [[00 - Introducción a LDAP Injection]].

> [!info]+ Fuentes
> Protocolo: [RFC 4511](https://www.rfc-editor.org/info/rfc4511) y el roadmap [RFC 4510](https://datatracker.ietf.org/doc/rfc4510/). Puertos: [KB 179442 — puertos de AD](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/config-firewall-for-ad-domains-and-trusts). rootDSE: [MS-ADTS §3.1.1.3.2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/96f7b086-1ca3-4764-9a08-33f8f7a543db). Signing/StartTLS: [LDAP signing](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/ldap-signing). Material de partida: módulo *Active Directory LDAP* de HTB Academy, modernizado a 2026.
