---
tags:
  - OSINT/Dark-Web
  - Pentesting/Enumeracion
Descripción: "Descubrir fuentes en la dark web: Ahmia, Torch y OnionSearch, directorios verificados, leak sites de ransomware y sitios de pastes"
Fecha de actualización: 2026-11-08
Nota previa: "[[01 - Acceso y OPSEC en Tor]]"
Nota siguiente: "[[03 - Monitorización y el ángulo CTI]]"
Area: "[[Dark web OSINT.base|Dark web OSINT]]"
---
---

No hay un Google de la dark web: los servicios ocultos no se indexan bien, cambian de dirección y muchos duran poco. Descubrir fuentes es un problema en sí mismo, resuelto con una mezcla de motores, directorios y feeds especializados — la mayoría consultables desde clearnet.

# Motores de búsqueda onion

- <mark style="background: #ADCCFFA6;">**Ahmia** es el buscador de referencia</mark>: indexa `.onion`, es accesible por **web normal** (no necesitas Tor para buscar, solo para abrir el resultado) y filtra contenido abusivo conocido.
- **Torch**: motor más antiguo con un índice amplio de `.onion`; útil como segunda fuente, con más ruido.
- <mark style="background: #ADCCFFA6;">**OnionSearch**</mark> es un meta-buscador en Python que consulta **varios** motores onion a la vez:

```shell-session
# Buscar un término en múltiples motores de la dark web
$ python3 onionsearch.py "acme.com" --output resultados.txt
```

Combinar Ahmia + OnionSearch es el método para descubrir leak sites o menciones nuevas antes de que se reporten.

# Directorios verificados

Como las direcciones son irrecordables y clonables ([[01 - Acceso y OPSEC en Tor#Verificar las direcciones .onion|clones]]), los directorios curados importan:

- **dark.fail**: lista de servicios conocidos (mercados, foros, buscadores) con sus **direcciones verificadas** y firmas PGP. El punto de partida fiable.

# Leak sites de ransomware

<mark style="background: #FFB86CA6;">Los grupos de ransomware publican a sus víctimas en sitios `.onion` dedicados</mark>, con muestras de los datos robados como prueba. Para el red teamer/CTI es una fuente de altísimo valor:

- **ransomware.live** (clearnet) **consolida** las víctimas de decenas de grupos — buscas el nombre del cliente y sabes si ha sido golpeado y por quién.
- Si el cliente aparece, deja de ser recon: es un incidente que **escalas de inmediato**.

*(RansomWatch, otra referencia histórica, está archivado — usa ransomware.live.)*

# Pastes y foros de brechas

- **Sitios de pastes** (Pastebin y alternativas): los atacantes vuelcan ahí credenciales, dumps y datos "de muestra". Son efímeros, así que se **monitorizan** (ver [[03 - Monitorización y el ángulo CTI|monitorización]]); servicios como psbdmp archivan pastes borrados.
- **Foros de brechas**: la estirpe de *BreachForums* (incautado y renacido varias veces) es donde se anuncian y venden bases de datos. El acceso suele requerir [[01 - Sock puppets - crear y mantener identidades de investigación|registro con sock puppet]]; recuerda, observar, no participar.

# Cómo buscar bien

<mark style="background: #8000E1A6;">Dirigido, no exploratorio</mark>: busca el **nombre del cliente**, sus **dominios**, sus **correos**, sus **marcas** y los **nombres de directivos**. No "navegues" la dark web — es ruido, estafa y riesgo. Lo que encuentres, verifícalo ([[05 - Verificación, fiabilidad de fuentes y sesgos|los vendedores mienten y plantan datos falsos]]) y, si es del cliente, documéntalo sin descargarlo salvo cobertura legal.

> [!info]+ Fuentes
> [Ahmia](https://ahmia.fi), Torch, [OnionSearch](https://github.com/megadose/OnionSearch) (megadose), [dark.fail](https://dark.fail), [ransomware.live](https://www.ransomware.live). Panorama de recursos: [Web Asha, *21 Dark Web Resources*](https://www.webasha.com/blog/top-21-dark-web-resources-every-osint-professional-should-know).
