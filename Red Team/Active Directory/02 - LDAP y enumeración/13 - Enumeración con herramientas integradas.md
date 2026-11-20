---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
Descripción: Enumerar AD living off the land con RSAT, el módulo ActiveDirectory, dsquery, CIM/WMI y ADSI, y por qué el módulo habla ADWS y no LDAP directo
Fecha de actualización: 2026-11-20
Nota previa: "[[12 - Filtros de alto valor para el atacante]]"
Nota siguiente: "[[14 - Bind anónimo y enumeración sin credenciales]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

Las herramientas que un administrador usaría a diario —el módulo `ActiveDirectory`, los Sysinternals, `dsquery`— suelen estar en la *allowlist* del EDR y pasan desapercibidas en entornos maduros. <mark style="background: #ADCCFFA6;">Enumerar con binarios integrados es *living off the land*</mark>: no sube herramientas, no toca disco con nada marcado y se mimetiza con la actividad legítima. Conocer varias vías —«offense in depth»— importa cuando una está bloqueada o monitorizada.

# RSAT: las herramientas de administración remota

**RSAT** (Remote Server Administration Tools) trae el módulo de PowerShell de AD, los snap-ins MMC (`Active Directory Users and Computers`, `ADSI Edit`) y las herramientas de línea de comandos. Instalación en Windows 11 (actualizada a 2026):

```powershell-session
PS C:\> Get-WindowsCapability -Online | Where-Object Name -like 'RSAT*'
PS C:\> Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```

El nombre de la *capability* es exactamente `Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0` (cuatro tildes), y requiere Windows **Pro o Enterprise** con privilegios de administrador — Home no.

> [!danger]+ Errata de HTB: el script de gist para Windows 10 1809/1903/1909
> El módulo instala RSAT con un script de gist para builds antiguas. <mark style="background: #FF5582A6;">Está obsoleto</mark>: desde Windows 10 1809 RSAT es *Features on Demand*, integrado en el sistema, y se instala con el one-liner de arriba o por *Optional Features* en la GUI. Las builds 1903/1909 están además fuera de soporte. Y un matiz ofensivo: **instalar RSAT en el host comprometido es ruidoso y casi siempre innecesario** — el módulo `ActiveDirectory` habla ADWS, que MDI ya detecta (ver abajo y [[16 - Detección y evasión de la enumeración LDAP]]).

# El módulo ActiveDirectory habla ADWS, no LDAP

Punto que HTB no menciona y que cambia por completo el perfil de detección: <mark style="background: #FFB86CA6;">los cmdlets `Get-AD*` **no hablan LDAP directo**; hablan ADWS (Active Directory Web Services, puerto 9389)</mark>. El DC traduce la petición ADWS a LDAP localmente. Consecuencias:

- Los sensores de red en 389/636 no ven las consultas de `Get-ADUser`.
- Pero el DC registra la consulta igual en el evento 1644… con el cliente como `[::1]`/`127.0.0.1`, no la IP real del atacante (la IP real sale en los eventos 1138/1139 de ADWS).
- Microsoft Defender for Identity tiene detección **dedicada** a la enumeración por ADWS (`Possible Active Directory enumeration via ADWS`).

O sea: ADWS dificulta la *atribución*, no la *detección*. Con eso en mente, las consultas con el módulo:

```powershell-session
PS C:\> Get-ADUser -Filter "adminCount -eq '1'" -Properties servicePrincipalName | where servicePrincipalName -ne $null | select SamAccountName, ServicePrincipalName
PS C:\> Get-ADGroup -LDAPFilter '(adminCount=1)' | select Name
```

# dsquery, CIM/WMI y ADSI

Varias vías integradas más, por si el módulo no está disponible:

**`dsquery`** — presente por defecto en todos los Windows modernos (requiere conectividad con el dominio):
```cmd-session
C:\> dsquery user "OU=Employees,DC=inlanefreight,DC=local" -name * -limit 0 | dsget user -samid -pwdneverexpires
```

**CIM/WMI** — consultar objetos del dominio por WMI:
```powershell-session
PS C:\> Get-CimInstance -ClassName win32_group -Filter "Domain='INLANEFREIGHT'" | Select Caption, Name
```

> [!warning]+ Errata de HTB: `Get-WmiObject` está muerto
> El módulo enumera con `Get-WmiObject`. <mark style="background: #FF5582A6;">Está superseded desde PowerShell 3.0 (2012) por `Get-CimInstance`, y **eliminado** en PowerShell 6/7</mark> — solo sobrevive en Windows PowerShell 5.1. La diferencia no es cosmética: `Get-WmiObject` usa DCOM/RPC (puerto 135 + efímeros), `Get-CimInstance` usa WS-Man/WinRM (5985/5986) por defecto. **Perfil de red y de detección completamente distinto.** Como 5.1 está en todos los Windows, `Get-WmiObject` sigue sirviendo como técnica LotL, pero el código nuevo se escribe con `Get-CimInstance`.

**ADSI** — interfaz COM de bajo nivel, vía el acelerador `[adsisearcher]`:
```powershell-session
PS C:\> ([adsisearcher]"(&(objectClass=Computer))").FindAll() | select Path
```
`[adsisearcher]` habla LDAP directo (389), al contrario que el módulo `ActiveDirectory` — útil precisamente cuando se quiere evitar ADWS.

# Interpretar `userAccountControl`

Las consultas devuelven `userAccountControl` como un entero (`512`, `66048`…) que hay que descomponer en sus flags (ver [[12 - Filtros de alto valor para el atacante]]). PowerView lo hace legible de serie:

```powershell-session
PS C:\> Get-DomainUser * -AdminCount | select samaccountname, useraccountcontrol
jenna.smith   PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
```

Una cuenta con `DONT_REQ_PREAUTH` es AS-REP roasteable; con `PASSWD_NOTREQD`, candidata a contraseña vacía. El estado de PowerView como herramienta (su repo original está archivado) y sus sustitutos vivos, en [[17 - Arsenal de enumeración LDAP]].

Estas vías integradas son útiles pero lentas en un dominio grande. Cuando no hay credenciales todavía, el punto de partida es el bind anónimo y el `rootDSE`, en la nota siguiente.

> [!info]+ Fuentes
> RSAT: [Install and Manage RSAT](https://learn.microsoft.com/en-us/windows-server/administration/install-remote-server-administration-tools). `Get-WmiObject` superseded: [docs del cmdlet](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject). ADWS como vector sigiloso y su detección: investigación de [SpecterOps (SOAPy)](https://specterops.io/blog/2025/07/25/make-sure-to-use-soapy-an-operators-guide-to-stealthy-ad-collection-using-adws/) y [Huntress](https://www.huntress.com/blog/ldap-active-directory-detection-part-5a). Módulo *Active Directory LDAP* de HTB Academy, modernizado (RSAT 2026, ADWS, `Get-CimInstance`).
