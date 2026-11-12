---
tags:
  - OSINT/Frameworks
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "Cuándo automatizar el OSINT y cuándo no, qué clase de framework resuelve cada necesidad y el compromiso entre velocidad, sigilo y precisión"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - SpiderFoot]]"
Area: "[[Frameworks OSINT.base|Frameworks OSINT]]"
---
---

Las dimensiones anteriores enseñan el OSINT "a mano". A escala —un dominio con cientos de activos, un pipeline de recon continuo, decenas de selectores que vigilar— eso no es viable. Los frameworks automatizan la recolección y la correlación. Pero automatizar tiene un coste, y conviene saber cuándo vale la pena.

# Manual vs automatizado

| | **Manual** | **Automatizado** |
| - | - | - |
| Precisión | Alta (verificas cada paso) | Más falsos positivos |
| Sigilo | Controlas la huella | Ruidoso (muchas consultas) |
| Escala | Baja | Alta |
| Cuándo | Objetivo concreto, pivotes delicados | Superficie amplia, monitorización continua |

<mark style="background: #8000E1A6;">La regla: automatiza la **amplitud**, verifica la **profundidad** a mano</mark>. Un framework te da el mapa completo de la superficie en minutos; el pivote fino sobre una persona lo haces tú. Nunca reportes un hallazgo de un framework sin verificarlo ([[05 - Verificación, fiabilidad de fuentes y sesgos|verificación]]).

# Las tres clases de framework

1. <mark style="background: #ADCCFFA6;">**Agregadores autónomos**</mark> (SpiderFoot): les das una semilla (dominio, correo, IP, nombre) y, con 200+ módulos, recolectan y **correlacionan solos** en un grafo. Amplitud máxima, poco control fino → [[01 - SpiderFoot]].
2. <mark style="background: #ADCCFFA6;">**Frameworks modulares**</mark> (recon-ng): tú eliges y encadenas módulos, con *workspaces* y base de datos. Más control, más trabajo → [[02 - recon-ng y Maltego]].
3. <mark style="background: #ADCCFFA6;">**Análisis de enlaces**</mark> (Maltego): no recolecta solo; **visualiza** entidades y relaciones en un grafo que exploras y expandes con *transforms*. El estándar para mapear redes de personas/infra → [[02 - recon-ng y Maltego]].

Y, transversal a todas, el patrón de **pipeline de CLI** (encadenar herramientas Unix especializadas) y el **OSINT asistido por IA**, que se tratan en [[03 - Pipelines, OSINT asistido por IA y deconfliction]].

# El compromiso velocidad / sigilo / precisión

<mark style="background: #FFB86CA6;">Un framework que consulta 200 fuentes por tu objetivo es un megáfono</mark>: muchas de esas consultas tocan al objetivo o a terceros que podrían avisarle, y todas salen de tu infraestructura. Antes de lanzar un escaneo automático:

- **OPSEC**: ejecútalo desde la [[02 - Infraestructura de investigación - VM, red y navegador|infraestructura de investigación]], con las API keys de cuentas desechables. Un SpiderFoot a toda máquina desde tu IP es un patrón ruidoso.
- **Alcance**: un agregador puede salirse del scope (resolver subdominios, tocar servicios). Configúralo para quedarse en pasivo si el activo no está autorizado.
- **Verificación**: el output es materia prima, no inteligencia. Filtra y corrobora.

> [!important]+ El vault ya tiene medio arsenal de automatización
> Muchas herramientas de automatización OSINT ya viven en [[Tools.base|Tools/]]: la suite **ProjectDiscovery** (`subfinder`, `dnsx`, `httpx`, `naabu`, `asnmap`, `uncover`, `notify`), que es la base del patrón de [[03 - Pipelines, OSINT asistido por IA y deconfliction|pipeline]]. Esta sub-área cubre los **frameworks OSINT** propiamente (SpiderFoot, recon-ng, Maltego) y cómo orquestarlo todo.

> [!info]+ Fuentes
> Comparativa de frameworks OSINT: listas curadas de la comunidad (jivoi/awesome-osint). Compromiso automatización/sigilo: práctica estándar de recon.
