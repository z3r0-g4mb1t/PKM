# Checklist ajustes ASUS XG27AQWMG

Nov 14, 2026 · @z3r0

Configuración final para el XG27AQWMG con RTX 5070 Ti y Windows 11: SDR con gama amplia para trabajar y jugar, HDR solo para contenido HDR y Antiparpadeo OLED desactivado. Los nombres de las opciones pueden variar un poco en el menú en español.

## 1. Conexión y resolución

DisplayPort (como ahora) a 2560×1440 y 280 Hz.

- [ ] Cable DisplayPort conectado (HDMI 2.1 también vale y evita la compresión DSC)
- [x] Windows > Sistema > Pantalla > Pantalla avanzada: 280 Hz
- [x] La barra de información del menú del monitor muestra 2560x1440 280Hz

## 2. Panel de control de NVIDIA

G-SYNC activo en todo, límite de 270 FPS y frecuencia fija solo para las apps de escritorio.

- [ ] Cambiar resolución: 2560×1440 (lista PC), 280 Hz
- [ ] Color: «Usar configuración de color de NVIDIA» → RGB · 10 bpc · rango dinámico Completo
- [ ] Color del escritorio: todo por defecto, vibración digital 50 %
- [ ] Configurar G-SYNC: activado para pantalla completa y modo ventana + «Habilitar la configuración para el modelo de pantalla seleccionado»
- [ ] Configuración 3D global > Sincronización vertical: Activada
- [ ] Configuración 3D global > Velocidad de fotogramas máxima: 270 FPS
- [ ] Configuración 3D global > Modo de baja latencia: Activado, no Ultra, porque ya hay límite de 270 FPS (en juegos con Reflex, activa Reflex en el juego). Triple búfer: Desactivado (solo afecta a OpenGL)
- [ ] Configuración de programa > navegador (chrome.exe, msedge.exe o firefox.exe), Discord y app de Claude → Tecnología del monitor: Frecuencia de actualización fija
- [ ] Escalado de imagen (NIS) y filtros de nitidez: desactivados
- [ ] Indicador G-SYNC (menú Pantalla): desactivado, solo para diagnosticar
- [ ] Opcional: DSR – Factores → DL 1.78x para juegos ligeros

## 3. Windows 11

Escritorio en SDR, escala al 100 % y texto ajustado para leer en modo oscuro.

- [x] Pantalla > Gráficos > Configuración predeterminada: Frecuencia de actualización variable y Optimizaciones para juegos en ventana activadas
- [x] Escala: 100 % (agranda el texto con la fuente del IDE o el zoom del navegador)
- [x] Texto: compara «Ajustar texto ClearType» con Better ClearType Tuner en escala de grises y quédate con el que menos canse (con modo oscuro, probablemente escala de grises)
- [x] Windows Terminal: `antialiasingMode` en `grayscale` (valor por defecto)
- [x] «Administrar color automáticamente para aplicaciones»: desactivado
- [x] Perfil de color: el del BenQ solo en el BenQ; sin perfil ICC de ASUS en el XG27AQWMG
- [x] HDR de Windows: desactivado por defecto (se activa con Win + Alt + B)
- [x] Auto HDR: desactivado (usa RTX HDR en su lugar)
- [x] «Calibración de HDR de Windows» instalada desde Microsoft Store
- [x] Salida de sonido: comprueba que no se haya cambiado al monitor
- [x] Energía: apagar pantalla a los 5–10 min; barra de tareas y fondo oscuros

## 4. Monitor: perfil Usuario (trabajo, lectura y uso general)

Modo Usuario con gama amplia y Brillo uniforme activado: tu perfil para todo.

- [x] Juego > GameVisual: Modo Usuario
- [x] Juego > Frecuencia de actualización variable: ON
- [x] Juego > ELMB: OFF (queda bloqueado con el VRR activado; es normal)
- [x] Juego > Shadow Boost: OFF (también Dynamic Shadow Boost y Dynamic Crosshair)
- [x] Imagen > Brillo: 30–40 (unos 110–140 nits en habitación oscura)
- [x] Imagen > Brillo uniforme: ON
- [ ] Imagen > Contraste: por defecto (80)
- [ ] Imagen > VividPixel: por defecto, sin subir
- [ ] Imagen > Antiparpadeo OLED: Desactivado
- [ ] Imagen > Filtro de luz azul: OFF
- [ ] Color > Espacio de color: Gama amplia (Wide Gamut)
- [ ] Color > Temperatura de color: 6500K
- [ ] Color > Gamma: 2.2
- [ ] Color > Saturación: por defecto
- [ ] DisplayWidget Center > AI Visual: desactivado

## 5. Monitor: perfil de juego (opcional, modo Carreras)

Un segundo perfil con más pegada para jugar; si te basta con Usuario, sáltate esta sección.

