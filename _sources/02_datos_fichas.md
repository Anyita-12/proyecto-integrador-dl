# Datos y fichas de datos

En este capítulo documentamos cada conjunto de datos antes de modelarlo: origen, términos de uso, recolección, anotación, composición, usos previstos y no previstos, limitaciones y riesgos de privacidad. Verificamos el esquema real de los archivos y registramos dónde difiere de lo que supone el enunciado. Las fichas se mantienen como archivos independientes en `reports/datasheets/`; aquí las incluimos sin cambios.

```{include} ../reports/datasheets/sartorius.md
```

```{include} ../reports/datasheets/jarvis.md
```

## Cómo se obtienen y verifican los datos

Un solo comando descarga, verifica y prepara los dos conjuntos:

```bash
make data      # descarga con la API de Kaggle si faltan los zip (variable KAGGLE_API_TOKEN) y verifica los SHA-256
make splits    # descomprime, reconstruye las dos particiones con la semilla 42 y comprueba que coinciden con data/splits/
```

`scripts/prepare_data.py` reconstruye cada partición desde cero y la compara con el archivo versionado. Si la partición reconstruida no coincide, el *script* se detiene. Las pruebas de `tests/test_data_splits.py` comprueban, además, que ningún `sample_id` (Sartorius) ni ningún grupo de casi duplicados (Jarvis) cruza particiones, y validan el esquema de los dos archivos con `pandera`.

## Unidad de análisis

| Dominio | Unidad de análisis | Consecuencia |
|---|---|---|
| Segmentación | Imagen | Los intervalos de confianza remuestrean imágenes, no píxeles ni células. Las células de una misma imagen comparten condiciones de cultivo y de adquisición, así que no son independientes. |
| Segmentación (verificación) | `sample_id` | Un modelo mixto con intercepto aleatorio por `sample_id` comprueba que las conclusiones no dependen de la correlación entre imágenes de la misma muestra. |
| NLP | Documento (test) y pliegue (validación cruzada) | McNemar y *bootstrap* sobre documentos de test; *t* corregida de Nadeau–Bengio sobre los pliegues {cite:p}`nadeau2003inference`. |
