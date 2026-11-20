---
tags:
  - Active-Directory
  - Windows
  - Pentesting/Enumeracion
  - Tipo/Deteccion
Descripción: "Cómo enumerar AD sin disparar las alertas de reconocimiento: BloodHound/SharpHound sigiloso, enumeración de sesiones, ritmo y qué eventos evitar"
Fecha de actualización: 2026-11-20
Nota previa: "[[39 - OPSEC en AD - el modelo de detección y el mindset]]"
Nota siguiente: "[[41 - OPSEC de Kerberos]]"
Area: "[[Evasión y OPSEC.base|Evasión y OPSEC]]"
---
---

El reconocimiento es la fase **más barata de arruinar y la más fácil de hacer bien**: casi todo el ruido de un engagement de AD sale de recolecciones masivas lanzadas sin pensar. Y es lo primero que modela MDI, precisamente porque el recon precede a todo. La enumeración LDAP fina (filtros, paginación, `SD_FLAGS`, eventos `1644`/`4662`) ya está en [[16 - Detección y evasión de la enumeración LDAP]]; esta nota es la visión AD-completa: BloodHound, sesiones, ritmo y las alertas concretas que hay que no despertar.

# BloodHound/SharpHound: el mayor generador de ruido

La recolección para BloodHound es, de lejos, lo que más alertas de reconocimiento dispara, por dos motivos: toca **el DC** (consultas LDAP masivas → MDI) y **todos los hosts** (si pide sesiones y admins locales → conexiones a cada equipo). El control está en los métodos de recolección:

| Método de SharpHound | Qué toca | Ruido |
| --- | --- | --- |
| `DCOnly` | **Solo** el DC (LDAP + GPO) | Bajo: una sesión LDAP, sin tocar hosts. Pierde datos de sesión/admin local |
| `Default` | DC + conexiones a cada host (sesiones, admins locales) | **Alto**: una conexión a cada equipo del dominio |
| `Session` / `LoggedOn` | Enumera sesiones activas host a host | **Muy alto** y lo más valioso a la vez |

<mark style="background: #FF5582A6;">La regla de oro: empieza con `DCOnly`</mark>. Da el grafo de ACLs, grupos, delegaciones y trusts —el 80% de las rutas— con una sola sesión LDAP contra el DC y sin tocar un solo host. Las sesiones (quién tiene sesión dónde, el dato que conecta "controlo a X" con "X es admin donde hay un DA") se recogen **después y dirigidas**, no barriendo:

```shell-session
# Grafo base, bajo ruido
SharpHound.exe --collectionmethods DCOnly
# Sesiones, pero solo donde importa y con ritmo
SharpHound.exe --collectionmethods Session --throttle 1500 --jitter 30
```

`--throttle` (espera entre peticiones) y `--jitter` (variación aleatoria) rompen el patrón de "ráfaga" que delata a un escáner. <mark style="background: #FFB8EBA6;">Evita siempre `--collectallproperties`</mark>: pide todos los atributos de cada objeto y el DC lo registra con el prefijo `[all_with_list]` en el evento `1644` — la firma que Huntress llama *"a dead giveaway"* de un volcado automatizado (detalle en [[16 - Detección y evasión de la enumeración LDAP]]).

> [!warning]+ Lo que MDI caza del recon, con nombre
> Microsoft Defender for Identity trae detecciones **dedicadas** al reconocimiento, no genéricas: *Security principal reconnaissance (LDAP)* (ID 2038, la antesala del Kerberoasting), *Account enumeration reconnaissance*, *User and Group membership reconnaissance (SAMR)*. Las que llevan **periodo de aprendizaje** (15 días por equipo la 2038, 4 semanas por DC la de SAMR) **no disparan los primeros días** desde un host nuevo — una ventana real que CRTO espera que conozcas. Las de **honeytoken** (ver [[44 - Movimiento lateral, ejecución y deception]]) no tienen aprendizaje y disparan a la primera.

# Enumeración de sesiones sin barrer

Saber *dónde tiene sesión un administrador* es la pieza que convierte el grafo en un plan de movimiento lateral, y a la vez la más ruidosa de recoger. Alternativas por orden de sigilo:

- **El propio grafo** primero: BloodHound ya marca sesiones conocidas; úsalo antes de ir a buscar más.
- **Consulta dirigida** a hosts concretos (los que el grafo dice que importan), no a todos: `NetSessionEnum`/`NetWkstaUserEnum` contra 3 servidores clave hace poco ruido; contra los 2.000 del dominio, es un barrido.
- **Desde el DC, no desde los hosts**: en Windows Server 2019+ y con el parche adecuado, la enumeración remota de sesiones (`NetSessionEnum`) exige privilegios que un usuario normal ya no tiene por defecto — otra razón para apoyarse en el grafo y en fuentes menos vigiladas.

# El principio transversal

Toda la nota se reduce a aplicar las tres preguntas de [[39 - OPSEC en AD - el modelo de detección y el mindset]] al recon:

1. **Pide lo mínimo**: `DCOnly` antes que `Default`, atributos por nombre antes que `*`, el sub-árbol concreto antes que el dominio entero.
2. **Desde quién importa**: el recon desde una cuenta de administración/servicio que ya consulta AD a diario se pierde en el ruido de fondo; desde una cuenta recién comprometida de un usuario que nunca lo hace, destaca.
3. **Con ritmo humano**: `throttle` + `jitter`, repartido en el tiempo, nunca una ráfaga.

Y una verdad incómoda para cerrar: <mark style="background: #8000E1A6;">en un entorno con MDI bien desplegado, un grafo completo de BloodHound **se va a notar**</mark>. La decisión no es "cómo lo hago invisible" sino "qué me llevo con el menor ruido y cuánto puedo reconstruir pasivamente" — a menudo un `DCOnly` puntual más el análisis offline basta para planear toda la cadena sin volver a tocar el DC.

> [!info]+ Fuentes
> [SharpHound collection — flags](https://bloodhound.specterops.io/collect-data/ce-collection/sharphound-flags) y [DCOnly](https://bloodhound.specterops.io/), [MDI reconnaissance alerts](https://learn.microsoft.com/en-us/defender-for-identity/alerts-mdi-classic), serie [*From Code to Coverage* (Huntress)](https://www.huntress.com/blog/ldap-active-directory-detection-part-5a). Mecánica LDAP fina en [[16 - Detección y evasión de la enumeración LDAP]].
