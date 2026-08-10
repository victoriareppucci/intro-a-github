# Buenas prácticas de programación y desarrollo de software

Guía práctica para organizar un proyecto de ciencia de datos sin caos. Cubre entornos virtuales, manejo de dependencias y estructura de carpetas. No entra en Docker ni en infraestructura, para un proyecto de tesis o de análisis no hace falta esa complejidad.

## 1. Entornos virtuales (venv)

### ¿Para qué sirve?

Un entorno virtual es una instalación de Python aislada para tu proyecto. Sin uno, todas las librerías que instalás van a parar al Python global de tu sistema, y ahí empiezan los problemas: un proyecto necesita `pandas==1.5` y otro `pandas==2.1`, se pisan entre sí, y termina rompiéndose alguno de los dos.

Con un entorno virtual, cada proyecto tiene sus propias versiones de librerías, aisladas del resto.

### Crear y usar un venv

```bash
python3 -m venv venv
```

Esto crea una carpeta `venv/` con una copia de Python y un lugar separado para instalar paquetes.

Activarlo:

```bash
# Linux / Mac
source venv/bin/activate

# Windows (PowerShell)
venv\Scripts\Activate.ps1
```

Cuando está activo, vas a ver `(venv)` al principio de la línea de la terminal. A partir de ahí, todo lo que instales con `pip` queda dentro de esa carpeta, no en el sistema.

Desactivarlo:

```bash
deactivate
```

### Importante: no subir el venv al repo

La carpeta `venv/` puede pesar cientos de MB y es específica de tu máquina. Nunca se sube a Git. Se agrega al `.gitignore`:

```
venv/
```

Lo que sí se comparte es la lista de dependencias (ver siguiente sección), para que cualquiera pueda recrear el mismo entorno.

## 2. requirements.txt

Es un archivo de texto plano que lista las librerías que necesita el proyecto, con sus versiones. Es el método más simple y el más usado en ciencia de datos.

### Crearlo

Con el venv activado y tus librerías ya instaladas:

```bash
pip freeze > requirements.txt
```

Esto vuelca todo lo que está instalado en el entorno, con su versión exacta. Un archivo típico se ve así:

```
numpy==1.26.4
pandas==2.2.1
scikit-learn==1.4.0
matplotlib==3.8.3
```

### Instalarlo (por ejemplo, en otra máquina o para un compañero de equipo)

```bash
pip install -r requirements.txt
```

### Recomendaciones

- Corré `pip freeze > requirements.txt` cada vez que agregues o actualices una librería importante, así el archivo no queda desactualizado.
- `pip freeze` vuelca TODO lo instalado, incluidas dependencias de dependencias. Si el archivo queda muy largo y desordenado, una alternativa es mantener a mano un `requirements.txt` solo con las librerías que realmente usás directamente (numpy, pandas, etc.), sin fijar versión o con un mínimo (`pandas>=2.0`), y dejar que pip resuelva el resto.

## 3. Poetry como alternativa

`pip` + `requirements.txt` funciona, pero tiene límites: no distingue bien entre dependencias de producción y de desarrollo, no resuelve conflictos de versiones de forma automática, y no versiona el entorno de forma reproducible al 100%. Poetry resuelve esto.

### Qué hace distinto

- Maneja el entorno virtual automáticamente (no hace falta crear el venv a mano).
- Usa un archivo `pyproject.toml` para declarar las dependencias, en vez de `requirements.txt`.
- Genera un `poetry.lock` que fija las versiones exactas de todo el árbol de dependencias, así el entorno es 100% reproducible entre máquinas.
- Separa fácilmente dependencias normales de las de desarrollo (por ejemplo, `pytest` o `black` no hace falta que estén en producción).

### Instalación de Poetry

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

### Uso básico

Iniciar un proyecto nuevo:

```bash
poetry init
```

Esto te va preguntando datos del proyecto (nombre, versión, dependencias) y arma el `pyproject.toml`.

Agregar una dependencia:

```bash
poetry add pandas
poetry add pytest --group dev   # solo para desarrollo
```

Instalar todo lo declarado en `pyproject.toml`:

```bash
poetry install
```

