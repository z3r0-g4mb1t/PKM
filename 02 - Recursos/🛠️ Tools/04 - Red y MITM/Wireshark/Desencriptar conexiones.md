---
tags:
  - Windows
  - Linux
  - Escaneo/Redes
Descripción: "Wireshark descifra el tráfico capturado si le das las claves: TLS por key log, WPA/WPA2 con PSK y handshake, Kerberos y RDP"
Fecha de actualización: 2026-11-12
Nota previa: "[[Uso avanzado de Wireshark]]"
Nota siguiente:
Area: "[[Wireshark.base|Wireshark]]"
---
---

Una captura cifrada no es un muro. <mark style="background: #ADCCFFA6;">Wireshark descifra el tráfico si le proporcionas el material de claves adecuado</mark>, y lo hace de forma **retroactiva**: cargas la clave y re-disecciona el `pcap` ya capturado. Los dos casos que más salen en un engagement son **TLS** (HTTPS, RDP moderno, casi todo) y **WPA/WPA2** (Wi-Fi).

# TLS: manda el *key log*, no la clave privada

Hoy casi todo el TLS negocia intercambio de claves **efímero** (`ECDHE`) → *forward secrecy*. <mark style="background: #FFB86CA6;">Con forward secrecy la clave privada RSA del servidor NO sirve para descifrar</mark>: cada sesión deriva un secreto que no está en esa clave. Esto incluye **todo TLS 1.3** y el 1.2 moderno.

La vía universal es el **(pre-)master-secret key log**. Muchos clientes —Chrome, Firefox, `curl`, librerías con OpenSSL/BoringSSL/NSS— escriben los secretos de sesión a un fichero si defines la variable `SSLKEYLOGFILE`:

```shell-session
$ export SSLKEYLOGFILE=~/keys.log
$ google-chrome    # o curl, firefox…
```

En Wireshark: *Edit ▸ Preferences ▸ Protocols ▸ TLS ▸ (Pre)-Master-Secret log filename* → apuntas a `keys.log`. Al instante, las tramas `Application Data` pasan a `HTTP`/`HTTP2` legibles, con *Follow ▸ HTTP Stream*.

El caso **legacy** (TLS ≤1.2 con intercambio **RSA**, sin PFS) sí admite la **clave privada del servidor** (*TLS ▸ RSA keys list*). Es cada vez más raro: si el `ClientHello` negoció `ECDHE`, este método no aplica.

> [!important]+ El truco que más rinde
> Si controlas el cliente (tu host, una caja de lab, un proxy), exportar `SSLKEYLOGFILE` **antes** de generar el tráfico te deja descifrar tu propio HTTPS sin montar un MITM con CA. En un engagement, <mark style="background: #FF5582A6;">un `key log` o una clave privada sacados de un host comprometido convierten una captura muda en texto claro</mark>.

# WPA/WPA2: PSK + handshake

Con la **PSK** (o la PMK) y un **4-way handshake** capturado en el `pcap`, Wireshark deriva la PTK y descifra los datos 802.11: *Preferences ▸ Protocols ▸ IEEE 802.11 ▸ Enable decryption* + *Decryption keys* → tipo `wpa-pwd` con `contraseña:SSID` (o `wpa-psk` con la PMK en hex).

<mark style="background: #FFB8EBA6;">Sin el handshake EAPOL en la captura no hay descifrado</mark>: la PTK se deriva de él. Si falta, hay que forzar una reconexión para capturarlo (ver [[02 - Airodump-ng]] y la fase de captura del área Wi-Fi). El mismo trabajo offline —escribir un `pcap` ya descifrado— lo hace [[05 - Airdecap-ng]]. <mark style="background: #FFB86CA6;">WPA3/SAE no cae con esto</mark> (forward secrecy por diseño).

# Kerberos y el caso RDP de las imágenes

**Kerberos**: con la clave de la cuenta o un `keytab` (o la clave del servicio), Wireshark descifra tickets y tráfico `KRB5` (*Protocols ▸ KRB5 ▸ Kerberos keytab file*). Útil en engagements de AD para leer lo que viaja cifrado entre cliente, KDC y servicio.

**RDP**: el RDP moderno va **sobre TLS**, así que se descifra igual que cualquier TLS (key log, o clave privada del servidor si negoció RSA; el *RDP Standard Security* antiguo usa la clave privada del servidor directamente). Debajo, la misma conexión RDP antes y después de cargar la clave:

![[trafico_rdp_encriptado.png]]
![[trafico_rdp_desencriptado.png]]

<mark style="background: #8000E1A6;">El panel pasa de `TLS Application Data` ilegible a la negociación y los canales RDP disecados</mark> — pulsaciones, portapapeles, nombres de recursos redirigidos.

> [!warning]+ Lo que se espera mal
> - **TLS 1.3 solo por key log** — no existe el atajo de la clave privada.
> - **WPA necesita el handshake completo**; media captura no descifra.
> - El descifrado es **retroactivo pero local**: las claves viven en tus *Preferences*, no en el `pcap`. Para compartir una captura ya descifrada hay que re-exportarla (`airdecap-ng` para Wi-Fi, o *File ▸ Export* tras aplicar las claves).

> [!info]+ Fuente
> [Wireshark — TLS decryption](https://wiki.wireshark.org/TLS) · [HowToDecrypt802.11](https://wiki.wireshark.org/HowToDecrypt802.11). La mecánica y los ataques a TLS viven en el área HTTPs/TLS del vault; la captura del handshake Wi-Fi, en [[Aircrack-ng.base|Aircrack-ng]].
