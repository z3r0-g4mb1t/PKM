---
tags:
  - OSINT/Contra-OSINT
  - Tipo/Deteccion
Descripción: "El eje defensivo del OSINT: por qué la recolección pasiva es casi indetectable y por qué la defensa es reducir huella y sembrar trampas, no detectar"
Fecha de actualización: 2026-11-08
Nota previa:
Nota siguiente: "[[01 - Reducción de huella personal y corporativa]]"
Area: "[[Contra-OSINT.base|Contra-OSINT]]"
---
---

El eje 2 del vault —detección y evasión— tiene en OSINT una forma peculiar. <mark style="background: #FF5582A6;">La recolección OSINT pasiva es **casi indetectable** para el objetivo: las consultas van a terceros, no a él</mark>. Eso invierte el problema defensivo: no puedes "detectar al que te hace OSINT", así que la defensa es **no tener nada que encontrar** y **sembrar trampas** para los pocos momentos en que el atacante sí deja rastro.

# Por qué no lo detectas

Cuando alguien consulta tu dominio en [[01 - Motores de búsqueda de dispositivos - Shodan, Censys y FOFA|Shodan]], busca tus empleados en [[01 - LinkedIn - la mina del red teamer|LinkedIn]] o tus credenciales en [[01 - Buscadores de brechas - HIBP, Dehashed e IntelX|una brecha]], **no toca tu infraestructura**. No hay un log tuyo que registre nada. <mark style="background: #8000E1A6;">Solo ves algo cuando el atacante pasa de recolectar a **actuar**</mark> — y para entonces el OSINT ya cumplió su función.

Consecuencia: la defensa contra-OSINT es **preventiva y proactiva**, no reactiva. Tres frentes:

| Frente | Qué hace | Nota |
| - | - | - |
| **Reducir huella** | Que no haya nada expuesto que recolectar | [[01 - Reducción de huella personal y corporativa]] |
| **Deception** | Trampas que delatan al atacante cuando toca el cebo | [[02 - Detección y deception contra OSINT]] |
| **Monitorización** | Vigilar tu propia exposición (dorkéate, CT logs, brechas) | [[02 - Detección y deception contra OSINT]] + [[03 - Monitorización y el ángulo CTI]] |

# El valor para el red teamer

<mark style="background: #ADCCFFA6;">Esta nota no es solo defensa: es inteligencia ofensiva</mark>. Entender qué deja rastro y qué no te dice:

- **Qué puedes hacer con impunidad** (recon pasivo invisible) y qué te delata (los [[05 - Verificación, fiabilidad de fuentes y sesgos#Datos envenenados: cuando la fuente te tiende una trampa|canary tokens]], los certificados que emites, el login que pruebas).
- **Qué nivel de higiene tiene el objetivo**: una empresa que no ha hecho nada de [[01 - Reducción de huella personal y corporativa|reducción de huella]] (la mayoría) seguirá siendo oro para el OSINT.
- **El reverso de tu propio [[03 - Cómo no quemar la operación - fugas y errores comunes|OPSEC]]**: lo que un objetivo hace para detectar al que le investiga es lo que te detectaría a ti.

# El puente al Blue Team

<mark style="background: #FFB86CA6;">La reducción de huella y la deception son capacidades defensivas estándar</mark> (las hace un equipo de seguridad o un servicio de *Digital Risk Protection*). El red teamer que las conoce redacta mejores hallazgos ("tu DMARC es `p=none`, tu huella de empleados es excesiva, no monitorizas tus CT logs") y entiende el juego desde los dos lados. Es el punto donde el OSINT ofensivo se toca con la defensa.

> [!info]+ Fuentes
> La asimetría detección/prevención del OSINT es consenso de la comunidad (práctica de *Digital Risk Protection*). Desarrollo en las dos notas siguientes.
