---
tags:
  - OSINT/Corporativo
  - Pentesting/Enumeracion
  - Tipo/Introduccion
Descripción: "La organización como superficie de ataque: entidad legal, filiales, activos digitales, personas, tecnología y proveedores, y cómo se encadenan"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Registros mercantiles y financieros]]"
Area: "[[OSINT corporativo.base|OSINT corporativo]]"
---
---

Cuando el objetivo es una empresa, el OSINT corporativo mapea **todo lo que la compone** para definir la superficie de ataque real — que casi siempre es más grande de lo que el cliente cree. <mark style="background: #ADCCFFA6;">Una organización no es un dominio: es un conjunto de entidades legales, filiales, activos digitales, personas, tecnología y proveedores, cada uno con su propia exposición</mark>.

# La organización como superficie de ataque

Las piezas que hay que mapear, y por qué cada una importa:

| Pieza | Qué aporta al ataque | Dónde se desarrolla |
| - | - | - |
| **Entidad legal y filiales** | Amplía el alcance: filiales y adquisiciones suelen tener seguridad más débil | [[01 - Registros mercantiles y financieros]] |
| **Activos digitales** | Dominios, IPs, rangos, certificados — la superficie técnica | [[OSINT de infraestructura.base\|infraestructura]] |
| **Personas** | Empleados → usuarios, correos, phishing | [[03 - Enumeración de personal\|personal]] + [[01 - LinkedIn - la mina del red teamer\|LinkedIn]] |
| **Tech stack** | Qué software corre → qué CVEs y defensas | [[02 - Tech stack y activos digitales]] |
| **Proveedores y terceros** | La cadena de suministro como vía lateral | [[03 - Cadena de suministro, filiales y M&A]] |

<mark style="background: #8000E1A6;">El error del junior es quedarse en el dominio principal</mark>. El valor está en lo que el cliente olvidó que tiene: una filial adquirida hace dos años con su propio dominio sin parchear, un proveedor con acceso VPN, un entorno de *staging* de una marca secundaria.

# El punto de partida: resolver la entidad

Antes de buscar activos, fija **qué es** exactamente el objetivo:

- **Nombre legal exacto** y jurisdicción (puede diferir del nombre comercial — "ACME" puede ser "ACME Technologies S.L.").
- **Grupo empresarial**: matriz, filiales, marcas. Esto define el alcance real.
- **Identificadores**: CIF/NIF (España), número de registro, `LEI` (Legal Entity Identifier global).

<mark style="background: #FFB86CA6;">Resolver bien la entidad al principio evita perder horas sobre el objetivo equivocado</mark> y descubre filiales que multiplican la superficie.

# El flujo corporativo

1. **Identidad legal** → registros mercantiles: filiales, administradores, finanzas → [[01 - Registros mercantiles y financieros]].
2. **Activos digitales** → dominios, rangos IP, ASN (puente a [[OSINT de infraestructura.base|infraestructura]]).
3. **Tech stack** → qué tecnología y qué correo usa → [[02 - Tech stack y activos digitales]].
4. **Personas** → [[01 - LinkedIn - la mina del red teamer|LinkedIn]] y [[03 - Enumeración de personal|personal]].
5. **Terceros** → proveedores, integraciones, adquisiciones → [[03 - Cadena de suministro, filiales y M&A]].

> [!important]+ El OSINT corporativo define el alcance real
> En un engagement, este mapeo es lo que convierte un "pentest de acme.com" en una imagen fiel de la exposición: todos los dominios, todas las filiales, toda la tecnología. Reportar activos que el cliente **no sabía que tenía expuestos** es, a menudo, el hallazgo de más valor del informe. Eso sí: actuar solo sobre lo que esté **en alcance** ([[04 - Marco legal y ético del OSINT|legal]]).

> [!info]+ Fuentes
> Marco de OSINT corporativo/financiero: [osintbench, *Corporate and Financial OSINT*](https://osintbench.com/guides/financial-osint/); [Max Intel, *Business OSINT*](https://maxintel.org/business.html).