- [ ] Juego > GameVisual: Modo Carreras, con los mismos ajustes de Color que Usuario
- [ ] Imagen > Brillo: 50–70
- [ ] Imagen > Brillo uniforme: OFF (picos más brillantes)
- [ ] Juego > Shadow Boost: OFF (nivel mínimo solo si en un competitivo no ves en las sombras)
- [ ] MyFavorite > Acceso directo: GameVisual, para cambiar de perfil con el joystick
- [ ] Opcional: DisplayWidget Center vinculando el perfil Carreras a cada juego

## 6. HDR (solo juegos y vídeo con HDR)

Actívalo con Win + Alt + B antes de abrir el juego o el vídeo, y desactívalo al terminar.

- [ ] Imagen > Configuración HDR: DisplayHDR 500 True Black (por defecto en habitación oscura); HDR Gaming si un juego se ve apagado
- [ ] HDR regulable: desactivado
- [ ] La primera vez en HDR: repasa el menú del monitor, algunos ajustes van aparte
- [ ] Calibración de HDR de Windows pasada con el modo HDR elegido
- [ ] En cada juego, brillo máximo: unos 800 nits (True Black 500) o unos 1.500 nits (HDR Gaming)
- [ ] Juegos SDR para un jugador: RTX HDR (app NVIDIA), nunca a la vez que Auto HDR
- [ ] Opcional: RTX Video HDR para vídeo SDR en el navegador

## 7. OLED Care y cuidado del panel

Deja la protección activada: cuida el panel y ASUS advierte que desactivarla puede afectar a la garantía de quemado (3 años).

- [ ] OLED Care > Movimiento de pantalla: Intenso
- [ ] OLED Care > Protector de pantalla: ON (OFF solo si te atenúa la pantalla mientras lees)
- [ ] OLED Care > Brillo automático del logo: ON (OFF solo si atenúa partes fijas del IDE)
- [ ] OLED Care > Sensor de proximidad Neo: ON, con el sensor limpio y la distancia ajustada a tu postura
- [ ] Limpieza de píxeles: deja que se ejecute; apaga con el botón del monitor, nunca desde la regleta
- [ ] Efecto de iluminación: luz del logo frontal apagada; Aura RGB trasero a tu gusto
- [ ] Opcional: tira LED tenue a 6500K detrás del monitor para leer a oscuras

## 8. Juegos

Resolución nativa siempre y FPS estables; con tu equipo, el LoL debería superar con holgura los 270 FPS.

- [ ] LoL > Vídeo > Límite de fotogramas: Sin límite (el tope de 270 lo pone NVIDIA)
- [ ] LoL > Esperar a la sincronización vertical: desactivado
- [ ] Perfil de League of Legends.exe en NVIDIA: sin límite de FPS propio
- [ ] Todos los juegos: resolución nativa 2560×1440; si falta rendimiento, DLSS Calidad (DLAA si sobra GPU)
- [ ] Juegos con NVIDIA Reflex: Reflex activado
- [ ] Si un juego parpadea en escenas oscuras: límite de FPS estable en su perfil de NVIDIA; si persiste en un juego para un jugador, «Frecuencia de actualización fija» solo para ese juego

## 9. Mantenimiento y trucos

Firmware al día (MCM107) y ajustes guardados para recuperarlos tras cualquier reinicio.

- [ ] Firmware: comprobar en Configuración del sistema > Información (actualmente MCM107)
- [ ] Guardar ajustes: MyFavorite > Ajuste personalizado (o exportarlos desde DisplayWidget Center)
- [ ] Tras tocar el Antiparpadeo o restablecer el monitor: comprobar que Windows sigue a 280 Hz
- [ ] Configuración del sistema > Configuración OSD > DDC/CI: activado (lo usa DisplayWidget Center)
- [ ] Bloqueo de teclas: se quita manteniendo el joystick hacia abajo más de 5 segundos

## Fuentes

- [TFTCentral – Asus ROG Strix XG27AQWMG](https://tftcentral.co.uk/reviews/asus-rog-strix-xg27aqwmg)
- [KitGuru – ASUS ROG Strix XG27AQWMG Review](https://www.kitguru.net/peripherals/monitors/dominic-moass/asus-rog-strix-xg27aqwmg-review-4th-gen-tandem-oled/all/1)
- [RTINGS – ASUS ROG Strix OLED XG27AQWMG](https://www.rtings.com/monitor/reviews/asus/rog-strix-oled-xg27aqwmg)
- [Manual ASUS ROG Strix OLED XG27AQWMG-R](https://cdn.iceshop.nl/objects/mmo_150196343_1787327638_6958_4008181.pdf)
- [ASUS – Firmware XG27AQWMG](https://www.asus.com/supportonly/xg27aqwmg/helpdesk_bios/)
- [TFTCentral – Tecnologías OLED de Asus 2025](https://tftcentral.co.uk/articles/helping-avoid-oled-burn-in-and-flicker-exploring-the-latest-asus-oled-technologies-for-2025)
- [ROG – How to use ASUS OLED Care](https://rog.asus.com/ch-en/articles/gaming-monitors/how-to-use-asus-oled-care-to-protect-your-rog-oled-monitor/)
