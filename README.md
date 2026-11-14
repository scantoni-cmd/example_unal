# Visualización genérica de datos de Google Earth Engine

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USUARIO/REPOSITORIO/blob/main/notebooks/gee_data_visualization.ipynb)

Notebook educativo y técnico para **consultar, procesar y visualizar** datasets del catálogo de [Google Earth Engine](https://earthengine.google.com/) (GEE) con Python, `earthengine-api` y `geemap`. Cambias unos pocos parámetros (dataset, banda, período, área de estudio, visualización) y obtienes mapas interactivos, estadísticas y, si quieres, exportaciones, **sin modificar la lógica principal**.

Funciona en **Google Colab** y en **Jupyter local**.

---

## Contenido

| Ejemplo | Dataset (ID en GEE) | Banda | Procesamiento | Unidades |
|---|---|---|---|---|
| 1. Cobertura del suelo | ESA WorldCover v200 (`ESA/WorldCover/v200`) | `Map` | Mosaico, paleta categórica + leyenda, área por clase | clase; ha y % |
| 2. Elevación | NASA SRTM 30 m (`USGS/SRTMGL1_003`) | `elevation` | Estadísticas, pendiente y sombreado | m; grados |
| 3. Precipitación | CHIRPS Daily v2.0 (`UCSB-CHG/CHIRPS/DAILY`) | `precipitation` | Acumulado (`sum`) y promedio (`mean`) del período | mm/d → mm y mm/d |
| 4. Temperatura | ERA5-Land Daily Aggregates (`ECMWF/ERA5_LAND/DAILY_AGGR`) | `temperature_2m` | Media del período, K → °C | K → °C |
| Final | Los cuatro anteriores | — | Mapa combinado con control de capas, inspector y *split map* | — |

AOI por defecto: un rectángulo de ~45 × 40 km en el **Valle del Cauca (Colombia)** con Cali, el valle del río Cauca y los Farallones de Cali. También se puede definir con un **polígono**, un **GeoJSON** o una **división administrativa** (FAO GAUL 2025).

---

## Estructura del repositorio

```
gee-data-visualization/
├── README.md                     ← este archivo
├── GOOGLE_CLOUD_SETUP.md         ← guía paso a paso: Google Cloud + Earth Engine
├── requirements.txt              ← dependencias para ejecución local
├── .gitignore                    ← excluye credenciales, rasters, entornos, checkpoints
├── notebooks/
│   └── gee_data_visualization.ipynb
└── data/
    └── aoi_example.geojson       ← AOI de ejemplo (Cali)
```

**Ubicación del notebook:** `notebooks/gee_data_visualization.ipynb`. El enlace de Colab depende de esa ruta.

Para crecer el proyecto sin tocar la lógica principal:

- **Nuevos datasets:** agrega una ficha al diccionario `DATASET_REGISTRY` del notebook (sección 15 del notebook). Si el registro crece, puede moverse a `config/datasets.json` y cargarse al inicio.
- **Nuevas áreas de estudio:** agrega archivos `.geojson` en `data/` y apunta `AOI_GEOJSON` a ellos.
- **Nuevos notebooks temáticos:** colócalos en `notebooks/` reutilizando las mismas funciones. Si se comparten entre varios notebooks, el siguiente paso natural es extraer las funciones de la sección 7 a un paquete `src/geeviz/`.

---

## Abrir en Google Colab

1. Sube el repositorio a GitHub.
2. Edita **dos lugares** para reemplazar `USUARIO` y `REPOSITORIO` por tu usuario (u organización) y el nombre del repositorio:
   - El badge al inicio de este `README.md`.
   - El badge de la primera celda del notebook.
3. Haz clic en el badge **Open in Colab**.

Formato del enlace:

```
https://colab.research.google.com/github/<USUARIO>/<REPOSITORIO>/blob/<RAMA>/<RUTA_AL_NOTEBOOK>
```

Ejemplo con la estructura propuesta:

```
https://colab.research.google.com/github/USUARIO/REPOSITORIO/blob/main/notebooks/gee_data_visualization.ipynb
```

Si mueves o renombras el notebook, o usas otra rama (p. ej. `master`), cambia `<RAMA>` y `<RUTA_AL_NOTEBOOK>` en ambos badges. El código Markdown del badge es:

```markdown
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USUARIO/REPOSITORIO/blob/main/notebooks/gee_data_visualization.ipynb)
```

> Colab abre **solo el notebook**, no el resto del repositorio. Por eso el notebook genera automáticamente el GeoJSON de ejemplo si no encuentra `data/aoi_example.geojson`, y también acepta una URL *raw* de GitHub en `AOI_GEOJSON`.

Repositorios privados: Colab pedirá autorizar el acceso a GitHub.

---

## Requisitos previos

1. Cuenta de Google.
2. Proyecto de Google Cloud con la **Earth Engine API habilitada** y **registrado para Earth Engine**.
3. Roles mínimos en ese proyecto: `roles/serviceusage.serviceUsageConsumer` + `roles/earthengine.viewer` (o ser Owner).

Si es tu primera vez con GEE, sigue **[GOOGLE_CLOUD_SETUP.md](GOOGLE_CLOUD_SETUP.md)** (15–20 minutos).

---

## Instalación local

```bash
git clone https://github.com/USUARIO/REPOSITORIO.git
cd REPOSITORIO

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# (opcional) autenticarse una vez desde la terminal
earthengine authenticate

jupyter lab notebooks/gee_data_visualization.ipynb
```

Requiere Python ≥ 3.10.

---

## Uso rápido

1. Abre el notebook (Colab o local) y ejecuta la celda de instalación.
2. En **Configuración global** cambia `PROJECT_ID` (o define `EE_PROJECT_ID` como variable de entorno / *Secret* de Colab).
3. *Ejecutar todo*.

### Visualizar cualquier dataset con la función genérica

```python
res = visualize_gee_dataset(
    dataset_id="UCSB-CHG/CHIRPS/DAILY",
    band="precipitation",
    aoi=AOI,                              # ee.Geometry, bbox, lista de vértices o GeoJSON
    dataset_type="ImageCollection",       # o "Image" o "auto"
    start_date="2024-03-01", end_date="2024-06-01",   # end_date exclusiva
    reducer="sum",                        # mean | sum | median | min | max | mode | mosaic | first
    vis_params={"palette": ["f7fbff", "6baed6", "08306b"]},   # min/max automáticos (p2–p98)
    colorbar_label="Precipitación acumulada (mm)",
    compute_stats=True,
)
res.statistics      # mínimo, máximo, media, desviación, n píxeles
res.map             # mapa interactivo
```

### Usar una ficha del registro

```python
res_t = visualize_from_registry("temp_era5land", AOI, start_date="2024-01-01", end_date="2024-04-01",
                                compute_stats=True)
```

### Cambiar el área de estudio

En la celda de configuración:

```python
AOI_METHOD = "bbox"        # "bbox" | "polygon" | "geojson" | "admin"
AOI_BBOX = (-74.20, 4.50, -73.95, 4.80)   # p. ej. Bogotá
```

---

## Validación realizada y pruebas que requieren credenciales

**Ejecutado sin credenciales de GEE** (en el entorno donde se generó el proyecto, con earthengine-api 1.7.52, geemap 0.39.0, pandas 3.1.1):

- El notebook es un `.ipynb` válido según el esquema de `nbformat`.
- Todas las celdas de código compilan (`ast`) y pasan `pyflakes` sin advertencias.
- Se ejecutaron las celdas de definición y se probaron las funciones locales: validación de bbox, polígonos y fechas; conteo de días; normalización del tipo de dataset; nombres de tareas; lectura de GeoJSON con generación automática del archivo de ejemplo; traducción de errores; registro de datasets.
- Se verificó que la paleta categórica de WorldCover asigna exactamente el color oficial a cada código de clase (10…100) según la regla de interpolación lineal de EE.
- Se verificaron las firmas reales de `geemap.Map` (`add_legend`, `add_colorbar`, `add_layer_control`, `split_map`, `add_inspector`) y de `ee.batch.Export.image.toDrive`, y que leyendas y barras de color se construyen correctamente.
- IDs, bandas, unidades, resoluciones y esquema de FAO GAUL 2025 se contrastaron con las páginas oficiales del catálogo.

**Requiere un proyecto de GEE real** (no ejecutado aquí, por diseño no se incluyen credenciales):

- Autenticación, inicialización y `verify_connection()`.
- Todas las consultas al servidor: carga de datasets, filtrado, agregaciones, `reduceRegion` (estadísticas, percentiles, áreas por clase), renderizado de teselas y exportaciones.
- Disponibilidad temporal efectiva de CHIRPS y ERA5-Land para las fechas elegidas (ambos productos se publican con rezago).

Por eso el notebook se distribuye **sin salidas**; al ejecutarlo con tu proyecto se generan los mapas y tablas.

---

## Fuentes de datos y citas

- **ESA WorldCover 10 m 2021 v200** — Zanaga, D. et al. (2022). doi:[10.5281/zenodo.7254221](https://doi.org/10.5281/zenodo.7254221). CC-BY-4.0.
- **NASA SRTM v3 (SRTMGL1)** — NASA JPL (2013). Dominio público.
- **CHIRPS v2.0** — Funk, C. et al. (2015). *Scientific Data* 2, 150066. doi:[10.1038/sdata.2015.66](https://doi.org/10.1038/sdata.2015.66). Dominio público.
- **ERA5-Land** — Muñoz-Sabater, J. et al. (2021). *Earth System Science Data* 13, 4349–4383. doi:[10.5194/essd-13-4349-2021](https://doi.org/10.5194/essd-13-4349-2021). Contiene información modificada de Copernicus Climate Change Service.
- **FAO GAUL 2025** — FAO (2025). Global Administrative Unit Layers. CC-BY-4.0.
- Páginas del catálogo: <https://developers.google.com/earth-engine/datasets>

---

## Seguridad

- No se versionan credenciales, tokens ni claves (ver `.gitignore`).
- El ID del proyecto no es un secreto, pero puede mantenerse fuera del notebook con `EE_PROJECT_ID` (variable de entorno o *Secret* de Colab).
- Para automatización desatendida usa cuentas de servicio sin subir sus claves (ver `GOOGLE_CLOUD_SETUP.md`, sección 13).

## Licencia

Define la licencia del código (p. ej. MIT) en un archivo `LICENSE`. Los datos mantienen las licencias de sus proveedores.
