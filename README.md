# Proyecto integrador de aprendizaje automático (sección 4.8)

Segmentación de instancias celulares (Sartorius), clasificación de texto (Jarvis Calling Hiring Contest) y despliegue con MLOps. Curso de Deep Learning, Universidad del Norte.

**Autores:** Anyely Sanjuan Benavides y Juan David Díaz Fernández · **Profesor:** Dr. Lihki Rubio.

## Estructura

```text
data/        datos crudos y procesados (fuera de Git; hashes en data/SHA256SUMS.txt)
notebooks/   00_prereg_protocol … 07_drift_simulation (leen results/, no entrenan)
src/project/ paquete instalable: data, metrics, models, stats, calibration, drift
app/         API FastAPI
configs/     configuración YAML por experimento
tests/       pytest: unitarias, datos, regresión de modelo y API
models/      artefactos registrados (con hash)
reports/     model cards, datasheets y figuras
book/        Jupyter Book
results/     salidas de los entrenamientos (tablas pequeñas versionadas)
```

## Datos

Los datos se descargan desde Kaggle tras aceptar las reglas de cada competencia y no se versionan en Git:

```powershell
kaggle competitions download -c sartorius-cell-instance-segmentation -p data\raw\sartorius
kaggle competitions download -c jarvis-calling-hiring-contest -p data\raw\jarvis
```

## Enlace HTML

https://anyita-12.github.io/proyecto-integrador-dl/intro.html

## Flujo de trabajo

Rama `feature/<tarea>` → pull request revisado por la otra persona → merge a `main` → recompilar y publicar el Book.
