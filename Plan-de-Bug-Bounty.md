# Plan de Bug Bounty a tiempo completo (2026)

## Resumen y expectativas

En 2026 el bug bounty sigue pagando, pero ya no paga volumen: paga validación e impacto demostrado, y castiga el ruido con bloqueos. Tu ventaja realista es web profunda (lógica de negocio, control de acceso, cadenas) más superficies de IA, en pocos programas y con informes impecables.

El plan en una línea: HackerOne e Intigriti como plataformas principales, YesWeHack como secundaria, 2–3 programas "casa" a la vez, 30–35 horas semanales y proteger tu reputación desde el primer informe para entrar en programas privados.

| Escenario (estimación propia) | Ingresos brutos en 12 meses | Qué tiene que pasar |
| --- | --- | --- |
| Conservador | 2.000–6.000 € | Solo programas públicos, muchos duplicados, sin invitaciones privadas antes del mes 8 |
| Base | 8.000–20.000 € | Primer High/Critical antes del mes 4 y 2–3 privados estables desde el mes 6 |
| Optimista | 40.000 € o más | Un programa "casa" muy rentable, varios Criticals y un evento en vivo |

Las cifras son una estimación orientativa, no un dato publicado. Los ingresos siguen una ley de potencias: pocos meses concentran casi todo y es normal pasar 4–8 semanas sin cobrar, así que conviene un colchón de al menos 6 meses de gastos.

## Situación del mercado en 2026

Descubrir bugs se ha abaratado con la IA; lo escaso ahora es validar, demostrar impacto y no hacer perder tiempo al triager. Todo el plan parte de esa idea.

