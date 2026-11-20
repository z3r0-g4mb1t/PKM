---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
Descripción: "Qué se puede sacar de AD sin credenciales: el bind anónimo (raro), el rootDSE legible (universal) y el LDAP ping para validar usuarios"
Fecha de actualización: 2026-11-20
Nota previa: "[[13 - Enumeración con herramientas integradas]]"
Nota siguiente: "[[15 - Enumeración LDAP con credenciales]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

Antes de tener una sola credencial, LDAP ya puede dar información. Cuánta depende de la configuración, y aquí el material clásico de HTB enseña una premisa que en 2026 es falsa. Conviene separar tres vías: el bind anónimo (raro), el `rootDSE` legible (universal) y el *LDAP ping* (para validar nombres de usuario).

# El bind anónimo: posible, pero NO común

<mark style="background: #ADCCFFA6;">Un bind anónimo permite consultar el directorio sin autenticarse</mark>: listado de usuarios, grupos, equipos, atributos y la política de contraseñas, todo material para después montar un *password spraying* o un AS-REP roasting.

> [!danger]+ Errata de HTB: «el bind anónimo es común en AD»
> El módulo presenta el bind anónimo como algo que te encuentras con frecuencia. <mark style="background: #FF5582A6;">Es falso desde Windows Server 2003</mark>: por defecto, las operaciones LDAP anónimas —más allá de las búsquedas y binds sobre el `rootDSE`— **no están permitidas**. Que exista requiere que alguien haya puesto deliberadamente el **séptimo carácter** de `dsHeuristics` a `2` (lo que las herramientas de auditoría como las STIG de DISA marcan como *finding*), o que el bosque arrastre un grupo «Pre-Windows 2000 Compatible Access» relajado. Se comprueba siempre, pero esperarlo como norma es un error. Fuente: [KB 326690](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/anonymous-ldap-operations-active-directory-disabled).

Donde sí es más frecuente es en **OpenLDAP** y *appliances* Linux (vCenter, por ejemplo), que a menudo permiten binds anónimos de serie.

# Lo que SÍ es universal: el rootDSE

La reformulación correcta: aunque el bind anónimo completo sea raro, <mark style="background: #8000E1A6;">el `rootDSE` es legible sin autenticar en prácticamente cualquier AD</mark>. Y de él sale un mapa útil sin una sola credencial (ver el detalle en [[10 - LDAP en Active Directory]]): `defaultNamingContext`, `supportedSASLMechanisms`, los niveles funcionales de dominio y bosque, el `dnsHostName` del DC y el `currentTime` (el *skew* de reloj, crítico antes de cualquier ataque Kerberos).

Confirmar un bind anónimo con `ldapsearch` y, si existe, volcar objetos:

```shell-session
$ ldapsearch -H ldap://10.129.1.207 -x -b "DC=inlanefreight,DC=local"
```

El `-x` fuerza autenticación simple (aquí, anónima). Si devuelve objetos del dominio, el bind anónimo está abierto. Si solo deja leer el `rootDSE`, es el comportamiento por defecto.

# El LDAP ping: validar usuarios sin credenciales

Hay una segunda vía anónima que **no es un bind**: el **LDAP ping** o **cLDAP** (LDAP sin conexión, sobre UDP/389), atendido por el servicio **Netlogon**, no por el motor de búsqueda del directorio. Es *anonymous by design* y permite algo muy concreto: <mark style="background: #FFB86CA6;">preguntar «¿existe este usuario?» y distinguir entre habilitado, deshabilitado e inexistente</mark>, sin autenticarse. Es la base del *username enumeration* previo a un *password spraying*.

La herramienta de referencia es [`ldapnomnom`](https://github.com/lkarlslund/ldapnomnom), que fuerza bruta de nombres de usuario vía cLDAP a decenas de miles por segundo. Microsoft lo nombra explícitamente en la alerta de MDI *Account Enumeration reconnaissance (LDAP)* (external ID 2437).

> [!warning]+ El cLDAP sobre UDP es el único punto ciego real que queda
> El LDAP ping sobre **UDP** lo atiende Netlogon, así que el evento 1644 (búsquedas LDAP caras) es **estructuralmente ciego** a él — 40 nombres consultados dejan 0 eventos 1644. Solo lo ve `netlogon.log` con el nivel de depuración subido, que registra cada usuario pero **no la IP de origen**. Matiz importante que corrige a la propia documentación de la herramienta: `ldapnomnom` en la práctica habla **TCP**, no UDP, así que el evento 5156 (Windows Filtering Platform) **sí** captura su IP real. El hueco de telemetría verdadero es el cLDAP sobre UDP puro. Detalle en [[16 - Detección y evasión de la enumeración LDAP]].

# Herramientas de enumeración anónima

Además de `ldapsearch`, históricamente se usaba `windapsearch` con usuario vacío (`-u ""`). <mark style="background: #FF5582A6;">`windapsearch` está abandonado desde 2020</mark> (ver [[17 - Arsenal de enumeración LDAP]]); hoy, para una enumeración anónima o con credenciales desde Linux, las referencias son `ldeep`, `NetExec` (módulos `ldap`) y, para el *username enum* por cLDAP, `ldapnomnom`.

Lo que se obtenga sin credenciales —la política de contraseñas, una lista de usuarios, un `password` olvidado en un campo `description`— alimenta el siguiente paso: la enumeración **con** credenciales, que abre el directorio entero.

> [!info]+ Fuentes
> Bind anónimo restringido por defecto: [KB 326690](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/anonymous-ldap-operations-active-directory-disabled) y la posición de `dsHeuristics` en [MS-ADTS §6.1.1.2.4.1.2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/e5899be4-862e-496f-9a38-33950617d2c5). `ldapnomnom` y cLDAP: [repo de lkarlslund](https://github.com/lkarlslund/ldapnomnom) y la alerta MDI 2437. Módulo *Active Directory LDAP* de HTB Academy, con la premisa del bind anónimo corregida.
