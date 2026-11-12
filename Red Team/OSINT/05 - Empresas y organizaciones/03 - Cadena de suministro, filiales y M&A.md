---
tags:
  - OSINT/Corporativo
  - Pentesting/Enumeracion
Descripción: "La superficie de ataque que el cliente olvida: filiales, adquisiciones recientes y proveedores con acceso, y cómo mapearlos desde fuentes públicas"
Fecha de actualización: 2026-11-08
Nota previa: "[[02 - Tech stack y activos digitales]]"
Nota siguiente:
Area: "[[OSINT corporativo.base|OSINT corporativo]]"
---
---

El perímetro de una organización no acaba en sus sistemas: incluye las empresas que **compró**, las **filiales** que gestiona con otro criterio y los **proveedores** con acceso a su red. <mark style="background: #ADCCFFA6;">Es la superficie que el cliente más olvida y, a menudo, la más débil</mark>.

# Filiales: superficie heredada

Cada filial descubierta en los [[01 - Registros mercantiles y financieros|registros]] es una organización con su propio dominio, sus empleados y su seguridad — normalmente **peor** que la de la matriz:

- **Dominios propios**: añádelos al alcance de activos digitales. Una filial pequeña con una web sin mantener es un punto de entrada a la red del grupo.
- **Confianza interna**: las filiales suelen tener conectividad o relaciones de confianza con la matriz (VPN, dominios AD con *trust*, SSO compartido). Comprometer la filial puede dar salto lateral al grupo.
- **Criterio de seguridad dispar**: lo que la matriz parchea, la filial puede ignorar.

# Adquisiciones (M&A): el caos como oportunidad

<mark style="background: #FFB86CA6;">Una adquisición reciente es una ventana de oportunidad</mark>. Durante la integración:

- La empresa adquirida llega con su **infraestructura legacy**, sus credenciales y sus vulnerabilidades, que tardan meses o años en alinearse con las políticas de la matriz.
- Aparecen **dominios y marcas nuevas** en el alcance que nadie ha revisado con criterio de seguridad del grupo.
- Los **empleados** recién incorporados son objetivos de phishing ideales: aún no conocen los procesos internos ni a quién verificar ("soy de IT de la nueva matriz, necesito que...").

Las adquisiciones se anuncian públicamente: notas de prensa, [[01 - Registros mercantiles y financieros|SEC EDGAR / 8-K]], Crunchbase, LinkedIn. <mark style="background: #8000E1A6;">Cada "ACME adquiere Beta Corp" es, para ti, "añade los dominios y empleados de Beta Corp al alcance"</mark> (si están autorizados).

# Proveedores y terceros: la cadena de suministro

El acceso de terceros es un vector lateral clásico. Un proveedor con VPN, un MSP que administra los sistemas, una integración SaaS con permisos amplios — comprometer a uno de ellos puede dar acceso al objetivo sin tocar su perímetro. Cómo mapear la cadena desde fuentes públicas:

- **Casos de éxito y notas de prensa**: "ACME elige [Proveedor X] para su CRM" te dice una integración y un posible pretexto.
- **Ofertas de empleo**: "experiencia con [producto de proveedor]" confirma qué terceros están dentro.
- **Páginas de integraciones / partners**: la propia web lista a sus socios tecnológicos.
- **Registros DNS y correo**: un subdominio `CNAME` a un SaaS, o un `include:` en el [[02 - Tech stack y activos digitales#La joya: spoofability por SPF y DMARC|SPF]] apuntando a un proveedor de email marketing, delata qué servicios externos usan.

<mark style="background: #FF5582A6;">Un proveedor con acceso y peor seguridad es, a menudo, la ruta más corta al objetivo</mark>. Es el principio del ataque a la cadena de suministro, trasladado a la fase de recon.

# El mapa ampliado

El resultado de esta nota es un alcance **real**, no el nominal:

```text
ACME Technologies S.L.  (matriz)
├── Beta Corp           (adquirida 2025 → legacy, dominios nuevos)
├── ACME France SARL    (filial → trust con la matriz)
├── Proveedor CRM        (acceso SaaS, include: en el SPF)
└── MSP de IT            (acceso VPN administrativo)
```

> [!warning]+ Alcance y autorización, otra vez
> Ampliar la superficie es inteligencia, pero **atacar** una filial, una adquirida o un proveedor exige que estén **explícitamente en el alcance**. Un proveedor es una empresa distinta: atacarlo sin autorización es ilegal aunque dé acceso al cliente. Repórtalo como superficie y confirma el alcance antes de tocar nada ([[04 - Marco legal y ético del OSINT|legal]]).

> [!info]+ Fuentes
> M&A y filiales desde registros y prensa: [[01 - Registros mercantiles y financieros|registros mercantiles]], SEC EDGAR (8-K), Crunchbase. Cadena de suministro desde SPF/DNS: ver [[02 - Tech stack y activos digitales]]. Riesgo de terceros: marco de *supply chain* (NIST SP 800-161).
