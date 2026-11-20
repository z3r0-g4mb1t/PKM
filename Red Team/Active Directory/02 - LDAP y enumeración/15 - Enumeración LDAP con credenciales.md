---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
Descripción: Qué se extrae de AD con una cuenta de dominio válida vía LDAP y cómo los límites del servidor (paginación, MaxPageSize) condicionan el barrido
Fecha de actualización: 2026-11-20
Nota previa: "[[14 - Bind anónimo y enumeración sin credenciales]]"
Nota siguiente: "[[16 - Detección y evasión de la enumeración LDAP]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

Con una sola cuenta de dominio —por insignificante que sea— LDAP abre casi todo el directorio. <mark style="background: #ADCCFFA6;">Un usuario básico sin privilegios puede enumerar la mayoría de los objetos de AD</mark>: usuarios, grupos y su pertenencia, equipos, trusts, GPOs, la política de contraseñas y las ACLs. Esa es la premisa que hace de AD una base de datos de lectura casi universal, y el motivo por el que conseguir *cualquier* credencial cambia el juego.

# Qué se extrae

Con credenciales, las mismas herramientas de Linux que se usan anónimamente rinden mucho más. El objetivo es construir el inventario que informará los ataques:

- **Política de contraseñas** → define la ventana de un *password spraying* (umbral de bloqueo, longitud mínima, complejidad).
- **Inventario de usuarios** → la *wordlist* de objetivos para spraying/roasting.
- **Cuentas Kerberoasteables y AS-REP roasteables** → con los filtros de [[12 - Filtros de alto valor para el atacante]].
- **Delegación (no restringida, KCD, RBCD)** y **shadow credentials** → caminos de escalada.
- **Pertenencia anidada de grupos** → con la matching rule `IN_CHAIN` ([[11 - Construir filtros de búsqueda LDAP]]).
- **Contraseñas en `description`/`info`** → el hallazgo barato que aparece una y otra vez.

Ejemplo con una herramienta de enumeración orientada a AD, que trae consultas predefinidas:

```shell-session
$ ldeep ldap -u james.cross -p Summer2020 -d inlanefreight.local -s ldap://10.129.1.207 pso
$ ldeep ldap -u james.cross -p Summer2020 -d inlanefreight.local -s ldap://10.129.1.207 users -k
```

O con NetExec, que agrupa decenas de módulos LDAP (`laps`, `daclread`, `shadow-creds`, `maq`, `get-desc-users`, `pre2k`, `obsolete`…):

```shell-session
$ nxc ldap 10.129.1.207 -u james.cross -p Summer2020 --kerberoasting kerb.txt
$ nxc ldap 10.129.1.207 -u james.cross -p Summer2020 -M get-desc-users
```

<mark style="background: #FFB86CA6;">La combinación «cuenta de dominio + NetExec/ldeep» es el estándar de 2026 para la enumeración LDAP desde Linux</mark>, no `windapsearch` ni `ldapsearch-ad` (ver [[17 - Arsenal de enumeración LDAP]]).

# Los límites del servidor condicionan el barrido

AD no entrega resultados ilimitados. El servidor LDAP impone cuotas que un barrido ingenuo choca de frente, con consecuencias que se confunden con «no hay datos»:

| Límite | Por defecto | Qué pasa al superarlo |
| --- | --- | --- |
| `MaxPageSize` | **1000 objetos** | Sin paginación, truncamiento **silencioso** |
| `MaxQueryDuration` | 120 s | El DC devuelve `timeLimitExceeded` |
| `MaxValRange` | **1500 valores** | Atributos multivaluados grandes (`member` de Domain Users) se cortan |
| `MaxConnIdleTime` | 900 s | Conexión inactiva → desconexión (mata el *low-and-slow* ingenuo) |

> [!warning]+ La paginación no es opcional
> <mark style="background: #FF5582A6;">Sin el control de resultados paginados (`1.2.840.113556.1.4.319`), cualquier consulta que devuelva más de 1000 objetos se **trunca sin avisar**</mark>: la herramienta parece funcionar y se pierde el resto. Las herramientas modernas (ldeep, NetExec, SharpHound) paginan solas; al escribir filtros a mano con `ldapsearch`, usar `-E pr=1000/noprompt`. Para atributos multivaluados enormes (`member`), el *value range retrieval* (`member;range=0-1499`). Y un dato que HTB no da: `MaxValRange` es **1500**, no 1000 (el 1000 era el valor fijo de Windows 2000).

Estos límites se leen sin acceso privilegiado desde el atributo `LDAPPoliciesEffective` del `rootDSE` ([[10 - LDAP en Active Directory]]) — `ntdsutil` solo muestra la política por defecto y «miente» si el entorno tiene una política propia, que es lo normal en una empresa grande.

# El destino natural: BloodHound

La enumeración manual con filtros es imprescindible para entender qué se busca y para *vivir off the land*, pero el volumen de relaciones de un dominio real pide un grafo. **SharpHound** (desde Windows) y la ingestión en **BloodHound CE** recogen por LDAP todo lo anterior y lo presentan como caminos de ataque navegables. La enumeración LDAP de este sub-tema es la base conceptual sobre la que BloodHound opera — y sus firmas de recolección son lo que los defensores cazan, en [[16 - Detección y evasión de la enumeración LDAP]].

> [!info]+ Fuentes
> Lo que un usuario de dominio puede enumerar: módulo *Active Directory LDAP* de HTB Academy. Límites del servidor: [View and set LDAP policy with Ntdsutil (KB 315071)](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/view-set-ldap-policy-using-ntdsutil). Control de paginación: [MS-ADTS](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/). Herramientas modernas verificadas contra la API de GitHub (2026-11-20).
