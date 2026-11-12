---
tags:
  - OSINT/Frameworks
  - Pentesting/Enumeracion
Descripción: "El framework modular recon-ng (workspaces, marketplace) y Maltego para análisis de enlaces: cómo se usan y para qué sirve cada uno"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - SpiderFoot]]"
Nota siguiente: "[[03 - Pipelines, OSINT asistido por IA y deconfliction]]"
Area: "[[Frameworks OSINT.base|Frameworks OSINT]]"
---
---

Frente al "todo automático" de [[01 - SpiderFoot|SpiderFoot]], estos dos dan **control**: recon-ng te deja elegir y encadenar módulos; Maltego te deja **ver** las relaciones en un grafo que exploras. Son complementarios.

# recon-ng: el framework modular

<mark style="background: #ADCCFFA6;">`recon-ng` es un framework de recon con la ergonomía de Metasploit</mark>: una consola interactiva, módulos por categoría, *workspaces* por engagement y una base de datos que acumula los hallazgos. Tú decides qué módulo correr y sobre qué.

```shell-session
$ recon-ng
# Espacio de trabajo aislado para este cliente
[recon-ng] > workspaces create acme
# Instalar módulos desde el marketplace
[recon-ng] > marketplace install all
# Cargar un módulo (p. ej. subdominios vía fuentes pasivas)
[recon-ng] > modules load recon/domains-hosts/hackertarget
[...hackertarget] > options set SOURCE acme.com
[...hackertarget] > run
# Los hosts quedan en la BBDD; otro módulo los consume
[recon-ng] > modules load recon/hosts-hosts/resolve
[...resolve] > run
```

<mark style="background: #FFB86CA6;">La gracia es la **BBDD compartida**</mark>: los subdominios que encuentra un módulo los consume el siguiente (resolver IPs, buscar en Shodan, sacar contactos). Encadenas módulos sobre datos acumulados, y al final exportas (`reporting` modules) a HTML/CSV/JSON.

> [!warning]+ recon-ng está estancado
> El repo **no recibe commits desde finales de 2024**. Sigue funcionando y su flujo de trabajo es didáctico, pero para **enumeración de infraestructura** moderna conviene **OWASP Amass** (mantenido, más potente en DNS/ASN) o la suite [[Tools.base|ProjectDiscovery]]. Usa recon-ng por su modelo de módulos+BBDD, no como tu motor de recon de infra principal en 2026.

# Maltego: análisis de enlaces visual

<mark style="background: #ADCCFFA6;">Maltego no recolecta por su cuenta: **visualiza** entidades y relaciones en un grafo interactivo que expandes con *transforms*</mark>. Un *transform* es una consulta que, dada una entidad, devuelve las relacionadas: de un **dominio** → sus subdominios, correos, DNS; de un **correo** → brechas, cuentas; de una **persona** → sus perfiles, afiliaciones.

El flujo de trabajo:

1. Arrastras una **entidad** semilla al lienzo (un dominio, una persona, un correo).
2. Ejecutas *transforms* sobre ella → aparecen las entidades relacionadas, enlazadas.
3. Ejecutas más *transforms* sobre las nuevas → el grafo crece y revela la **estructura**.

<mark style="background: #8000E1A6;">Su valor es ver lo que una lista no muestra</mark>: clústeres, nodos centrales, el empleado que conecta dos departamentos, la infraestructura compartida entre dos dominios. Es la herramienta por excelencia para el [[05 - Análisis de red, actividad y correlación|análisis de red]] de personas y para mapear estructuras corporativas.

- **Maltego CE** (Community Edition) es **gratis** con *transforms* y resultados limitados por consulta — suficiente para aprender y para grafos pequeños.
- Las ediciones de pago amplían *transforms*, integraciones (incluida la de SpiderFoot y fuentes comerciales) y tamaño de grafo.

# Cuándo cada uno

| Necesidad | Herramienta |
| - | - |
| Footprint amplio automático | [[01 - SpiderFoot\|SpiderFoot]] |
| Recon controlado, módulo a módulo, con BBDD | recon-ng (+ Amass para infra) |
| **Ver** y explorar relaciones | Maltego |
| Enumeración de infra moderna | Amass / [[Tools.base\|ProjectDiscovery]] |

<mark style="background: #FFB86CA6;">Se combinan</mark>: recolectas con SpiderFoot/recon-ng/ProjectDiscovery y **visualizas** el resultado en Maltego para encontrar los pivotes que una tabla esconde.

> [!info]+ Fuentes
> [recon-ng](https://github.com/lanmaster53/recon-ng) (lanmaster53; sin commits desde finales de 2024). [OWASP Amass](https://github.com/owasp-amass/amass) como alternativa de infra. [Maltego](https://www.maltego.com) (CE gratuito con límites).