Correr un script dentro del entorno de Poetry:

```bash
poetry run python mi_script.py
```

Activar una shell dentro del entorno (similar a `source venv/bin/activate`):

```bash
poetry shell
```

### ¿Cuál conviene usar?

| Situación | Conviene |
|---|---|
| Proyecto chico, notebook de análisis, TFI | `venv` + `requirements.txt` — más simple, todo el mundo lo conoce |
| Proyecto con muchas dependencias o que vas a mantener en el tiempo | Poetry — más prolijo y reproducible |
| Vas a compartir el proyecto como librería instalable | Poetry — está pensado para eso |

No hace falta elegir "el mejor" en abstracto: para un TFI, `venv` + `requirements.txt` suele alcanzar y sobrar.

## 4. Estructura de carpetas típica

Para un proyecto de ciencia de datos, una estructura ordenada y estándar ayuda a que cualquiera (incluido vos, dentro de tres meses) entienda dónde está cada cosa. Una organización común:

```
mi-proyecto/
├── data/
│   ├── raw/              # datos originales, sin tocar (no se editan a mano)
│   ├── processed/        # datos ya limpios/transformados, listos para usar
│   └── external/         # datos de fuentes externas (opcional)
│
├── notebooks/            # Jupyter notebooks de exploración y prototipado
│   └── 01-exploracion-inicial.ipynb
│
├── src/                  # código fuente reutilizable (funciones, clases)
│   ├── __init__.py
│   ├── preprocesamiento.py
│   ├── features.py
│   └── modelo.py
│
├── models/                # modelos entrenados serializados (.pkl, .joblib)
│
├── reports/                # resultados, gráficos, informes generados
│   └── figures/
│
├── tests/                  # tests del código en src/
│
├── requirements.txt         # (o pyproject.toml + poetry.lock si usás Poetry)
├── .gitignore
├── README.md
└── config.py                # rutas, constantes y parámetros centralizados (opcional)
```

### Ideas detrás de esta organización

- **`data/raw/` es intocable**: los datos originales nunca se editan a mano ni se sobreescriben. Cualquier transformación se hace por código y el resultado va a `data/processed/`. Así siempre podés reproducir el pipeline desde cero.
- **Separar notebooks de `src/`**: los notebooks son buenos para explorar y probar ideas rápido, pero tienden a ensuciarse (celdas fuera de orden, código repetido). Una vez que una función "sirve", conviene moverla a un archivo `.py` en `src/` para poder reusarla e importarla, tanto desde otros notebooks como desde scripts.
- **`data/` normalmente va en `.gitignore`**: los datasets suelen ser pesados y a veces no se pueden compartir públicamente (por privacidad, tamaño, o licencia). Se sube el código que los genera o los procesa, no los datos en sí — a menos que sean chicos y sea necesario tenerlos versionados.
- **Numerar los notebooks** (`01-`, `02-`, `03-...`) ayuda a que se entienda el orden en que se pensaron o se deben correr, sobre todo si hay varios.

Esta estructura no es una ley fija, es una convención bastante extendida en la comunidad (inspirada en proyectos como Cookiecutter Data Science). Se puede simplificar si el proyecto es chico, pero tener esta forma de referencia ayuda a no reinventar la organización cada vez.

## 5. Otras buenas prácticas generales

- **Un README claro**: qué hace el proyecto, cómo instalarlo (`pip install -r requirements.txt`), y cómo correrlo. Es lo primero que lee cualquiera que abre el repo.
- **Nombres de variables y funciones descriptivos**: `limpiar_datos(df)` dice más que `f(x)`.
- **Funciones chicas y con una sola responsabilidad**: más fáciles de testear, reusar y debuggear.
- **Docstrings en las funciones de `src/`**: una o dos líneas explicando qué hace, qué recibe y qué devuelve, alcanza.
- **No hardcodear rutas ni valores mágicos**: centralizalos en un `config.py` o al principio del script, para poder cambiarlos en un solo lugar.
- **Versionar el código, no los resultados pesados**: modelos entrenados grandes, datasets, o notebooks con outputs de varios MB generalmente no van al repo (o van aparte, con Git LFS si hiciera falta).
