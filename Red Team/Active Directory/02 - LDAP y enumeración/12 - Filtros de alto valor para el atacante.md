---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
Descripción: "El catálogo de filtros LDAP que dan botín: flags de userAccountControl, cuentas Kerberoasteables, delegación, RBCD, shadow credentials, LAPS y gMSA"
Fecha de actualización: 2026-11-20
Nota previa: "[[11 - Construir filtros de búsqueda LDAP]]"
Nota siguiente: "[[13 - Enumeración con herramientas integradas]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

Esta es la nota de consulta durante un `engagement`: los filtros LDAP que, con una sola cuenta de dominio, localizan los caminos de ataque. Todos los nombres de atributo están verificados contra el esquema de AD — un atributo mal escrito devuelve cero y hace creer que no hay nada.

# Los flags de `userAccountControl`

`userAccountControl` es una máscara de bits; se consulta con la matching rule `BIT_AND` (`:1.2.840.113556.1.4.803:=<decimal>`). Los que importan ofensivamente:

| Flag | Decimal | Relevancia |
| --- | --- | --- |
| `ACCOUNTDISABLE` | 2 | Filtrar cuentas muertas para no hacer ruido |
| `PASSWD_NOTREQD` | 32 | Posible contraseña vacía → bind directo |
| `DONT_EXPIRE_PASSWORD` | 65536 | Contraseña antigua → crackeable |
| `TRUSTED_FOR_DELEGATION` | 524288 | **Delegación no restringida** |
| `NOT_DELEGATED` | 1048576 | Cuenta «sensible»: el defensor marca lo que no se puede robar |
| `DONT_REQ_PREAUTH` | 4194304 | **AS-REP roasting** |
| `TRUSTED_TO_AUTH_FOR_DELEGATION` | 16777216 | **Delegación con transición de protocolo (S4U2Self)** |
| `SERVER_TRUST_ACCOUNT` | 8192 | Es un **DC** — sirve para excluir DCs de la caza |

> [!warning]+ Dos trampas de `userAccountControl` que HTB no avisa
> **(1)** Los DCs llevan `TRUSTED_FOR_DELEGATION` **por diseño** (su UAC por defecto es `0x82000` = `SERVER_TRUST_ACCOUNT | TRUSTED_FOR_DELEGATION`). <mark style="background: #FF5582A6;">Cualquier caza de delegación no restringida que no excluya `SERVER_TRUST_ACCOUNT` (8192) devuelve **todos los DCs** como falso hallazgo</mark>. **(2)** Las cuentas de trust llevan `PASSWD_NOTREQD` por defecto: salen siempre en esa búsqueda y no son un hallazgo. Y ojo: los bits `LOCKOUT` y `PASSWORD_EXPIRED` están **sustituidos** desde Windows Server 2003 por el atributo construido `msDS-User-Account-Control-Computed`; leerlos de `userAccountControl` da basura.

# El catálogo de caza

Filtros verificados, listos para `-LDAPFilter`, `ldapsearch` o `ldeep`:

**Cuentas Kerberoasteables** (tienen SPN, no son `krbtgt`, no están deshabilitadas):
```text
(&(objectCategory=person)(objectClass=user)(servicePrincipalName=*)(!(sAMAccountName=krbtgt))(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
```
`objectCategory=person` excluye las cuentas de máquina, que tienen SPN siempre.

**AS-REP roasteables** (no requieren pre-autenticación):
```text
(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
```

**Delegación no restringida** (excluyendo DCs con `!8192`):
```text
(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288)(!(userAccountControl:1.2.840.113556.1.4.803:=8192)))
```

**Delegación restringida (KCD)** — el atributo `msDS-AllowedToDelegateTo` lista a qué servicios puede delegar:
```text
(msDS-AllowedToDelegateTo=*)
```

**RBCD (delegación restringida basada en recursos)** — el atributo clave:
```text
(msDS-AllowedToActOnBehalfOfOtherIdentity=*)
```

