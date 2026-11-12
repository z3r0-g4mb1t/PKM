---
tags:
  - OSINT/OPSEC
  - Pentesting/Enumeracion
  - Tipo/Arsenal
Descripción: "Montar el entorno de investigación: VM dedicada con snapshots, compartimentación del navegador y la capa de red (VPN, proxy residencial, Tor)"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Sock puppets - crear y mantener identidades de investigación]]"
Nota siguiente: "[[03 - Cómo no quemar la operación - fugas y errores comunes]]"
Area: "[[OPSEC OSINT.base|OPSEC OSINT]]"
---
---

Las capas 2 y 3 del [[00 - El OPSEC del investigador OSINT|modelo]] —máquina aislada y red anonimizada— son infraestructura. Esta nota la monta.

# La máquina: VM dedicada con snapshots

<mark style="background: #ADCCFFA6;">Investiga siempre desde una VM separada de tu equipo real</mark>. Dos motivos: aísla el malware (un enlace, un *tracking pixel* o un JS malicioso solo afecta a la VM, que revierte), y separa tu actividad de investigación de tu vida personal.

Flujo de trabajo con **snapshots** (VirtualBox o VMware):

```shell-session
# Crear snapshot limpio tras configurar la VM
$ VBoxManage snapshot "OSINT-VM" take "limpio" --description "estado base"
# Tras una operación, revertir al estado limpio (borra cookies, historial, rastros)
$ VBoxManage snapshot "OSINT-VM" restore "limpio"
```

<mark style="background: #8000E1A6;">La regla: una operación, un snapshot limpio al empezar y revertir al acabar</mark>. Así ninguna investigación contamina la siguiente.

Distribuciones listas para OSINT (traen el *toolset* preinstalado y perfiles de navegador endurecidos):

- **Trace Labs OSINT VM** — basada en Kali, orientada a los CTF de *Search Party*; set de herramientas OSINT preconfigurado. La release 2026.10 añadió repositorio `apt` propio (`apt.tracelabs.org`) y plantilla para QubesOS *(dato de una sola fuente, verificar)*.
- **CSI Linux** — distribución de forense digital + OSINT.
- **Tsurugi Linux** — forense/OSINT, muy completa.
- **Tails** — amnésica (no deja rastro en disco) y enruta todo por Tor; para objetivos sensibles.

No necesitas una distro especial: una **Ubuntu/Kali limpia** con tus herramientas instaladas y un snapshot vale perfectamente. Lo importante es que sea **dedicada**.

> [!warning]+ La VM aísla malware, NO esconde tu IP
> Una VM separa tu investigación de tu equipo, pero **sale a Internet por tu misma línea**: tu IP real queda en los logs del sitio que visitas. La anonimización de red es una **capa aparte** (la de abajo). Confundir las dos es el error de infra más común.

# El navegador: compartimentar sesiones

Las cookies y sesiones son lo que vincula identidades. <mark style="background: #FFB86CA6;">Si navegas con el puppet y con tu cuenta real en el mismo navegador, las plataformas pueden correlacionarlos</mark>. Opciones, de menos a más estricta:

- **Perfiles separados** (Firefox/Chrome): un perfil por persona/investigación. Cada perfil tiene sus cookies, historial y extensiones aislados.
- **Firefox Multi-Account Containers**: aísla cookies **por pestaña** dentro de un mismo Firefox — cómodo para separar varias identidades sin abrir perfiles distintos.
- **Navegadores distintos**: el real en un navegador, cada puppet en otro. Simple y robusto.
- **Mullvad Browser** (de los creadores de Tor, sin Tor): endurecido contra *fingerprinting*; hace que tu navegador se parezca al de todos los demás.

Imprescindible en cualquier caso: **desactivar WebRTC** (filtra tu IP real aunque uses VPN) y revisar extensiones que identifiquen.

# La red: VPN, proxy residencial o Tor

Cada capa de red encaja en un uso. Elegir mal te delata o te bloquea:

| Capa | Qué parece | Cuándo usarla | Cuidado |
| - | - | - | - |
| **VPN** (reputada) | IP de datacenter | Navegación general, ocultar tu IP real | Las plataformas **marcan** rangos de VPN → no para registrar cuentas |
| **Proxy residencial / móvil** | IP de un usuario doméstico | Trabajo con cuentas, registro, *scraping* a escala | De pago; elige proveedor con IPs éticas |
| **Tor** | Nodo de salida Tor (conocido) | Dark web, máximo anonimato | Lento; muchos sitios **bloquean** nodos de salida |

<mark style="background: #FF5582A6;">Regla práctica: VPN para OPSEC general, proxy **residencial** para registrar/operar cuentas (no te marcan), Tor para dark web y lo más sensible</mark>. Nunca registres una cuenta por VPN/Tor (ver [[01 - Sock puppets - crear y mantener identidades de investigación#3. Registrar sin que te marquen|registro sin flagging]]).

# Atribución gestionada "de empresa"

Para equipos, hay servicios comerciales que montan todo esto en la nube y hacen que tu tráfico salga desde su infraestructura, no la tuya:

- **Authentic8 Silo** — navegador en la nube; todo se ejecuta y sale desde su infra, nada toca tu equipo.
- **Ntrepid Nfusion** — VM desechable completa en la nube (no solo navegador).
- **Menlo Security** — aislamiento de navegador.

Son caros y orientados a CTI/empresa, pero resuelven la atribución gestionada "llave en mano". Para un pentester individual, la VM + navegador compartimentado + VPN/proxy cubre el 90 % a coste cero.

> [!info]+ Fuentes
> VMs de OSINT y el principio de aislamiento: [SpecialEurasia, *OSINT: Operational Security, Virtualisation and Trace Labs*](https://www.specialeurasia.com/2026/04/16/osint-virtualisation-tracelabs/); release 2026.10 de Trace Labs vía [Max Intel](https://maxintel.org/opsec-osint-guide-2026.html) *(sin confirmar en fuente oficial)*. Navegadores para OSINT: [osintme.com, *Web browsers for OSINT investigators*](https://osintme.com/index.php/2026/09/30/web-browsers-for-osint-investigators/). Soluciones comerciales (Silo, Nfusion, Menlo): review 2026 de cloud browsers.