**El dinero está concentrado.** HackerOne pagó 81 M$ entre julio de 2024 y junio de 2025, un 13 % más que el año anterior ([Cyber Security News](https://cybersecuritynews.com/hackerone-bug-bounty-paid/amp/)). Pero solo los 100 programas top pagaron 51 M$ de ese total ([Red Hot Cyber](https://www.redhotcyber.com/?p=135497)): hay que estar en los programas que pagan, no en muchos.

**La IA es superficie de ataque y competencia a la vez.** Los informes válidos sobre IA crecieron un 210 % y la inyección de prompts un 540 % ([HackerOne](https://www.hackerone.com/report/hacker-powered-security)). Un agente autónomo (XBOW) llegó al número 1 del ranking de EE. UU. de HackerOne en 2025 por volumen ([TechRepublic](https://www.techrepublic.com/article/news-ai-xbow-tops-hackerone-us-leaderboad/)), aunque los rankings siguen dominados por los mismos cazadores humanos ([LessWrong](https://www.lesswrong.com/posts/qefrWyeiMvWEFRitN/be-skeptical-of-milestone-announcements-by-young-ai-startups)).

**El "AI slop" ha cambiado las reglas.** curl cerró su bounty en enero de 2026 y Nextcloud en abril ([ForkLog](https://forklog.com/en/the-end-of-bug-bounties/)). Bugcrowd vio sus colas crecer un 334 % en tres semanas y endureció sus normas en marzo ([Bugcrowd](https://www.bugcrowd.com/blog/bugcrowd-policy-changes-to-address-ai-slop-submissions/)); Apple puede pausar 180 días a quien envíe informes de IA sin validar ([AI Weekly](https://aiweekly.co/alerts/apple-pauses-researchers-180-days-over-ai-generated-bug-reports)).

**Los públicos pagan menos; los privados y VIP, más.** El Internet Bug Bounty se pausó en abril y redujo un Critical de 9.250 $ a 2.257 $ ([The Register](https://www.theregister.com/security/2026/05/21/hackerone-takes-an-axe-to-its-bug-bounty-rewards/5244458)). GitHub fijó su programa público desde el 27 de julio en 250/2.000/5.000/10.000 $ y reserva pagos unas 3–4 veces mayores para su VIP por invitación ([SC Media](https://www.scworld.com/brief/github-to-implement-two-tier-bug-bounty-program-amid-ai-generated-report-surge), [Help Net Security](https://www.helpnetsecurity.com/2026/07/23/github-bug-bounty-program-changes/)).

**Entrar es más difícil sin reputación.** En HackerOne, si no llegas al Signal que pide un programa, solo puedes enviar informes de prueba: 4 por programa y 6 en toda la plataforma cada 30 días si tienes menos de 5 resueltos ([HackerOne Docs](https://docs.hackerone.com/hackers/restricted-from-submissions.html)). Node.js y GitHub ya exigen Signal ([Node.js](https://nodejs.org/en/blog/announcements/hackerone-signal-requirement)).

Qué significa para ti:

- Evita open source tipo IBB y programas públicos saturados de tarifa fija baja: el retorno por hora ha caído.
- Tu primer objetivo no es dinero, es reputación limpia: 5–10 informes válidos sin N/A para desbloquear privados.
- Usa IA para acelerar, nunca para enviar sin reproducir tú el fallo; es lo que más se sanciona en 2026.

## Plataformas: dónde cazar y cómo jugar en cada una

Concéntrate en dos plataformas principales y una secundaria: la reputación no se transfiere entre plataformas y repartirla te retrasa las invitaciones privadas.

| Plataforma | Papel en tu plan | Por qué |
| --- | --- | --- |
| HackerOne | Principal | Mayor volumen de programas y pagos; lo que más programas de IA tiene en alcance |
| Intigriti | Principal | Empresas europeas, recompensas en euros, menos competencia en muchos programas |
| YesWeHack | Secundaria | Clientes europeos y públicos, incluida España y la Comisión Europea |
| Bugcrowd | Selectiva | Solo programas concretos (p. ej. OpenAI); reglas anti-slop muy duras |
| Synack Red Team | Desde el mes 6–9 | Ingresos más estables, pero con proceso de selección |
| Programas propios de IA | Transversal | Google AI VRP, Anthropic, OpenAI, Microsoft: encajan con tu interés en IA |

### HackerOne

- **Tu Signal es tu llave.** Se calcula sobre los últimos 365 días y los programas pueden exigir un mínimo; sin él solo tienes informes de prueba limitados ([HackerOne Docs](https://docs.hackerone.com/hackers/restricted-from-submissions.html)). Cada N/A te cuesta acceso, no solo puntos.
- **Gasta los informes de prueba solo en hallazgos sólidos.** Con menos de 5 resueltos tienes 4 por programa y 6 en total cada 30 días; con más de 5 suben a 8 y 12.
- **Lee el Hacktivity del programa antes de empezar.** Los informes divulgados te dicen qué paga, qué es duplicado habitual y cómo razona el equipo.
- **Verifica tu identidad pronto** y, cuando puedas, solicita HackerOne Clear: abre programas empresariales y gubernamentales con menos competencia.
- **Evita el open source.** El Internet Bug Bounty está pausado y sus tablas bajaron más de un 75 % ([The Register](https://www.theregister.com/security/2026/05/21/hackerone-takes-an-axe-to-its-bug-bounty-rewards/5244458)).
- **Ojo con los públicos de dos niveles.** GitHub usa su público como cantera para el VIP; la barra para entrar se sitúa en un Critical confirmado o unos siete válidos de baja severidad ([TechRound](https://techround.co.uk/cybersecurity/how-ai-slop-is-forcing-github-to-close-its-doors/)).

### Intigriti

- **Declara siempre el uso de IA.** Su código de conducta lo exige y pide que la IA mejore la claridad, no sustituya la comprensión ([Intigriti KB](https://kb.intigriti.com/en/articles/5247238-community-code-of-conduct)).
- **Cuidado con traducir con IA.** Hay casos de suspensiones de 30 días por informes que "parecían IA" ([Reddit](https://reddit.sentinel-team.org/posts/1riql7d/snapshots/2026-03-03T02%3A34%3A55.31011Z)); revisa y reescribe con tu voz.
- **Hazte el ID-check.** Algunos programas solo son visibles para investigadores verificados ([Intigriti](https://intigriti.com/product/bug-bounty)).
- **Gana visibilidad con sus retos mensuales y CTF.** El equipo los usa para conocer cazadores, y en una plataforma más pequeña se nota antes.

### YesWeHack

- **Úsala para objetivos europeos y públicos.** Es proveedora preferente de bug bounty de la Comisión Europea con un marco de hasta 7,7 M€ ([Business Wire](https://www.businesswire.com/news/home/20251001559762/it)) y cuenta clientes en España ([SecurityWeek](https://www.securityweek.com/french-bug-bounty-platform-yeswehack-raises-28-million/amp/)).
- **Elige 1 programa "casa" aquí**, no más, mientras construyes en las principales.

### Bugcrowd

- **Reglas estrictas desde marzo de 2026:** 10 inválidos seguidos llevan a revisión y posible suspensión de 30 días; 10 inválidos en total obligan a verificar identidad; el "farming" es baneo permanente ([Bugcrowd](https://www.bugcrowd.com/blog/bugcrowd-policy-changes-to-address-ai-slop-submissions/)).
- **Merece la pena por programas concretos**, como el de seguridad y el de seguridad de IA de OpenAI ([SecurityWeek](https://www.securityweek.com/openai-launches-bug-bounty-program-for-abuse-and-safety-risks/amp/)).
- **Rellena tu perfil de habilidades** (web, API, IA): sus invitaciones se asignan por encaje de perfil.

### Synack Red Team

- **Es un equipo cerrado de más de 1.500 investigadores** ([Synack](https://www.synack.com/?p=20246)) con verificación de identidad y de antecedentes ([oferta SRT](https://resumegeni.com/jobs/synack-red-team-pentester-synacksrt-ca3bfd99fb)).
- **Solicítalo cuando tengas historial público** (informes divulgados, CVE, rankings); su programa Pathways prioriza certificaciones verificables ([Synack](https://www.synack.com/press-releases/synack-expands-crest-partnership-with-new-certification-pathways-for-the-synack-red-team/)).

### Programas de IA (tu nicho)

- **Google AI VRP:** paga sobre todo "acciones no autorizadas" e inyección indirecta con impacto real; hasta 20.000 $ base y 30.000 $ con multiplicadores. La inyección directa y los jailbreaks no cuentan ([The Register](https://www.theregister.com/2025/10/07/google_ai_bug_bounty/)).
- **Anthropic:** su programa es público en HackerOne desde mayo de 2026 ([GIGAZINE](https://gigazine.net/gsc_news/en/20260508-anthropic-bug-bounty-program)).
- **OpenAI:** programa de seguridad y otro de abuso y seguridad de IA, ambos en Bugcrowd ([SecurityWeek](https://www.securityweek.com/openai-launches-bug-bounty-program-for-abuse-and-safety-risks/amp/)).
- **El filón real son los SaaS normales con asistentes de IA.** HackerOne contó 1.121 programas con IA en alcance ([Red Hot Cyber](https://www.redhotcyber.com/?p=135497)): busca inyección indirecta que exfiltre datos o ejecute acciones de otro usuario.

**Fuera del plan por ahora:** Immunefi y Web3 (exigen auditar contratos inteligentes, otra especialidad) y móvil/IoT, que no encajan con tu foco.

## Selección de programas y nicho

Trabaja 2–3 programas "casa" a la vez durante 4–8 semanas cada uno: el conocimiento de una aplicación se acumula y es lo único que los bots no replican bien. Saltar de programa en programa te condena a competir en lo superficial.

| Criterio | Umbral orientativo (propio) | Por qué |
| --- | --- | --- |
| Tabla de recompensas | Critical ≥ 5.000 $, High ≥ 1.500 $ | Por debajo, un mes sin Critical no compensa |
| Tiempos de respuesta | Triaje < 7 días, pago < 30 días | Tu flujo de caja depende de ello |
| Superficie | App autenticada, varios roles o tenants, API documentada | Es donde vive el control de acceso y la lógica |
| Registro propio | Puedes crear 2+ cuentas y organizaciones | Imprescindible para probar IDOR y escaladas |
| Novedad | Funcionalidad nueva, adquisición o IA recién añadida | Código reciente, menos cazadores encima |
| Requisitos | Cumples el Signal o el ID que pide | Evita gastar informes de prueba |

**Tu nicho, por orden de rentabilidad esperada:**

1. Control de acceso y lógica de negocio en SaaS B2B: IDOR/BOLA entre tenants, escaladas de rol, flujos de pago y facturación. El 58 % de los investigadores encuestados cree que la IA falla justo en lógica y cadenas ([HackerOne](https://www.hackerone.com/report/hacker-powered-security)).
2. Autenticación federada: OAuth, OIDC, SAML, SSO, invitaciones y recuperación de cuenta.
3. Funciones de IA dentro de aplicaciones: inyección indirecta que lea datos de otro usuario o dispare acciones (enviar, borrar, compartir).
4. Técnicas de nivel CWEE que pocos dominan: desincronización HTTP, envenenamiento de caché, condiciones de carrera, deserialización.

**Lo que no debes perseguir:** XSS reflejado aislado, configuraciones por defecto, subdominios huérfanos o cabeceras. Es terreno de escáneres y agentes, con duplicados casi seguros.

**Regla de rotación:** si tras 2 semanas no tienes ni una pista sólida, cambia de programa. Si tienes una, quédate: las vulnerabilidades suelen aparecer en racimos alrededor de la misma lógica.

## Metodología de trabajo

La IA abarató encontrar candidatos; lo que paga es la validación humana. Tu metodología debe invertir tiempo justo donde el bot falla: entender la aplicación y confirmar el impacto.

**1. Recon dirigido, no masivo.** El recon amplio con nuclei contra todo es exactamente lo que hacen los agentes y produce duplicados y ruido. En su lugar: enumera a fondo un objetivo (subdominios, endpoints de API, parámetros, roles, flujos), mapea la superficie de autenticación y de autorización, y cataloga cada acción que un rol puede hacer sobre un recurso. Herramientas que ya conoces (Burp Suite, ffuf, Amass/Subfinder) sirven; la diferencia es la profundidad, no el volumen.

**2. Deep-dive por aplicación.** Crea 2–3 cuentas con distintos roles y organizaciones. Intercepta todo el tráfico con Burp y construye un mapa mental de "quién debería poder hacer qué". Después pon a prueba cada límite: IDOR entre tenants, cambios de rol, respuestas que filtran campos de más, endpoints que no revalidan permisos. Aquí es donde tu formación CWES/CWEE rinde.

**3. Bucle de testing.** Para cada hipótesis: reproduce → confirma impacto real → documenta con pasos mínimos → decide si es reportable o duplicado probable. No envíes nada que no hayas reproducido tú al menos dos veces.

**4. Uso de IA (clave en 2026).** Úsala como acelerador, nunca como emisor:

- Sí: resumir documentación, generar wordlists a medida, revisar tráfico en busca de patrones, redactar el borrador del informe que luego reescribes, crear scripts en Python/Go para automatizar pruebas repetitivas.
- No: enviar su salida sin reproducir, dejar que invente impacto, copiar su redacción tal cual (los triagers detectan los patrones LLM y sancionan).
- Declárala si la plataforma lo pide (Intigriti) y asume siempre tú la responsabilidad del informe.

**5. Tu ventaja como desarrollador.** Programas tus propias herramientas en Python y Go. Automatiza lo aburrido y repetible (monitorización de cambios de alcance, detección de nuevos subdominios, comparación de respuestas entre roles), pero deja el juicio para ti. Un monitor que te avise de funcionalidad nueva en tus programas "casa" vale más que un escáner genérico más.

**6. Monitoriza cambios.** Las vulnerabilidades aparecen cuando el código cambia. Vigila despliegues nuevos, adquisiciones y features de IA recién añadidas en tus programas "casa": ser el primero en mirar código nuevo es la forma más fiable de evitar duplicados.

## IA como herramienta: cómo, cuál y cuándo

Esta sección es sobre usar la IA como copiloto para ir más rápido, no sobre atacar sistemas de IA (eso es tu nicho ofensivo, otra cosa). El principio que repiten todos los cazadores top es "human in the loop": tú pilotas y decides, la IA ejecuta lo tedioso ([Wiz](https://www.wiz.io/bug-bounty-masterclass/foundations/ai-bug-bounty)).

**Por qué funciona (y dónde falla), con datos.** En una investigación interna de Wiz, modelos frontera resolvieron 9 de 10 retos basados en bugs reales, a 1–10 $ cada uno, pero su rendimiento se desplomaba con alcance amplio: se dispersaban entre objetivos y fallaban bugs que habrían encontrado si los apuntabas directamente. Además, la IA itera sobre el mismo enfoque mientras que el humano pivota: en un caso un agente hizo más de 500 llamadas en una hora sin encontrar nada, y un humano halló el fallo en 5 minutos ([Wiz](https://www.wiz.io/bug-bounty-masterclass/foundations/ai-bug-bounty)). Traducción: la IA brilla cuando le das un objetivo concreto y acotado; tu trabajo es encontrarlo y apuntarla.

### Qué LLM y herramientas usar

| Capa | Herramienta recomendada | Para qué |
| --- | --- | --- |
| LLM principal (agente) | Claude (vía Claude Code) | Lo que más usan los cazadores top; razona bien en seguridad y maneja contexto largo |
| LLM secundario (chat) | Gemini | Preguntas rápidas y análisis de archivos muy grandes por su ventana de contexto |
| Proxy + IA | Burp Suite Pro con Burp AI / Caido AI | Conectar el LLM a tu tráfico real (Repeater, análisis de peticiones) |
| Plantillas de detección | nuclei con generación por IA (`-ai`) | Convertir un PoC o CVE en plantilla de detección en minutos |
| Scripting a medida | Claude Code / Codex | Escribir y afinar tus herramientas en Python/Go |

En las entrevistas de YesWeHack a cazadores de élite, el consenso es claro: Claude es el LLM preferido, y Claude Code el agente de cabecera, conectado al proxy (Caido AI Skill) y con skills y agentes propios ([YesWeHack: Rhynorater](https://www.yeswehack.com/community/llms-bug-bounty-interview-rhynorater), [Aituglo](https://www.yeswehack.com/community/llms-bug-bounty-interview-aituglo)). Burp AI de PortSwigger viene con créditos gratis para empezar a experimentar ([CyberSecurityNews](https://cybersecuritynews.com/burp-ai/)).

### Cuándo SÍ usarla (casos concretos)

1. **Destripar JavaScript.** Pega un bundle minificado y pídele que actúe como revisor de seguridad: endpoints de API, secretos hardcodeados, funciones que tocan input sensible. Minutos en vez de horas ([Wiz](https://www.wiz.io/bug-bounty-masterclass/foundations/ai-bug-bounty)).
2. **Entender peticiones post-autenticación.** Vuelca una petición compleja con decenas de parámetros y pídele que explique cada uno y proponga vectores. Después los pruebas tú.
3. **Brainstorm de vectores.** "Tengo una subida de archivos que solo acepta imágenes, las redimensiona y convierte a PNG: ¿qué pruebas creativas haría?". Te da un checklist que tú filtras.
4. **Construir el exploit cuando ya tienes la pista.** El bug famoso de Rhynorater (XSS en un dominio top) salió de que Claude Code le quitó la fricción de explotar un formato protobuf complejo ([YesWeHack](https://www.yeswehack.com/community/llms-bug-bounty-interview-rhynorater)). La idea fue suya; la IA puso los "tornillos".
5. **Programar tus herramientas.** Tu ventaja con Python/Go se multiplica: que la IA escriba monitores de alcance, comparadores de respuestas entre roles o parsers. Tú revisas el código.
6. **Borrador de informe.** Que redacte el primer borrador; tú lo reescribes con tu voz y confirmas cada dato.

### Cuándo NO usarla

- **Recon masivo a ciegas.** Lanzarla contra todo el alcance es justo donde falla y donde compites con los bots. Apúntala a un objetivo concreto.
- **Enviar sin reproducir.** Su salida es una hipótesis, nunca un informe. Reproduce tú siempre.
- **Decidir el pivote.** Cuando un enfoque no da fruto, la IA insiste; el cambio de estrategia lo pones tú.
- **Copiar su redacción tal cual.** Los triagers detectan patrones LLM y sancionan, incluso informes válidos.

### Montaje práctico para tu plan

- **Semana 0:** instala Claude Code, conéctalo a tu proxy (Caido AI Skill o Burp con MCP/Montoya) y crea 2–3 skills propias (análisis JS, modo-proxy, validador de hallazgos). Rhynorater usa un "agente validador" solo para reducir falsos positivos ([YesWeHack](https://www.yeswehack.com/community/llms-bug-bounty-interview-rhynorater)).
- **Automatización en segundo plano:** monta un entorno donde un agente pruebe objetivos que tú predefinas sobre tus programas "casa", y tú validas lo que marque. Nunca lo dejes enviar solo.
- **Regla de oro:** la IA multiplica tu criterio, no lo sustituye. Sin tu base de CWES/CWEE, amplifica ruido; con ella, amplifica señal. Por eso tu formación es justo lo que hace rentable este stack.

## Rutina semanal sostenible

El objetivo es ingreso máximo sin jornadas de 12 horas. La clave no son más horas, sino proteger el bloque de deep-dive y no dejar los informes para "cuando salga".

&#91;embedded content: semana tipo · 4 días, \~32 h, bloque de informes el viernes\]

Principios que sostienen el ritmo:

- **Un bloque núcleo innegociable** (Mié–Jue) donde haces el trabajo que paga: entender y romper lógica. El resto se adapta a tu energía.
- **Los informes tienen horario fijo.** Si los dejas sueltos, se acumulan y pierdes el pago rápido y el acceso a privados.
- **Caja de tiempo por programa.** Dos semanas sin pista sólida = rotar; esto evita las sesiones interminables persiguiendo un bug que no existe.
- **Descansa de verdad.** El burnout es la causa número uno de abandono en bug bounty; con ingresos irregulares, tu cabeza es el activo que no puedes quemar.

## Informes que cobran

En 2026 el informe es tu producto, no un trámite. El triager distingue en segundos entre un cazador serio y "AI slop", y esa impresión decide tu pago, tu severidad y tu reputación.

**Anatomía de un informe que cobra bien:**

1. **Título claro:** tipo de vulnerabilidad + componente + impacto en una línea.
2. **Resumen de impacto primero:** qué puede hacer un atacante y a quién afecta, en lenguaje de negocio. Google exige que el informe demuestre la amenaza "en lenguaje claro" ([Infosecurity](https://infosecurity-magazine.com/news/google-launches-ai-bug-bounty)).
3. **Pasos de reproducción mínimos:** numerados, sin relleno, que cualquiera siga sin preguntarte.
4. **Prueba de concepto real:** petición exacta, captura o vídeo corto, cuenta y rol usados.
5. **Severidad justificada con CVSS** y el razonamiento, no solo el número.
6. **Remediación sugerida.** El valor se mueve hacia ayudar a arreglar, no solo a encontrar; ofrecerla te distingue ([The Register](https://www.theregister.com/security/2026/05/21/hackerone-takes-an-axe-to-its-bug-bounty-rewards/5244458)).

**Reglas de oro:**

- **Reproduce dos veces antes de enviar.** Un informe no reproducible es un N/A y daña tu Signal.
- **Un bug por informe.** No agrupes; cada hallazgo se evalúa y paga por separado.
- **No infles la severidad.** Pedir Critical por un Medium quema confianza con el triager, que es quien te invita a privados.
- **Si usas IA para redactar, reescribe con tu voz.** Los patrones LLM se detectan y penalizan, incluso en informes válidos.

**Duplicados y mediación:**

- El duplicado es inevitable; minimízalo siendo rápido en código nuevo y profundo donde el bot no llega.
- Si no estás de acuerdo con un cierre, argumenta con datos (impacto, diferencia con el supuesto duplicado), con calma y una sola vez. Las plataformas tienen mediación, pero el tono profesional pesa.
- Algunos programas dan margen de "primer reportero" a investigadores con buen historial frente a duplicados de baja calidad ([Decryption Digest](https://www.decryptiondigest.com/blog/bug-bounty-programs-ai-era-hackerone-bugcrowd-2026)): otra razón para cuidar tu reputación.

## Cómo crecer y ganar más

El dinero de verdad está en los programas privados e invitaciones; el 78 % de los programas empresariales son privados por invitación ([VoxBooster](https://voxbooster.com/blog/bug-bounty-statistics-2026)). Todo tu crecimiento consiste en ganarte esas invitaciones y, dentro de ellas, ser de los primeros.

**La escalera de crecimiento:**

1. **Reputación limpia (meses 1–3).** Objetivo: 5–10 informes válidos sin N/A y Signal por encima de 1.0 en tu plataforma principal. Aquí no buscas dinero, buscas la llave.
2. **Primeras invitaciones privadas (meses 4–6).** Llegan solas con buen Signal. Menos competencia = menos duplicados = más pago por hora.
3. **Programas privados estables (meses 6–12).** Conviértete en "casa" en 2–3 privados buenos. Es el escalón donde se estabiliza el ingreso.
4. **Eventos en vivo e invitaciones VIP (año 2+).** Los live hacking events de HackerOne, Intigriti y Bugcrowd concentran pagos altos y contactos; se entra por invitación según historial.

**Palancas concretas:**

- **Especialízate visiblemente.** Web + IA es tu diferenciador: hay menos cazadores compitiendo en superficies de IA. Que tu perfil y tus hallazgos cuenten esa historia.
- **Divulga lo que puedas.** Informes públicos, CVE y charlas construyen la reputación que miran Synack y los programas VIP.
- **Marca personal ligera.** Un blog con write-ups, GitHub con tus herramientas en Python/Go y presencia en la comunidad hispana de ciberseguridad. No es vanidad: es lo que convierte Signal en invitaciones y, llegado el caso, en ofertas de consultoría que suavizan los meses flojos.
- **Cumple requisitos de acceso pronto.** ID-check, HackerOne Clear y certificaciones verificables (tu CWES/CWEE cuenta) abren programas con menos competencia.
- **Colabora.** Hacer equipo con otro cazador en un privado reparte el bug pero multiplica superficie cubierta; está permitido si ambos tenéis acceso ([Intigriti KB](https://kb.intigriti.com/en/articles/5247238-community-code-of-conduct)).

**La métrica que importa no es cuántos bugs envías, sino tu ingreso por hora y tu ratio de validez.** Súbelos y el resto (invitaciones, eventos, ofertas) viene detrás.

## Límites legales, éticos y de plataforma

- **El alcance es la ley.** Prueba solo lo que el programa autoriza expresamente. Salirte del scope puede ser delito, no solo un informe rechazado.
- **Respeta Safe Harbor.** El 84 % de los programas empresariales ofrecen protección legal explícita ([VoxBooster](https://voxbooster.com/blog/bug-bounty-statistics-2026)); si un programa no la tiene clara, trátalo con cautela extra.
- **Datos personales, los mínimos.** Si tropiezas con datos reales de usuarios, no los extraigas ni los guardes; demuestra el acceso y para. Intigriti tiene reglas específicas sobre esto ([Intigriti KB](https://kb.intigriti.com/en/articles/5247238-community-code-of-conduct)).
- **Nada de DoS ni fuerza bruta destructiva** salvo permiso explícito; están prohibidos por defecto en casi todos los programas.
- **Divulgación coordinada.** No publiques un hallazgo sin aprobación de la plataforma y la empresa, sea cual sea su estado.
- **Las sanciones por mala praxis de IA son reales:** suspensiones de 30 días, verificación forzada de identidad y baneos permanentes por "farming" ([Bugcrowd](https://www.bugcrowd.com/blog/bugcrowd-policy-changes-to-address-ai-slop-submissions/)).
- **No mezcles tu trabajo de SOC con el bug bounty.** Usa equipos, cuentas y redes separadas, y revisa que tu contrato laboral no restrinja la actividad externa ni genere conflicto de interés.
- **Contexto regulatorio:** la Cyber Resilience Act de la UE entra en vigor el 11 de septiembre de 2026 y obliga a los fabricantes a mantener canales de reporte de vulnerabilidades ([ForkLog](https://forklog.com/en/the-end-of-bug-bounties/)). A medio plazo esto puede crear más VDP y programas con los que trabajar, aunque no te afecta directamente como cazador.

## Fiscalidad y administración en España

Esto es orientación general, no asesoramiento fiscal. Antes de empezar a cobrar en serio, habla con una gestoría o asesor: las bounties son una fuente de ingresos atípica y conviene hacerlo bien desde el primer euro.

**Lo esencial:**

- **Toda bounty es ingreso tributable desde el primer euro**, aunque llegue sin nómina ni retención ([Beancount](https://beancount.io/ca/blog/2026/09/13/bug-bounty-income-taxes-hackerone-bugcrowd-1099-hobby-business-guide)). Declararlo todo es lo que te protege.
- **Si es actividad recurrente y tu medio de vida, es actividad económica:** en la práctica implica darte de alta en Hacienda (modelo 036/037) y, con habitualidad, en el RETA (autónomos).
- **Cuota de autónomos 2026:** sistema de 15 tramos por rendimientos netos reales, desde unos 200 €/mes en el tramo más bajo ([Billeo](https://www.billeo.es/guias/cuanto-paga-autonomo-seguridad-social-2026)). Como nuevo autónomo puedes pedir la **tarifa plana**: 80 €/mes de cuota base (unos 87–88 € con el MEI) durante los 12 primeros meses ([Declarando](https://declarando.es/?p=43248)), y algunas comunidades (p. ej. Madrid) la complementan con "cuota cero".
- **Un consejo práctico sobre el alta:** dado tu plan de "un tiempo", valora el momento del alta. La tarifa plana solo se concede al darte de alta y es un activo que no querrás gastar en meses de prueba sin ingresos. Coméntalo con tu asesor.

**IVA e internacional:**

- Las plataformas (HackerOne, Intigriti, YesWeHack, Bugcrowd) están fuera de España, así que cobrarás de empresas de la UE o de fuera. El tratamiento de IVA de estos servicios es un punto que **debe revisar tu asesor**: suele implicar darse de alta en el ROI (modelo 036) y presentar el modelo 349 por operaciones intracomunitarias, pero depende de cada caso.
- **Guarda todo:** cada pago, su fecha, la plataforma y el programa. No hay "W-2" ni justificante automático; el registro lo llevas tú.

**Gastos deducibles habituales** (si eres autónomo): equipo, VPS y servidores de recon, licencias (Burp Suite Pro), formación (HTB Academy), cuota de la gestoría, parte proporcional de internet. Guarda las facturas.

**Idea clave:** separa desde el principio una cuenta para la actividad y reserva una parte de cada bounty para impuestos y cuota. Con ingresos irregulares, la sorpresa fiscal de fin de año es el error más caro.

## Hoja de ruta por fases

Un año dividido en cuatro fases. Los KPIs son metas, no promesas: en bug bounty el resultado es irregular por naturaleza.

| Fase | Meses | Foco | KPI de salida |
| --- | --- | --- | --- |
| 0. Preparación | Antes de empezar | Montar entorno, elegir 1 principal, hablar con gestoría, dejar listas tus herramientas en Python/Go | Entorno listo, decisión de alta tomada |
| 1. Reputación | 1–3 | 2–3 programas "casa" públicos bien elegidos; informes impecables | Signal > 1.0, 5–10 válidos, 0 N/A evitables |
| 2. Primeros privados | 4–6 | Aceptar invitaciones, primer High/Critical, afinar nicho web+IA | 2–3 invitaciones privadas, 1er High/Critical |
| 3. Estabilización | 6–9 | Convertir 2–3 privados en "casa", solicitar Synack/VIP si hay historial | Ingreso recurrente, ratio de validez > 50 % |
| 4. Escalado | 9–12 | Evento en vivo, marca personal, optimizar ingreso/hora | 1 evento o VIP, mejor €/hora que el mes 6 |

**Antes de dar el salto a tiempo completo**, ten estas tres cosas: colchón de 6 meses de gastos, un ingreso por hora que supere tu alternativa laboral, y al menos un programa privado "casa" rentable y estable. Si falta alguna, sigue compaginándolo con tu empleo un poco más: casi todos los que triunfan a tiempo completo tuvieron antes una larga etapa a tiempo parcial.

**Revisión mensual (30 min):** ingreso del mes, ratio de validez, Signal, horas reales trabajadas e ingreso por hora. Si el €/hora baja dos meses seguidos, cambia de programas o de enfoque antes de cambiar de horas.

## Fuentes

- [HackerOne — Hacker-Powered Security Report 2025](https://www.hackerone.com/report/hacker-powered-security)
- [Cyber Security News — HackerOne pagó 81 M$](https://cybersecuritynews.com/hackerone-bug-bounty-paid/amp/)
- [Red Hot Cyber — top 100 programas y programas con IA](https://www.redhotcyber.com/?p=135497)
- [The Register — HackerOne recorta recompensas del IBB](https://www.theregister.com/security/2026/05/21/hackerone-takes-an-axe-to-its-bug-bounty-rewards/5244458)
- [Dark Reading — HackerOne pausa el IBB](https://darkreading.com/application-security/ai-led-remediation-crisis-prompts-hackerone-pause-bug-bounties)
- [ForkLog — The end of bug bounties?](https://forklog.com/en/the-end-of-bug-bounties/)
- [Bugcrowd — cambios de política por "AI slop"](https://www.bugcrowd.com/blog/bugcrowd-policy-changes-to-address-ai-slop-submissions/)
- [AI Weekly — Apple pausa 180 días por informes de IA](https://aiweekly.co/alerts/apple-pauses-researchers-180-days-over-ai-generated-bug-reports)
- [GitHub — reestructuración del programa (SC Media)](https://www.scworld.com/brief/github-to-implement-two-tier-bug-bounty-program-amid-ai-generated-report-surge) · [Help Net Security](https://www.helpnetsecurity.com/2026/07/23/github-bug-bounty-program-changes/)
- [HackerOne Docs — Signal e informes de prueba](https://docs.hackerone.com/hackers/restricted-from-submissions.html)
- [Node.js — requisito de Signal](https://nodejs.org/en/blog/announcements/hackerone-signal-requirement)
- [TechRepublic — XBOW nº1 en el ranking de EE. UU.](https://www.techrepublic.com/article/news-ai-xbow-tops-hackerone-us-leaderboad/) · [LessWrong — escepticismo sobre XBOW](https://www.lesswrong.com/posts/qefrWyeiMvWEFRitN/be-skeptical-of-milestone-announcements-by-young-ai-startups)
- [Intigriti — código de conducta y uso de IA](https://kb.intigriti.com/en/articles/5247238-community-code-of-conduct) · [Intigriti — producto Bug Bounty](https://intigriti.com/product/bug-bounty)
- [Reddit — suspensión de 30 días por "posible IA"](https://reddit.sentinel-team.org/posts/1riql7d/snapshots/2026-03-03T02%3A34%3A55.31011Z)
- [YesWeHack — proveedor de la Comisión Europea](https://www.businesswire.com/news/home/20251001559762/it) · [SecurityWeek — ronda y clientes](https://www.securityweek.com/french-bug-bounty-platform-yeswehack-raises-28-million/amp/)
- [Synack — SRT y plataforma](https://www.synack.com/?p=20246) · [CREST Pathways](https://www.synack.com/press-releases/synack-expands-crest-partnership-with-new-certification-pathways-for-the-synack-red-team/) · [oferta SRT (requisitos)](https://resumegeni.com/jobs/synack-red-team-pentester-synacksrt-ca3bfd99fb)
- [The Register — Google AI VRP](https://www.theregister.com/2025/10/07/google_ai_bug_bounty/) · [Infosecurity — Google AI VRP](https://infosecurity-magazine.com/news/google-launches-ai-bug-bounty)
- [GIGAZINE — Anthropic abre su programa público](https://gigazine.net/gsc_news/en/20260508-anthropic-bug-bounty-program) · [SecurityWeek — OpenAI programa de abuso/seguridad de IA](https://www.securityweek.com/openai-launches-bug-bounty-program-for-abuse-and-safety-risks/amp/)
- [VoxBooster — estadísticas de bug bounty 2026](https://voxbooster.com/blog/bug-bounty-statistics-2026) · [Decryption Digest — HackerOne y Bugcrowd en la era IA](https://www.decryptiondigest.com/blog/bug-bounty-programs-ai-era-hackerone-bugcrowd-2026) · [TechRound — GitHub y el modelo de reputación](https://techround.co.uk/cybersecurity/how-ai-slop-is-forcing-github-to-close-its-doors/)
- [Beancount — fiscalidad de bug bounties](https://beancount.io/ca/blog/2026/09/13/bug-bounty-income-taxes-hackerone-bugcrowd-1099-hobby-business-guide)
- [Billeo — cuota de autónomos 2026](https://www.billeo.es/guias/cuanto-paga-autonomo-seguridad-social-2026) · [Declarando — tarifa plana 2026](https://declarando.es/?p=43248)