**Shadow credentials** (`msDS-KeyCredentialLink`, el vector de PKINIT):
```text
(msDS-KeyCredentialLink=*)
```
<mark style="background: #FFB8EBA6;">Caveat: Windows Hello for Business rellena este atributo legítimamente</mark> — no todo hallazgo es un ataque; hay que distinguir.

**gMSA** — descubrirlas por su clase, y pedir la contraseña por su atributo (no filtrar por él):
```text
(objectClass=msDS-GroupManagedServiceAccount)
```
El secreto vive en `msDS-ManagedPassword`, un atributo **construido** que hay que pedir explícitamente como atributo de salida; `(msDS-ManagedPassword=*)` como filtro **no funciona**, y el GC nunca lo devuelve.

**LAPS** — distinguir el legado del nuevo:
```text
(ms-Mcs-AdmPwd=*)                                    # LAPS legado
(|(msLAPS-Password=*)(msLAPS-EncryptedPassword=*))   # Windows LAPS (nuevo)
```

**Contraseñas en campos de texto libre** (hallazgo clásico y frecuente):
```text
(&(objectCategory=person)(objectClass=user)(|(description=*pass*)(description=*pwd*)(info=*pass*)(userPassword=*)))
```

**Cuentas administrativas protegidas** (`adminCount=1`):
```text
(adminCount=1)
```

**`MachineAccountQuota`** — cuántas máquinas puede unir un usuario (clave para RBCD):
```text
# base = defaultNamingContext, scope base, atributo: ms-DS-MachineAccountQuota
```

> [!danger]+ El nombre del atributo MachineAccountQuota es una trampa
> El `ldapDisplayName` es **`ms-DS-MachineAccountQuota`** — **con guiones en el prefijo**, no `msDS-MachineAccountQuota` como los demás atributos `msDS-*`. Escribirlo al estilo de los otros devuelve cero. Microsoft **no documenta su valor por defecto** (el conocido `10` se lee del dominio, no de la referencia del atributo).

# Dos matices que cambian la interpretación

**`adminCount=1` es un marcador histórico, no estado actual.** <mark style="background: #8000E1A6;">El sistema lo pone al añadir una cuenta a un grupo administrativo, pero **no lo limpia** al sacarla</mark>. Encontrar `adminCount=1` en una cuenta que ya no es admin señala una cuenta que *lo fue*: suele conservar la ACL endurecida, la herencia deshabilitada y, a menudo, una contraseña vieja y privilegios residuales. HTB lo trata como simple sinónimo de «es admin»; es más que eso — es un rastro de privilegio pasado. El mecanismo (`AdminSDHolder` + SDProp cada 60 min en el PDC Emulator) está en [[35 - Hardening de Active Directory]].

**El Global Catalog no sirve aquí.** Casi todos los atributos de esta nota —`msDS-AllowedToDelegateTo`, RBCD, `msDS-ManagedPassword`, `adminCount`, `ms-DS-MachineAccountQuota`— están marcados `In Global Catalog: False`. Consultar al GC (3268) los omite. El GC es para el mapa del bosque; la caza se hace contra el DC del dominio (389). Los atributos de LAPS, además, son **confidenciales** (`SearchFlags` con `fCONFIDENTIAL`): un usuario de dominio cualquiera no los lee aunque el atributo exista — hace falta `CONTROL_ACCESS` concedido.

Estos filtros alimentan directamente a las herramientas de las notas siguientes ([[13 - Enumeración con herramientas integradas]], [[15 - Enumeración LDAP con credenciales]]) y a los ataques desarrollados en [[06 - Enumeración inicial del dominio|enumeración]], Kerberoasting ([[23 - Kerberoasting]]) y abuso de ACLs ([[24 - Primer de abuso de ACLs]]).

> [!info]+ Fuentes
> Flags: [UserAccountControl property flags (KB 305144)](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties). Atributos verificados contra el [AD Schema](https://learn.microsoft.com/en-us/windows/win32/adschema/) y [MS-ADA2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-ada2/). Windows LAPS: [referencia técnica](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-technical-reference). Filtros ampliados y corregidos respecto al módulo *Active Directory LDAP* de HTB Academy con los vectores de 2026 (RBCD, shadow credentials, Windows LAPS, gMSA).
