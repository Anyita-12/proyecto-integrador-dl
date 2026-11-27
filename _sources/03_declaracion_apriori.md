# Declaración a priori

En este capítulo reunimos las decisiones que fijamos antes de entrenar cualquier modelo final: hipótesis, métricas, particiones, semillas, presupuesto, criterios de parada y plan estadístico. Las congelamos el 25 de noviembre de 2026 en `results/common/preregistro_2026-11-25.md`, con la etiqueta Git `prereg` (commit `4405406`). El archivo no se modifica después de la congelación. Toda decisión posterior aparece en la tabla de desviaciones al final del capítulo y en `reports/bitacora.md`, con fecha, hora y motivo.

## Tabla de pre-registro

| Elemento | Segmentación (Sartorius) | Clasificación de texto (Jarvis) |
|---|---|---|
| Unidad de análisis | Imagen | Documento (test) y pliegue (validación cruzada) |
| Métrica primaria | AP@[0.5:0.95] por imagen, como en la competencia | F1 macro |
| Métricas secundarias | AP50, AP75, PQ (SQ, RQ), AJI, precisión, *recall* y F1 de instancia, error de conteo, Dice, IoU, precisión y *recall* de píxel, HD95, F1 de borde, AUC-ROC y AUC-PR de píxel, *balanced accuracy* de píxel, taxonomía de errores, latencia, parámetros, VRAM, CO₂ | *Accuracy*, *balanced accuracy*, precisión, *recall* y F1 (macro, micro y ponderado), MCC, ROC-AUC y PR-AUC uno contra resto, *log-loss*, Brier, ECE (15 intervalos), F1 por clase, latencia, parámetros, CO₂ |
| Partición | `StratifiedGroupKFold` agrupado por `sample_id`, estratificado por tipo celular: 421 / 92 / 93 | Sin duplicados exactos, casi duplicados agrupados (MinHash/LSH), estratificada por categoría: 1736 / 373 / 373 |
| Efecto mínimo relevante δ | 0.02 de AP | 0.02 de F1 macro (unos 7 documentos de test) |
| Semillas | 3 en modelos con entrenamiento (U-Net++, Mask R-CNN); 1 en los exploratorios | 5 en todos los modelos |
| Búsqueda de hiperparámetros | Solo post-proceso, en validación | Optuna TPE con *MedianPruner*, 30 *trials* por modelo ligero {cite:p}`akiba2019optuna,bergstra2011tpe`; DistilBERT con rejilla de 3 tasas de aprendizaje |
| Parada | U-Net++ y Hover: ≤ 35 épocas de 100 pasos, paciencia 8; Mask R-CNN: ≤ 24 épocas, paciencia 5 | Redes ligeras: paciencia 3; DistilBERT: ≤ 6 épocas, paciencia 2 |
| Presupuesto | Unas 29 h de GPU en total (RTX 4060 Laptop, 8 GB), con 20 % de holgura | (compartido) |

Elegimos δ = 0.02 en los dos dominios. En segmentación, por debajo de esa diferencia de AP la elección entre modelos la deciden el costo, la latencia y la robustez. En texto, con 24 clases y 373 documentos de test, una diferencia menor cambia la clase de menos de 7 currículos.

## Hipótesis

| ID | Hipótesis | Criterio de refutación |
|---|---|---|
| H1 | Mask R-CNN preentrenado en COCO supera en AP a la U-Net++ entrenada desde cero cuando ambos usan el 25 % del entrenamiento, y la brecha se reduce con el 100 %. | El IC 95 % *bootstrap* de la diferencia media pareada por imagen al 25 % incluye 0; o la diferencia al 100 % no es menor que al 25 %. |
| H2 | El AUC-ROC y la *balanced accuracy* de píxel superan 0.95 en todos los modelos con mapa de probabilidad y no los discriminan; AP@[0.5:0.95] sí. | La correlación de Spearman entre el orden por AUC-ROC de píxel y el orden por AP es mayor que 0.9. |
| H3 | El orden de los modelos por AP cambia al estratificar por tipo celular. | El orden es idéntico en astrocitos, corticales y SH-SY5Y. |
| H4 | La diferencia de F1 macro entre DistilBERT y TF-IDF + XGBoost **no** es significativa tras la corrección de Holm. | McNemar exacto en test o *t* de Nadeau–Bengio sobre 5 pliegues con $p_\text{Holm} < 0.05$. Además, comprobamos si el IC de la diferencia cae entero en $[-\delta, \delta]$: no significativo no implica equivalente. |
| H5 | El *fine-tuning* de DistilBERT produce una ECE de test mayor que 0.05 y mayor que la de TF-IDF + regresión logística; el escalado de temperatura ajustado en validación la reduce sin cambiar la exactitud. | La ECE antes del escalado es ≤ 0.05, o el IC *bootstrap* de la reducción de ECE incluye 0. La exactitud idéntica se cumple por construcción (capítulo {doc}`04_fundamentos_teoricos`). |

