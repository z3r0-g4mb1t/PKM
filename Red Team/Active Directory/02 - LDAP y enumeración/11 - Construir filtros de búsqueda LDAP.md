---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
Descripción: La sintaxis de notación polaca de los filtros LDAP, las matching rule OIDs para operar sobre bits y cadenas de pertenencia, y el ámbito de búsqueda
Fecha de actualización: 2026-11-20
Nota previa: "[[10 - LDAP en Active Directory]]"
Nota siguiente: "[[12 - Filtros de alto valor para el atacante]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

Enumerar AD con LDAP es construir filtros de búsqueda. Dominar su sintaxis separa al que depende de que sus herramientas funcionen del que, sin ninguna, *vive off the land* con los cmdlets integrados. La sintaxis está definida en [RFC 4515](https://www.rfc-editor.org/info/rfc4515), y entenderla también explica cómo funcionan por debajo PowerView, BloodHound o `ldapsearch`.

# Notación polaca

<mark style="background: #ADCCFFA6;">Los operadores lógicos van **delante** de los operandos, no en medio</mark> — es notación polaca (prefija). Cada criterio va entre paréntesis, y los grupos se anidan con más paréntesis:

| Operador | Función |
| --- | --- |
| `&` | AND |
| `\|` | OR |
| `!` | NOT |

Así, `(C1 AND C2)` se escribe `(&(C1)(C2))`, y `(C1 AND C2) OR (C3 AND C4)` se escribe `(|(&(C1)(C2))(&(C3)(C4)))`. El AND admite dos o más criterios: `(&(C1)(C2)(C3))`.

Los criterios sobre un atributo usan uno de estos tipos de coincidencia:

| Tipo | Significado | Ejemplo |
| --- | --- | --- |
| `=` | Igual a | `(sAMAccountName=jdoe)` |
| `=*` | El atributo existe (presente) | `(servicePrincipalName=*)` |
| `=valor*` | Subcadena | `(name=SQL*)` |
| `~=` | Aproximadamente igual | `(sAMAccountName~=jason)` |
| `>=` / `<=` | Mayor/menor o igual | `(maxStorage>=100000)` |

Y los caracteres que **deben escaparse** si forman parte de un valor (no de la sintaxis), con su código hex:

| Carácter | Escapado |
| --- | --- |
| `*` | `\2a` |
| `(` | `\28` |
| `)` | `\29` |
| `\` | `\5c` |
| `NUL` | `\00` |

Ese escapado es exactamente lo que falla en una [[00 - Introducción a LDAP Injection|inyección LDAP]]: cuando la aplicación mete entrada del usuario en un filtro sin escaparla, el `*` y los paréntesis cambian la lógica de la consulta.

# Matching rule OIDs: operar sobre bits y cadenas de pertenencia

Para atributos que son **máscaras de bits** (como `userAccountControl`) o para recorrer **cadenas de pertenencia anidada**, LDAP ofrece reglas de coincidencia extensibles, identificadas por OID. La sintaxis es `<atributo>:<OID>:=<valor>`, sin espacios y con el valor en **decimal**:

| OID | Nombre | Qué hace |
| --- | --- | --- |
| `1.2.840.113556.1.4.803` | `LDAP_MATCHING_RULE_BIT_AND` | Coincide si **todos** los bits del valor están en el atributo (AND de bits) |
| `1.2.840.113556.1.4.804` | `LDAP_MATCHING_RULE_BIT_OR` | Coincide si **alguno** de los bits está (OR de bits) |
| `1.2.840.113556.1.4.1941` | `LDAP_MATCHING_RULE_IN_CHAIN` | Recorre la cadena de ancestros/miembros hasta la raíz (pertenencia transitiva) |
| `1.2.840.113556.1.4.2253` | `LDAP_MATCHING_RULE_DN_WITH_DATA` | Para atributos *linked* con valor DN-String/DN-Binary |

> [!warning]+ Errata de Microsoft: hay cuatro, no tres (ni dos)
> La página *Search Filter Syntax* de ADSI dice que *«LDAP define **dos** matching rule OIDs»* y acto seguido tabula **tres**, omitiendo el `.2253`. La fuente autoritativa es [MS-ADTS §3.1.1.3.4.4](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/4e638665-f466-4597-93c4-12f2ebfabab5), que documenta los **cuatro**. HTB solo enseña los tres primeros — correctos, pero incompletos. Dato que casi nadie cita: <mark style="background: #FFB8EBA6;">el DC **no expone** ningún atributo que diga qué matching rules soporta</mark> (al contrario que los controles, listados en `supportedControl`); hay que probarlas.

El `BIT_AND` es el caballo de batalla de la enumeración ofensiva. Buscar cuentas deshabilitadas (`ACCOUNTDISABLE` = bit 2) es:

```powershell-session
PS C:\> Get-ADUser -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=2)' | select name
```

El `IN_CHAIN` resuelve la pertenencia **anidada**, invisible a una consulta normal de `memberOf`. Si `bob` está en `Security Operations` y ese grupo está en `Domain Admins`, `bob` es Domain Admin efectivo aunque no lo diga su `memberOf` directo. Para listar **todos** los grupos de un usuario, directos e indirectos:

```powershell-session
PS C:\> Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)' | select Name
```

> [!important]+ El `IN_CHAIN` sobre subtree es caro y se registra
> La propia documentación de Microsoft avisa de que las consultas transitivas con mucho *fan-out* son intensivas en CPU y pueden quedar registradas como búsqueda «cara» (evento 1644 — ver [[16 - Detección y evasión de la enumeración LDAP]]). Acotar el ámbito antes de lanzarlas.

# PowerShell: `Filter` vs `LDAPFilter`

Los cmdlets del módulo `ActiveDirectory` aceptan dos parámetros de filtrado:

- **`-Filter`** usa la sintaxis de PowerShell (`-eq`, `-like`, `-and`), más cómoda para búsquedas simples: `Get-ADUser -Filter "name -eq 'sally jones'"`.
- **`-LDAPFilter`** acepta la sintaxis LDAP cruda de RFC 4515, imprescindible para las matching rules y los filtros complejos: `Get-ADUser -LDAPFilter '(servicePrincipalName=*)'`.

<mark style="background: #8000E1A6;">`-LDAPFilter` es el que importa dominar</mark>: es portable (sirve igual en `ldapsearch`, PowerView, BloodHound), expresa lo que `-Filter` no puede (operaciones de bits), y es lo que de verdad viaja al DC.

# Acotar el ámbito: SearchBase y SearchScope

Dos parámetros reducen el volumen devuelto y el coste de la consulta:

- **`SearchBase`** fija el DN desde el que buscar: `"OU=Employees,DC=INLANEFREIGHT,DC=LOCAL"`.
- **`SearchScope`** fija la profundidad:

| Nombre | Nº | Qué recorre |
| --- | --- | --- |
| `Base` | 0 | Solo el objeto del `SearchBase` |
| `OneLevel` | 1 | Los hijos directos, sin descender |
| `SubTree` | 2 | El `SearchBase` y todos sus descendientes, recursivo |

Acotar no es solo rendimiento: en un dominio con decenas de miles de objetos, un `SubTree` desde la raíz con `(objectClass=*)` es la consulta que más delata al atacante. Pedir solo lo necesario, desde la base más concreta posible, es la primera medida de sigilo — desarrollada en [[16 - Detección y evasión de la enumeración LDAP]].

Con la sintaxis dominada, el siguiente paso es el catálogo de filtros que de verdad dan botín en un pentest.

> [!info]+ Fuentes
> Sintaxis: [RFC 4515](https://www.rfc-editor.org/info/rfc4515) y [Search Filter Syntax (ADSI)](https://learn.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax). Matching rules: [MS-ADTS §3.1.1.3.4.4](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/4e638665-f466-4597-93c4-12f2ebfabab5). Cmdlets: [ActiveDirectory module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/). Módulo *Active Directory LDAP* de HTB Academy, corregido (4 OIDs) y ampliado con las implicaciones de detección.
