---
tags:
  - OSINT/Corporativo
  - Pentesting/Enumeracion
Descripción: "Fuentes oficiales para la identidad legal, propietarios, administradores y filiales: OpenCorporates, BORME, Companies House, SEC EDGAR y grafos de propiedad"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - OSINT corporativo - mapear la organización y su superficie]]"
Nota siguiente: "[[02 - Tech stack y activos digitales]]"
Area: "[[OSINT corporativo.base|OSINT corporativo]]"
---
---

Los registros mercantiles son fuentes **oficiales** (fiabilidad `A` en el [[03 - Taxonomía de fuentes y disciplinas OSINT#Fiabilidad: el sistema Admiralty|código Admiralty]]) que dan la identidad legal, los administradores, la propiedad y las filiales. Es la forma de pasar de "ACME" a su estructura real.

# El agregador global: OpenCorporates

<mark style="background: #ADCCFFA6;">`OpenCorporates` es la mayor base de datos abierta de empresas del mundo</mark>: empresas, administradores, registros y filiales **cross-jurisdicción**. Es el punto de partida cuando no sabes dónde está registrada una empresa o quieres mapear un grupo multinacional. Busca por nombre o por administrador, y salta de una empresa a las demás que comparten un director — así afloran las filiales y las sociedades instrumentales.

# España: BORME y Registro Mercantil

- <mark style="background: #FFB86CA6;">**BORME** (Boletín Oficial del Registro Mercantil)</mark>: publica nombramientos y ceses de **administradores y apoderados**, constituciones y cambios societarios. Es gratuito y conecta **personas con empresas** — clave para cruzar con el [[04 - Buscadores de personas y registros públicos|OSINT de personas]].
- **Registro Mercantil Central**: datos societarios (objeto social, capital, domicilio); algunos documentos (cuentas anuales) son de pago.
- **Identificadores**: el CIF/NIF identifica la entidad; el `LEI` la identifica globalmente en operaciones financieras.

# Otras jurisdicciones clave

| Fuente | País | Qué da |
| - | - | - |
| **Companies House** | Reino Unido | Gratis, muy completo: administradores, cuentas, *persons with significant control* |
| **SEC EDGAR** | EE. UU. (cotizadas) | Informes primarios (10-K, 8-K): finanzas, riesgos, operaciones, propiedad |
| **North Data** | Alemania/UE | Red de empresas y personas, participaciones |
| **OCCRP Aleph** | Global/investigativo | Registros + **documentos filtrados** indexados |

<mark style="background: #FF5582A6;">El **10-K** de EDGAR es oro para el red teamer</mark>: en el apartado de "Risk Factors" y en las notas, una empresa cotizada estadounidense describe sus sistemas, sus adquisiciones recientes y a veces sus incidentes de seguridad — información que ellos mismos están obligados a publicar.

# Grafos de propiedad y filiales

Para empresas con estructuras complejas (holdings, filiales en varios países), los grafos de propiedad aceleran el mapeo:

- **Sayari** (comercial): grafo de redes corporativas y riesgo comercial a través de registros globales.
- **North Data** y **OpenCorporates**: permiten recorrer la cadena matriz → filiales.

<mark style="background: #8000E1A6;">Cada filial descubierta es superficie de ataque nueva</mark>: su propio dominio, sus propios empleados, su propia (y a menudo peor) seguridad. Las filiales recién adquiridas son especialmente jugosas — se tratan en [[03 - Cadena de suministro, filiales y M&A]].

# Qué extraer para el engagement

- **Administradores y directivos**: nombres reales → [[04 - Buscadores de personas y registros públicos|personas]] y objetivos de *whaling*.
- **Domicilios sociales**: ubicaciones físicas → recon físico y GEOINT.
- **Filiales y marcas**: expanden el alcance de activos digitales.
- **Finanzas y adquisiciones**: contexto para pretextos (una ronda de financiación reciente, una compra) y para entender prioridades.

> [!warning]+ Fuente oficial ≠ fuente actualizada
> Los registros tienen retardo: un administrador puede haber cesado y el BORME tardar en reflejarlo; las cuentas pueden ser del ejercicio anterior. Son `A` en fiabilidad de origen pero pueden ir desfasados — corrobora los nombres con [[01 - LinkedIn - la mina del red teamer|LinkedIn]] (que suele estar más al día).

> [!info]+ Fuentes
> [OpenCorporates](https://opencorporates.com), [BORME](https://www.boe.es/diario_borme/), [Companies House](https://find-and-update.company-information.service.gov.uk/), [SEC EDGAR](https://www.sec.gov/edgar), North Data, [OCCRP Aleph](https://aleph.occrp.org). Marco: [osintbench, *Corporate and Financial OSINT*](https://osintbench.com/guides/financial-osint/).