La H2 compara órdenes de 4 a 5 modelos, y con tan pocos elementos la correlación de Spearman tiene poca potencia. Lo tenemos en cuenta al interpretar el resultado.

## Plan estadístico

Usamos α = 0.05 y corrección de Holm en toda familia de comparaciones.

- **Intervalos de confianza:** *bootstrap* percentil con 10 000 remuestreos sobre la unidad de análisis (imágenes o documentos).
- **Dos modelos de segmentación:** Wilcoxon de rangos con signo, pareado por imagen.
- **Tres o más modelos de segmentación:** Friedman, Wilcoxon–Holm por pares y diagrama de diferencia crítica (Nemenyi) {cite:p}`demsar2006comparisons`.
- **Tamaños de efecto:** *d* de Cohen pareado y delta de Cliff, interpretados respecto a δ.
- **Estructura jerárquica:** modelo mixto `AP ~ modelo + (1 | sample_id)`.
- **Dos modelos de texto:** McNemar exacto en test, con la predicción por voto de las 5 semillas, y *t* remuestreada corregida de Nadeau–Bengio sobre los pliegues {cite:p}`nadeau2003inference,dietterich1998tests`.
- **Ablaciones:** modelo mixto `AP ~ variante + (1 | imagen)` sobre la media entre semillas por imagen, y Wilcoxon–Holm contra la variante de referencia.
- **Varianza:** media ± DE entre semillas, separada de la varianza entre imágenes y entre pliegues {cite:p}`bouthillier2021variance`.

## Reglas de reporte

- Toda cifra va como media [IC 95 %].
- Las figuras cualitativas muestran una muestra aleatoria con semilla fija y los 5 mejores y 5 peores casos por métrica.
- Cada métrica tiene su definición y su implementación: `src/project/metrics_seg.py` y `src/project/metrics_nlp.py`.
- El test se evalúa una sola vez por modelo final. Los entrenamientos guardan las probabilidades de test con la configuración congelada, pero las métricas de test solo se calculan en la evaluación final, después de cerrar todas las decisiones en validación.
- Todo resultado con menos semillas de las exigidas (5 en modelos ligeros, 3 en pesados) se rotula como *exploratorio*.

## Recortes declarados frente al enunciado

Los recortes se aplican por igual a todos los modelos comparados. Los fijamos antes de entrenar, a partir de un *benchmark* de la U-Net++ (0.43 s por paso de 8 recortes de 256 × 256 y 7.0 GB de VRAM pico):

1. Sin búsqueda bayesiana de los hiperparámetros de entrenamiento en segmentación. Solo optimizamos el post-proceso en validación.
2. Sin validación anidada completa en CNN-1D y BiLSTM: TPE sobre un único *split* y validación cruzada de 5 pliegues con la configuración elegida.
3. Mask R-CNN con *backbone* ResNet-50 en lugar de Swin; variante propia tipo Hover-Net en lugar de la implementación oficial; sin RoBERTa; SAM sin *fine-tuning*.
4. Sin ablaciones de resolución, *augmentation* ni preentrenamiento en segmentación. La ablación de preentrenamiento queda cubierta en parte por H1 (Mask R-CNN preentrenado frente a U-Net++ desde cero).
5. Sin validación cruzada de 5 pliegues en segmentación. Estimamos la varianza entre semillas y entre imágenes.

## Desviaciones y análisis añadidos después de la congelación

