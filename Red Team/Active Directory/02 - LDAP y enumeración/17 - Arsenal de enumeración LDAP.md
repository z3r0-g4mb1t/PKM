---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
  - Tipo/Arsenal
Descripción: El set profesional de 2026 para enumerar AD por LDAP desde Linux y Windows, con el estado real de cada herramienta y qué sustituye a lo que HTB enseña
Fecha de actualización: 2026-11-20
Nota previa: "[[16 - Detección y evasión de la enumeración LDAP]]"
Nota siguiente: "[[18 - Envenenamiento LLMNR y NBT-NS]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

El material clásico de HTB enseña `windapsearch` y `ldapsearch-ad` como el arsenal de enumeración LDAP. En 2026 ese consejo está desfasado: la primera está abandonada y la segunda, estancada. Esta nota fija el set real, con el estado de cada herramienta verificado contra la API de GitHub (noviembre de 2026).

# Desde Linux

| Herramienta | Función | Estado 2026 |
| --- | --- | --- |
| **`ldeep`** | Enumeración LDAP completa con consultas predefinidas | 🟢 **Vivo y activo** (v2.0.4, nov-2026). La referencia moderna |
| **`NetExec` (`nxc ldap`)** | ~40 módulos LDAP (`laps`, `daclread`, `shadow-creds`, `maq`, `pre2k`, `obsolete`…) | 🟢 **El más activo del ecosistema** (commits a diario) |
| **`godap`** | Cliente LDAP interactivo (TUI) | 🟢 Vivo (v2.12.2) |
| **`pywerview` / `powerview.py`** | PowerView portado a Python | 🟢 Vivos; `powerview.py` incluye funciones de escritura |
| **`ldapdomaindump`** | Volcado rápido a HTML | 🟡 Mantenimiento mínimo, funciona |
| **`ldapnomnom`** | *Username enum* por cLDAP (sin credenciales) | 🟡 Estancado pero funcional; nicho concreto |
| **`Certipy`** | Enumeración y ataque a ADCS por LDAP | 🟢 Vivo (5.1.0) |
| ~~`windapsearch`~~ | — | 🔴 **Abandonado** (último commit 2020-05-24) |
| ~~`ldapsearch-ad`~~ | — | 🟠 Estancado ~2 años; no muerto, pero superado |

> [!danger]+ Errata de HTB: `windapsearch` como herramienta de referencia
> `windapsearch` lleva sin un commit en `master` desde **mayo de 2020**; su sucesor Go (`go-windapsearch`) desde 2021. Depende de `python-ldap` (binding C que hay que compilar) en lugar de `ldap3`, y <mark style="background: #FF5582A6;">su tabla de niveles funcionales se queda en «2016»: no reconoce el DFL/FFL **10** de Windows Server 2025</mark>, así que su opción estrella `--functionality` falla contra un dominio moderno. Reemplazo directo: `ldeep` + `NetExec ldap`.

`ldapsearch` de OpenLDAP sigue siendo útil para consultas puntuales a mano (con la paginación `-E pr=1000/noprompt`, ver [[15 - Enumeración LDAP con credenciales]]).

# Desde Windows

| Herramienta | Función | Estado 2026 |
| --- | --- | --- |
| **SharpHound + BloodHound CE** | Recolección por LDAP → grafo de caminos de ataque | 🟢 La referencia (SharpHound v2.18, BloodHound CE v9.10) |
| **Módulo `ActiveDirectory` (RSAT)** | Enumeración LotL vía ADWS | 🟢 Integrado; detectable (ver [[13 - Enumeración con herramientas integradas]]) |
| **`adPEAS`** | Barrido PowerShell «todo en uno» | 🟢 Muy activo (v2.11, nov-2026) |
| **`[adsisearcher]`** | LDAP directo (389) por COM, sin subir nada | 🟢 Nativo |
| **PingCastle** | Auditoría/postura de todo el dominio | 🟢 Vivo, pero **migrado a `netwrix/pingcastle`** |
| ~~PowerView (`PowerSploit`)~~ | — | 🔴 **Repo archivado** (2020) |

> [!danger]+ Errata de HTB: PowerView desde `PowerShellMafia/PowerSploit`
> El repo original de PowerView (`PowerSploit`, rama `dev`) está **archivado y de solo lectura desde 2020**, y **no hay un fork canónico vivo** con tracción. <mark style="background: #FFB86CA6;">PowerView sigue siendo didácticamente útil y sus funciones funcionan</mark>, pero como «herramienta de referencia mantenida» ya no existe. Sustitutos reales: `powerview.py`/`pywerview` (Python), BloodHound CE + SharpHound (grafo), `adPEAS` (barrido). En Windows, PowerView hoy se ejecuta desde copias embebidas en frameworks, no desde el repo original.

# ADWS como alternativa sigilosa

Para recolección con menos atribución, **SOAPy** (`logangoins/SOAPy`, Python, por proxy SOCKS de C2) recoge vía ADWS sin ejecutar un ensamblado .NET en el host. Sustituye a **SOAPHound** (`FalconForce`), que está **muerto** (feb-2024) y compila para BloodHound v4, incompatible con la rama CE actual. Recordatorio de [[16 - Detección y evasión de la enumeración LDAP]]: ADWS evita la atribución, no la detección — MDI ya lo caza.

# El criterio que sobrevive a las versiones

Las versiones cambian cada mes; el criterio no:

- **Enumeración desde Linux con credenciales** → `ldeep` o `NetExec`.
- **Mapa de caminos de ataque** → SharpHound → BloodHound CE.
- **Sin credenciales, validar usuarios** → `ldapnomnom` (cLDAP).
- **LotL en Windows** → módulo `ActiveDirectory` (asumiendo que ADWS deja huella) o `[adsisearcher]` (LDAP directo).
- **Postura del dominio entero** → PingCastle, `adPEAS`.

> [!warning]+ Un repo huérfano es blanco de suplantación
> Varias herramientas que HTB enlaza están abandonadas o migradas (`windapsearch`, `SOAPHound`, `PowerSploit`, `vletoux/pingcastle` → `netwrix/pingcastle`). Antes de clonar, verificar el repo contra la API de GitHub: un nombre de repositorio huérfano es un objetivo fácil de *typosquatting* y de *account takeover*.

> [!info]+ Fuentes
> Estado de cada herramienta verificado contra la API de GitHub el 2026-11-20 (metadata de repo + releases). Herramientas: [ldeep](https://github.com/franc-pentest/ldeep), [NetExec](https://github.com/Pennyw0rth/NetExec), [BloodHound CE](https://github.com/SpecterOps/BloodHound), [SOAPy](https://github.com/logangoins/SOAPy), [adPEAS](https://github.com/61106960/adPEAS). Net-new respecto al módulo *Active Directory LDAP* de HTB Academy, cuyo arsenal (windapsearch/ldapsearch-ad) está desfasado.
