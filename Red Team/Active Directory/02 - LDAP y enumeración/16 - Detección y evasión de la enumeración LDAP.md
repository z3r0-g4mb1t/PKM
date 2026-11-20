---
tags:
  - Active-Directory
  - LDAP
  - Windows
  - Pentesting/Enumeracion
  - Tipo/Deteccion
Descripción: Qué telemetría deja una enumeración LDAP (eventos 1644, 4662, MDI) y cómo se evade hoy con paginación, atributos selectivos y SD_FLAGS
Fecha de actualización: 2026-11-20
Nota previa: "[[15 - Enumeración LDAP con credenciales]]"
Nota siguiente: "[[17 - Arsenal de enumeración LDAP]]"
Area: "[[AD LDAP.base|LDAP]]"
---
---

La enumeración LDAP es ruidosa si no se cuida, y en 2026 el defensor tiene telemetría dedicada a cazarla. Esta nota cubre las dos caras: qué rastro deja (para evadirlo) y cómo se detecta e impide la evasión (para configurarlo).

# Qué telemetría deja

| Evento | Qué registra | Condición |
| --- | --- | --- |
| **1644** | Búsqueda LDAP «cara/ineficiente/larga», con el **filtro y los atributos pedidos** | Requiere subir `15 Field Engineering` a 5 + umbrales |
| **4662** | Operación sobre un objeto (lectura/escritura de propiedad) | Solo si hay **SACL** en el objeto |
| **2889** | Bind sin firmar o simple bind en claro, con **IP e identidad** del cliente | Requiere `16 LDAP Interface Events` = 2 |
| **1138/1139** | Conexión y autenticación ADWS, con IP real y principal | ADWS (9389) |

<mark style="background: #FF5582A6;">El evento 1644 es la joya para el defensor y la pesadilla para el atacante</mark>: registra **el filtro exacto y la lista de atributos pedidos**, que casan carácter a carácter con el código de la herramienta. Por eso pedir `-Properties *` (que el DC registra con el prefijo `[all_with_list]`) es, en palabras de Huntress, *«a dead giveaway»*.

> [!warning]+ El 1644 está apagado por defecto — y ahí está medio juego
> Los umbrales de fábrica del 1644 son altos (búsqueda cara > 10 000 entradas visitadas; tiempo > 30 000 ms), y `15 Field Engineering` está en `0` salvo que alguien lo active. Microsoft insiste en activarlo *«solo durante troubleshooting»* por el coste, <mark style="background: #FFB86CA6;">así que en la práctica casi nunca está encendido</mark>. Una consulta paginada y selectiva ni se acerca a los umbrales. Esa es toda la evasión del 1644: no cruzar el umbral de «cara» ni pedir todos los atributos.

# Microsoft Defender for Identity: la capa que sí está encendida

MDI no depende de los umbrales del 1644 y trae detecciones de reconocimiento LDAP con nombre propio:

| Alerta | ID | Notas |
| --- | --- | --- |
| Security principal reconnaissance (LDAP) | 2038 | *«primera fase de un Kerberoasting»*. **Aprendizaje de 15 días por equipo** |
| Account Enumeration reconnaissance (LDAP) | 2437 | Nombra `ldapnomnom`; cLDAP |
| Honeytoken was queried via LDAP | 2429 | **Sin aprendizaje: dispara a la primera** |
| Possible Kerberoasting LDAP reconnaissance | `xdr_PossibleKerberoastingLdapRecon` | Modela «SPN + no krbtgt + no deshabilitadas» |
| Suspicious LDAP query targeting sensitive attributes | `xdr_SuspiciousSensitiveAttributeLdapQuery` | LAPS, gMSA, etc. |

<mark style="background: #8000E1A6;">Dos verdades que matan mitos</mark>: **(1)** «ADWS es un punto ciego» ya no es cierto — MDI tiene `Possible Active Directory enumeration via ADWS`; la ventaja de ADWS es de atribución, no de evasión. **(2)** Hay detección dedicada al reconocimiento **previo** al Kerberoasting, incluida una variante llamada literalmente *«stealthy LDAP search»* — el patrón del filtro Kerberoasteable está modelado.

Dato de evasión que sale de la propia doc: las alertas con **periodo de aprendizaje** (15 días la 2038, 4 semanas la 2021) no disparan en los primeros días desde un host nuevo; las de **honeytoken** (2429) no tienen aprendizaje y disparan siempre.

