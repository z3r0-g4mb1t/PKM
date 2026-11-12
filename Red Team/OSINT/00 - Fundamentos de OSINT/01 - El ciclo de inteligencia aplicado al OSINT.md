---
tags:
  - OSINT/Fundamentos
  - Pentesting/Enumeracion
Descripción: "Las cinco fases del ciclo de inteligencia aplicadas a un engagement, con un plan de recolección trabajado paso a paso"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - Qué es OSINT y por qué da ventaja]]"
Nota siguiente: "[[02 - Pasivo, semipasivo y activo - el gradiente de huella]]"
Area: "[[Fundamentos OSINT.base|Fundamentos OSINT]]"
---
---

El error más común del OSINT amateur es **recolectar sin plan**: abrir veinte pestañas, volcar nombres y correos en un `.txt` y acabar con un montón de datos que no responden a ninguna pregunta. El antídoto es el **ciclo de inteligencia**, el proceso que la doctrina militar (`JP 2-0`, `ATP 2-22.9`) usa para convertir requisitos en inteligencia, y que se aplica igual a un engagement.

<mark style="background: #ADCCFFA6;">El ciclo tiene cinco fases encadenadas y realimentadas: dirección, recolección, procesamiento, análisis y difusión</mark>. No es lineal: la difusión genera nuevos requisitos que reinician el ciclo.

```text
┌─────────────────────────────────────────────────────────┐
│                                                          │
▼                                                          │
Dirección ─► Recolección ─► Procesamiento ─► Análisis ─► Difusión
(requisitos)  (datos)        (normalizar)    (inteligencia) (entregar)
│                                                          ▲
└──────────────── feedback: nuevos requisitos ────────────┘
```

# 1. Dirección (planning & direction)

Se definen los **requisitos de inteligencia** — en la jerga, los `PIR` (*Priority Intelligence Requirements*): las preguntas concretas que el OSINT debe responder. <mark style="background: #FFB8EBA6;">Un requisito mal formulado ("averigua cosas de ACME") garantiza una recolección sin foco</mark>. Un requisito bien formulado es una pregunta con respuesta accionable.

De cada requisito sale un **plan de recolección**: qué fuentes, qué métodos, qué herramientas y qué límites (tiempo, alcance, OPSEC). Ejemplo para un engagement de *red team* cuyo objetivo global es **conseguir acceso inicial a la red corporativa de ACME**:

| PIR (pregunta) | Fuentes / método | Qué produce |
| - | - | - |
| ¿Qué empleados y qué correos tiene ACME? | LinkedIn, `theHarvester`, patrón de correo (`hunter.io`) | Lista de usuarios y emails |
| ¿Qué tecnología de acceso remoto expone? (VPN, OWA, Citrix) | Buscadores, certificados, `Shodan` | Portal de login objetivo |
| ¿Hay credenciales de ACME en filtraciones? | HIBP, Dehashed, `stealer logs` | Credenciales para *spraying* |
| ¿Qué pretextos sirven para *phishing*? | RRSS, notas de prensa, ofertas de empleo | Guion de ingeniería social |

<mark style="background: #FF5582A6;">Fíjate en que cada PIR apunta directamente a un paso del ataque</mark>: la lista de correos alimenta el *spray*, el portal es el blanco, las credenciales filtradas son la munición, el pretexto es el phishing. Eso es dirección: recolectar *hacia* un objetivo.

# 2. Recolección (collection)

Ejecutar el plan. Suele ser la fase **más larga**, sobre todo si se hace a mano. Aquí se decide el equilibrio pasivo/activo ([[02 - Pasivo, semipasivo y activo - el gradiente de huella|siguiente nota]]) y entra el OPSEC del investigador ([[OPSEC OSINT.base|sock puppets e infraestructura]]): recolectar genera rastro en las plataformas consultadas, y una cuenta real tuya logueada en LinkedIn mirando 300 perfiles de ACME es un patrón visible.

Regla de oro: **registra la procedencia de cada dato en el momento de recolectarlo** (fuente, URL, fecha, captura). Un dato sin procedencia no es verificable y no sirve para un informe.

# 3. Procesamiento (processing)

Convertir el material crudo en algo utilizable: normalizar correos a un formato (`nombre.apellido@acme.com`), deduplicar, traducir, transcribir, extraer metadatos, estructurar en una hoja o un grafo. <mark style="background: #FFB8EBA6;">Sin procesamiento, el análisis se ahoga en ruido</mark>. En OSINT de volumen, esta fase es la que más se beneficia de la automatización (ver [[Frameworks OSINT.base|frameworks]]).

# 4. Análisis y producción (analysis)

Dar **significado** a los datos procesados: cruzarlos, establecer relaciones, priorizar. El dato "Juan García, SysAdmin" + el dato "jgarcia@acme.com aparece en la brecha de Dropbox 2012" + el dato "ACME expone un portal OWA" se cruzan en una hipótesis accionable: <mark style="background: #8000E1A6;">*spray* la contraseña filtrada de Juan (y sus variantes) contra el OWA</mark>. Aquí se evalúa también la **fiabilidad** de cada fuente y la confianza en la conclusión (ver [[05 - Verificación, fiabilidad de fuentes y sesgos]]).

# 5. Difusión (dissemination)

Entregar la inteligencia a quien la va a usar — en un engagement, a la fase de acceso o al informe. La difusión **cierra y reabre** el ciclo: si el *spray* falla porque las contraseñas estaban rotadas, eso es *feedback* que genera un requisito nuevo ("¿qué política de contraseñas y MFA tiene ACME?") y el ciclo recomienza.

> [!important]+ Por qué esto importa en la práctica
> El ciclo evita las dos patologías del OSINT: la **parálisis por recolección** (recoger sin parar sin producir nada) y el **salto al análisis** (concluir sin datos suficientes). Un operador disciplinado siempre sabe responder: *¿qué pregunta estoy intentando contestar con esta búsqueda?* Si no hay respuesta, está perdiendo el tiempo.

> [!info]+ Fuentes
> Ciclo de inteligencia doctrinal: `JP 2-0 Joint Intelligence` y `ATP 2-22.9` (US Army). La formulación de `PIR` y planes de recolección es estándar en inteligencia; su traslado al pentest es el precedente de la fase `Information Gathering` ([[05 - Recopilación de información]]).