| Fecha y hora | Cambio | Motivo | Efecto en la comparación |
|---|---|---|---|
| 11-25 07:36 | Tope de tiempo en la búsqueda de hiperparámetros: 60 min para CNN-1D y 90 min para BiLSTM | Presupuesto de GPU | Si el tope corta antes de 30 *trials*, el número real queda en `hpo.pkl` y lo reportamos en el capítulo de modelos de texto |
| 11-25 07:47 | Validación anidada completa también en TF-IDF + XGBoost y FastText | El enunciado la exige para los modelos ligeros y en CPU es barata | Reduce el recorte 2 |
| 11-25 07:48 | Ablaciones de segmentación intercaladas por semilla | Si falta GPU, cada variante tiene al menos una semilla | Las variantes con menos de 3 semillas se rotulan como exploratorias |
| 11-25 08:00 | Robustez de texto con 11 perturbaciones de test predichas por el mismo modelo final | Ahorra un reentrenamiento por modelo | Análisis añadido |
| 11-25 08:00 | Curvas de aprendizaje de texto (10, 25, 50 y 100 %) y ablación de *embeddings* de la BiLSTM | Exigidas por el enunciado y ausentes del pre-registro | Análisis añadidos |
| 11-25 08:04 | *Benchmark* de complejidad (U-Net++, BiLSTM y auto-atención) | Verificación empírica del punto 6 de los fundamentos teóricos | Análisis añadido |
| 11-25 14:33 | La validación anidada de TF-IDF + XGBoost y FastText pasa al final de la cola, después de las ablaciones de segmentación (corrige la fila de las 07:47) | Medimos el costo en la búsqueda simple: XGBoost tardó 117 min en 30 *trials* y FastText unos 6 min por *trial* (ambos en CPU), de modo que cada validación anidada (5 × 31 ajustes) requiere de 10 a 15 h, más que el presupuesto disponible | Si el corte del jueves llega antes, esos dos modelos quedan sin corrección del sesgo optimista, igual que CNN-1D, BiLSTM y DistilBERT; el capítulo de modelos de texto lo indica |
| 11-25 14:35 | Orden de la cola por prioridad: (1) trabajos de texto que deciden H4 y H5 y los entregables de texto; (2) trabajos de segmentación que deciden H1 (U-Net++ y Mask R-CNN al 25 %), segunda y tercera semilla de Mask R-CNN, HoVer-Net, Cellpose ajustado, SAM automático, selección, robustez, exportación y prueba de carga; (3) análisis secundarios baratos de texto; (4) ablaciones de segmentación con la semilla 0; (5) análisis secundarios caros de texto; (6) semillas 1 y 2 de las ablaciones; (7) validaciones anidadas de XGBoost y FastText | Con las duraciones medidas hasta las 14:40, el trabajo pendiente supera el tiempo hasta el corte del jueves a las 12:00; en el orden anterior, el par que decide H1 quedaba después del corte | Lo que no se ejecute antes del corte aparece en el Book como no disponible y en el capítulo de uso de IA como desviación |
| 11-25 18:06 | Ablaciones de segmentación con semilla 0 en el orden pérdidas → LOCTO → fracciones 10 y 50 % | Con las duraciones medidas, el trabajo pendiente termina el viernes y el corte del jueves deja fuera unas 2 de las 8 ablaciones s0; la curva de aprendizaje ya tiene los puntos de 25 % y 100 % | Las fracciones 10 y 50 % y las semillas 1–2 de todas las ablaciones probablemente quedan sin ejecutar; las ablaciones que sí corren tienen una semilla y se rotulan como exploratorias |
| 11-25 18:17 | Corte de entrenamientos movido al jueves a las 17:00. Orden nuevo después de la segmentación prioritaria: (1) las 8 ablaciones con la semilla 0; (2) semilla 1 de las ablaciones de pérdida y de LOCTO; (3) análisis secundarios caros de texto; (4) semilla 2. Las validaciones anidadas de XGBoost y FastText quedan fuera de la cola (corrige las filas de las 07:47 y 14:33). Regla de evidencia por semillas: con 1 semilla la ablación es exploratoria; con 2 informamos la diferencia de cada semilla y si su signo coincide, sin DE; con 3 o más, también la DE entre semillas (corrige la fila de las 07:48) | Cambio del plazo de entrega; con las duraciones medidas, las 8 ablaciones s0 y 6 ablaciones s1 caben antes del corte | Las fracciones 10 y 50 % quedan con una semilla; pérdidas y LOCTO, con dos si la cola cumple la estimación. XGBoost y FastText quedan sin corrección del sesgo optimista |
| 11-26 05:50 | Implementación del criterio de H1: la primera parte se refuta si el IC 95 % de Mask R-CNN − U-Net++ al 25 % no queda entero por encima de 0 (el código anterior solo comprobaba si incluía 0) | H1 es direccional: un IC entero por debajo de 0 es evidencia en contra de «Mask R-CNN supera», no a favor | Aclara el criterio sin cambiarlo; detectado al leer el resultado y corregido antes de redactar el capítulo de resultados |