# Evasión realista (lo que funciona en 2026)

Ordenado por lo que de verdad reduce la detección:

1. **Paginar siempre** (control `1.2.840.113556.1.4.319`) — no por sigilo, sino porque sin ello hay truncamiento y datos falsos.
2. **Pedir atributos por nombre, nunca `*`** — esquiva `[all_with_list]` y no cruza el umbral de «cara».
3. **Filtros acotados por base DN**, evitar `(objectClass=*)` masivos y las consultas `IN_CHAIN` con mucho *fan-out*.
4. **`LDAP_SERVER_SD_FLAGS_OID` (`1.2.840.113556.1.4.801`) con `0x4`** cuando solo necesitas la DACL: reduce lo que viaja y cambia la firma registrada (BloodHound/PowerView piden el descriptor completo, que es una firma).
5. **Nunca usar atributos inexistentes como truco de filtro**: el DC reescribe `(!atributoFalso=*)` como `(!(FALSE))`, que es en sí una firma de herramienta.
6. **Ritmo y distribución**: respetar `MaxConnIdleTime`, y aprovechar los periodos de aprendizaje de MDI por host.
7. **ADWS** dificulta la atribución (no la detección); **cLDAP/UDP** es el único hueco de telemetría real, y solo vale para validar nombres de usuario.

La ofuscación por mayúsculas/minúsculas o espaciado **no sirve**: el DC normaliza antes de registrar.

# Perspectiva Blue Team: cómo detectar mejor e impedir la evasión

Lo que el defensor configura para cerrar los huecos de arriba:

- **Activar el logging de consultas caras** (`15 Field Engineering` = 5 + los tres umbrales, log de Directory Services a 200 MB), asumiendo el coste, al menos en los DCs críticos — es lo que ilumina el 1644 que el atacante cuenta con que esté apagado.
- **SACL de auditoría sobre *lectura* (`ReadProperty`), no solo escritura**, en los objetos sensibles → genera eventos 4662 de reconocimiento, que por defecto no existen.
- **Honeytokens**: cuentas señuelo etiquetadas en MDI, y meterlas en las *wordlists* probables → alertas 2429/2014 sin periodo de aprendizaje. Colgarles un SPN falso caza el reconocimiento de Kerberoasting antes de que empiece.
- **Auditoría de Filtering Platform (evento 5156)** en los DCs —desactivada por defecto— para atribuir ADWS y cLDAP-sobre-TCP.
- **Enviar `netlogon.log` al SIEM** (`nltest /dbflag:0x2080ffff`) — el único testigo del LDAP ping sobre UDP.
- **Exigir LDAP signing + channel binding** y aplicar el parche de octubre de 2026 (`CVE-2026-70229`): *«exigir solo uno deja un camino abierto»*. Monitorizar 2887/2889 para localizar los clientes sin firmar antes de forzar.
- **Políticas LDAP propias** (`queryPolicy`) para apretar `MaxPageSize`/`MaxQueryDuration`.

> [!warning]+ Errata de Microsoft en su propia tabla
> La tabla actual de la página de LDAP signing etiqueta el evento **2889** como *«Signed LDAP communications»*, lo contrario de su nombre simbólico (`DIRLOG_UNSIGNED_CLIENT_DETAILS`) y del texto del evento, que describe un bind **sin firmar**. Una regla SIEM escrita desde esa tabla alerta al revés. Gana la definición del evento.

> [!info]+ Fuentes
> Eventos: [AD diagnostic event logging (KB 314980)](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/configure-ad-and-lds-event-logging), [evento 4662](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4662). MDI: [alertas clásicas](https://learn.microsoft.com/en-us/defender-for-identity/alerts-mdi-classic) y [XDR](https://learn.microsoft.com/en-us/defender-for-identity/alerts-xdr). Evasión: serie *From Code to Coverage* de [Huntress](https://www.huntress.com/blog/ldap-active-directory-detection-part-5a) y guía OPSEC de [SpecterOps](https://specterops.io/blog/2025/07/25/make-sure-to-use-soapy-an-operators-guide-to-stealthy-ad-collection-using-adws/). `SD_FLAGS`: [MS-ADTS](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/932a7a8d-8c93-4448-8093-c79b7d9ba499). Net-new respecto a HTB, que no cubre la detección.
