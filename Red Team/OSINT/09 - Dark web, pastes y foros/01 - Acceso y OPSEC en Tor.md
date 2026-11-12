---
tags:
  - OSINT/Dark-Web
  - Pentesting/Enumeracion
Descripción: "Entrar en Tor con seguridad: Tor Browser, Tails/Whonix, verificar direcciones .onion contra clones y las reglas de conducta que no se rompen"
Fecha de actualización: 2026-11-08
Nota previa: "[[00 - Dark web y fuentes cerradas - qué hay y para qué]]"
Nota siguiente: "[[02 - Buscar en la dark web - motores y directorios]]"
Area: "[[Dark web OSINT.base|Dark web OSINT]]"
---
---

Cuando una fuente solo vive en `.onion` y las [[00 - Dark web y fuentes cerradas - qué hay y para qué|pasarelas clearnet]] no la cubren, toca entrar en Tor. Hazlo con OPSEC estricto: aquí los errores se pagan más caro que en la web normal.

# El acceso: de básico a serio

- <mark style="background: #ADCCFFA6;">**Tor Browser** es la línea base</mark>: enruta el tráfico por tres saltos cifrados, ocultando tu IP real al servicio oculto. Suficiente para consultas puntuales de baja sensibilidad.
- **Trabajo serio** (objetivos capaces, fuentes delicadas): aíslalo del sistema real:
  - **Tails**: SO amnésico en USB que enruta **todo** por Tor y no deja rastro en disco.
  - **Whonix**: dos VMs (gateway + workstation) que fuerzan **todo** el tráfico de la workstation por Tor, de modo que ni una fuga de aplicación revela tu IP.
- Siempre desde la [[02 - Infraestructura de investigación - VM, red y navegador|máquina de investigación]], no tu equipo real.

<mark style="background: #FFB86CA6;">Tor te anonimiza la red, pero no tu **conducta**</mark>: el resto del OPSEC sigue aplicando.

# Verificar las direcciones .onion

<mark style="background: #FF5582A6;">Las direcciones `.onion` son cadenas largas y aleatorias (`v3`, 56 caracteres), imposibles de recordar — terreno perfecto para el phishing de clones</mark>. Es endémico: sitios falsos que imitan mercados y buscadores para robar o engañar (Ahmia mismo sufrió clones fraudulentos en 2015). Antes de abrir un `.onion`:

- Consíguela de una **fuente verificada**: **dark.fail** mantiene un directorio de direcciones comprobadas con firmas PGP.
- Desconfía de direcciones sacadas de un foro o un buscador sin verificar. Una letra distinta es otro sitio.

# Las reglas que no se rompen

> [!danger]+ Conducta en Tor
> - **No descargues ni ejecutes** nada. Un fichero de un `.onion` es malware hasta que se demuestre lo contrario — y lo abres en una VM desechable, nunca en el host.
> - **JavaScript al mínimo**: pon Tor Browser en el nivel de seguridad **"Safest"** (desactiva JS), que cierra la mayoría de exploits de desanonimización.
> - **Nunca** inicies sesión en cuentas personales, ni mezcles la identidad Tor con la clearnet. Una sola sesión real te desanonimiza.
> - **No interactúes**: no escribas en foros, no contactes vendedores, no "preguntes". Observa.
> - **No accedas a contenido ilegal**: parte del contenido es delito con solo poseerlo/visualizarlo. Si una búsqueda te lleva ahí, retrocede. Esto no es OSINT, es un delito.

# OPSEC de identidad

Igual que en el resto del OSINT: [[01 - Sock puppets - crear y mantener identidades de investigación|sock puppet]] si una fuente exige registro (muchos foros lo piden), nunca tu identidad real, y el dispositivo/identidad Tor compartimentado del resto. <mark style="background: #8000E1A6;">La regla de oro: en Tor eres un observador anónimo y desechable, no un participante</mark>.

> [!warning]+ Entrar en Tor no es gratis en riesgo
> En algunos entornos corporativos o jurisdicciones, el mero uso de Tor levanta banderas. Y la navegación por ciertos servicios puede tener consecuencias legales. Entra con un objetivo concreto (una fuente que necesitas ver), haz lo mínimo, y sal. Si el dato está en [[00 - Dark web y fuentes cerradas - qué hay y para qué|clearnet vía pasarela]], no entres en Tor.

> [!info]+ Fuentes
> Herramientas de anonimato: [Tor Project](https://www.torproject.org), [Tails](https://tails.net), [Whonix](https://www.whonix.org). Verificación de direcciones: [dark.fail](https://dark.fail). Clones fraudulentos de Ahmia (2015): [Ahmia (Wikipedia)](https://en.wikipedia.org/wiki/Ahmia).
