---
tags:
  - Web/Red-Team
  - Deserializacion
  - Pentesting/Explotacion
Descripción: "Las otras librerías de serialización de Python vulnerables: jsonpickle, PyYAML y ruamel, qué API es insegura y qué cambió para cerrarlas por defecto"
Fecha de actualización: 2026-11-20
Nota previa: "[[10 - Bypass de filtros en deserialización Python]]"
Nota siguiente: "[[12 - Detección y evasión de deserialización]]"
Area: "[[Deserialización.base|Deserialización]]"
---
---

`pickle` es la librería nativa, pero no la única que deserializa objetos Python arbitrarios. `jsonpickle`, `PyYAML` y `ruamel.yaml` sufren el mismo problema de raíz —construyen objetos a partir de datos— y comparten la técnica de explotación: un `__reduce__` que devuelve un callable y sus argumentos. Lo que cambia es el formato del `payload` y, sobre todo, **qué API es segura y cuál no**, porque aquí los vendors sí han movido los defaults.

# jsonpickle

`jsonpickle` serializa objetos a JSON conservando el tipo. La explotación es idéntica a `pickle`: se define `__reduce__` y se codifica con `jsonpickle.encode`. El `sink` es `jsonpickle.decode`.

```python
import jsonpickle, os

class RCE:
    def __reduce__(self):
        return os.system, ("id",)

payload = jsonpickle.encode(RCE())
# {"py/reduce": [{"py/function": "posix.system"}, {"py/tuple": ["id"]}]}
jsonpickle.decode(payload)   # ejecuta id
```

<mark style="background: #ADCCFFA6;">La firma reconocible es `py/object`, `py/reduce`, `py/function` o `py/tuple` dentro de un JSON</mark> — un JSON «normal» no las lleva. Si aparecen, hay deserialización de objetos Python detrás de lo que parece una simple API JSON.

> [!danger]+ `safe=True` NO hace seguro a jsonpickle
> `jsonpickle.decode()` tiene un parámetro `safe`, y desde la **v4.0.0** vale `True` por defecto. Es una trampa: <mark style="background: #FF5582A6;">`safe=True` solo desactiva la ruta `eval()` heredada (objetos serializados con `repr` de antes de la 0.7.0); **no** impide que jsonpickle instancie clases arbitrarias por las etiquetas `py/object`/`py/reduce`</mark>. La documentación oficial lo dice con todas las letras: *«arguments like `safe=True` do not make it safe to load an untrusted jsonpickle string»*. Es decir, `jsonpickle.decode()` sigue siendo un `sink` de RCE con entrada no confiable, `safe=True` incluido. El parámetro está además marcado para desaparecer (deprecado en v4.1.2). La verdadera restricción es el parámetro `classes` como allowlist, con sus límites.

# PyYAML — y la errata de HTB

YAML puede llevar tipos ricos, y `PyYAML` los construye. La explotación clásica:

```python
import yaml, subprocess

class RCE:
    def __reduce__(self):
        return subprocess.Popen, (["id"],)

payload = yaml.dump(RCE())       # genera !!python/object/apply:...
```

El `payload` es texto YAML con la etiqueta `!!python/object/apply:` o `!!python/object/new:`, que es la firma a reconocer.

> [!danger]+ ERRATA del material de HTB: `yaml.load(payload)` ya no funciona así
> El ejemplo del módulo deserializa con `yaml.load(exploit)` a secas. <mark style="background: #FF5582A6;">Desde PyYAML **6.0** (2021-10-13, PR #561), `yaml.load()` exige el argumento `Loader` de forma posicional; sin él, CPython aborta con `TypeError: load() missing 1 required positional argument: 'Loader'` antes de parsear nada</mark>. El ejemplo de HTB lleva cinco años sin ejecutarse tal cual. Cronología: PyYAML 5.1 (2019) añadió el aviso y `FullLoader`/`UnsafeLoader`; 5.4 arregló CVE-2020-14343; 6.0 hizo `Loader` obligatorio. En un objetivo actual el `sink` vulnerable es uno de estos:
> ```python
> yaml.load(data, Loader=yaml.Loader)       # inseguro, explícito
> yaml.load(data, Loader=yaml.UnsafeLoader) # inseguro, explícito
> yaml.unsafe_load(data)                     # inseguro, atajo
> ```
> El `loader` seguro es `yaml.SafeLoader` (o `yaml.safe_load`), que **solo** construye tipos básicos y rechaza `!!python/object`. Buscar en código `Loader=Loader`, `UnsafeLoader` o `unsafe_load`; un `safe_load` no es explotable.

La implicación para el pentester: encontrar `yaml.load(user_data)` sin `Loader` en una base de código actual significa que esa dependencia es **anterior a PyYAML 6.0** (2021) o que el código fija el `loader` inseguro aparte — en ambos casos, un hallazgo adicional sobre dependencias desactualizadas.

# ruamel.yaml

`ruamel.yaml` es un fork de PyYAML con mejor soporte de YAML 1.2 y preservación de comentarios. Comparte la mecánica y la superficie:

```python
from ruamel.yaml import YAML
yaml = YAML(typ='unsafe')   # <-- el typ inseguro
yaml.load(payload)
```

<mark style="background: #FFB8EBA6;">La API moderna de `ruamel.yaml` exige elegir un `typ` al instanciar</mark>: `safe` (solo tipos básicos), `rt` (round-trip, por defecto, también seguro para construcción de objetos arbitrarios) y `unsafe` (construye objetos Python, explotable). El `sink` es `YAML(typ='unsafe').load(...)`. El estilo antiguo de funciones de módulo (`ruamel.yaml.load(data, Loader=...)`) era igualmente inseguro, pero **se eliminó en la 0.18** (hoy da `AttributeError`, no un aviso): en código reciente el único vector es `typ='unsafe'`.

# Tabla de decisión rápida

Lo que de verdad hay que llevar a un `engagement`:

| Librería | `sink` (inseguro) | API segura |
| --- | --- | --- |
| `pickle` | `pickle.loads` / `load` | *(no hay modo seguro; no usar con datos no confiables)* |
| `jsonpickle` | `jsonpickle.decode` (incluido con `safe=True`) | No hay modo realmente seguro; restringir con el parámetro `classes` como allowlist |
| `PyYAML` | `yaml.load` con `Loader`/`UnsafeLoader`, `yaml.unsafe_load` | `yaml.safe_load` / `SafeLoader` |
| `ruamel.yaml` | `YAML(typ='unsafe')` | `YAML(typ='safe')` |
| `numpy` | `load(..., allow_pickle=True)` | `allow_pickle=False` (defecto moderno) |
| `pandas` | `read_pickle` | *(evitar con datos no confiables)* |

# PEAS: el multi-tool

**PEAS** (`python-deserialization-attack-payload-generator`) genera `payloads` para `pickle`, `jsonpickle`, `PyYAML` y `ruamel.yaml` desde una sola interfaz. Como se vio en [[10 - Bypass de filtros en deserialización Python]], su `payload` de serie usa `subprocess.Popen`, que cae en muchas blacklists, y suele requerir edición. Su estado de mantenimiento y las alternativas actuales están en [[14 - Arsenal de herramientas de deserialización]].

> [!important]+ El ecosistema ML es el frente caliente
> Más allá de la web, la deserialización Python se ha convertido en el principal vector de supply chain en machine learning: `torch.load`, `joblib`, `keras` y los formatos de modelo son `pickle` por debajo. Es un tema propio con su superficie y sus herramientas (`fickling`, `picklescan`, `modelscan`), tratado en [[11 - Pickle y la deserialización insegura de modelos]] y [[12 - Esteganografía en tensores]]. Si el objetivo carga modelos de terceros, ese es el camino.

> [!info]+ Fuentes
> Cambio de `yaml.load` en PyYAML 6.0: [PyYAML — documentación y changelog](https://pyyaml.org/wiki/PyYAMLDocumentation) y el repo [yaml/pyyaml](https://github.com/yaml/pyyaml). Modos de `ruamel.yaml`: [ruamel.yaml docs](https://yaml.readthedocs.io/). Endurecimiento de jsonpickle: [repo jsonpickle](https://github.com/jsonpickle/jsonpickle). Ejemplos de partida del módulo 169 de HTB Academy, con la errata de `yaml.load` corregida y las APIs actualizadas a 2026.
