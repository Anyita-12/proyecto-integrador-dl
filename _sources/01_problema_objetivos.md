# Problema, objetivos y preguntas de investigación

En este capítulo definimos formalmente los dos problemas, fijamos el objetivo general y los objetivos específicos, y relacionamos cada pregunta de investigación con el capítulo que la responde.

## Dos problemas con un mismo protocolo

### Segmentación de instancias celulares

La segmentación de células en imágenes de microscopía es el paso previo a medir su número, forma y crecimiento en cultivos. La competencia Sartorius {cite:p}`sartorius2021` plantea este problema para neuronas y células gliales en contraste de fase, donde las células tienen poco contraste con el fondo, se tocan entre sí y varían mucho de forma entre tipos celulares. El conjunto LIVECell {cite:p}`edlund2021livecell` es la referencia de contexto para esta modalidad.

**Formalización.** Una imagen es $x \in \{0,\dots,255\}^{520\times 704}$. La salida es un conjunto de máscaras binarias disjuntas $\{M_1,\dots,M_K\}$, una por célula, con $K$ desconocido. Un modelo es bueno si, para cada célula de referencia, produce exactamente una máscara con IoU alto y no produce máscaras sin célula. La métrica de la competencia, que adoptamos como primaria, promedia sobre umbrales de IoU $t\in\{0.50, 0.55, \dots, 0.95\}$ la razón $\text{TP}/(\text{TP}+\text{FP}+\text{FN})$ por imagen (AP@[0.5:0.95]).

Dos rasgos del problema guían el diseño:

- **Separar células que se tocan.** El 73 % de las células toca a otra (91 % en astrocitos). Una segmentación semántica buena (primer plano frente a fondo) puede tener un AP de instancia bajo si fusiona células vecinas.
- **Escala heterogénea.** Las neuronas corticales y SH-SY5Y son pequeñas y numerosas; los astrocitos son unas tres veces más grandes y tienen formas irregulares.

### Clasificación de hojas de vida

La competencia Jarvis Calling Hiring Contest {cite:p}`jarvis_kaggle` ofrece 2484 currículos en inglés, cada uno con una de 24 categorías profesionales. Clasificar un currículo por categoría es una tarea de enrutamiento: dirigir el documento a la bolsa de empleo o al reclutador correspondiente. Como ocurre en un contexto de selección de personal, un error sistemático contra un grupo de candidatos tiene consecuencias para personas reales, y por eso el análisis de equidad forma parte de la evaluación.

**Formalización.** Un documento es una secuencia de *tokens* $w_{1:T}$, con $T$ variable (mediana de 757 palabras). La salida es una distribución $\hat p(y\mid w_{1:T})$ sobre $y\in\{1,\dots,24\}$. La métrica primaria es la F1 macro, que da el mismo peso a las 24 clases y no oculta un mal desempeño en las clases pequeñas (BPO tiene 22 documentos en total).

Dos rasgos del problema guían el diseño:

- **Textos largos.** El 89.9 % de los currículos supera los 510 *tokens* que admite DistilBERT, lo que obliga a truncar.
- **Atajos léxicos.** En el 67.8 % de los documentos el nombre de la categoría aparece entre las 10 primeras palabras. Un modelo puede acertar leyendo solo el título, sin usar el resto del currículo.

## Objetivo general

Diseñar, implementar y comparar con validez estadística modelos de *deep learning* y clásicos para la segmentación de instancias celulares y la clasificación de currículos, y cuantificar no solo cuánto aciertan, sino cuándo, por qué, con qué incertidumbre, a qué costo y con qué riesgos fallan.

## Objetivos específicos

| ID | Objetivo | Dónde se cumple |
|---|---|---|
| O1 | Auditar los datos: sesgos, fugas, duplicados y ruido de anotación | Capítulos {doc}`02_datos_fichas`, {doc}`10_seg_eda` y {doc}`20_nlp_eda` |
| O2 | Implementar desde cero al menos un modelo por dominio, con pérdidas y métricas propias, y configurar los demás | U-Net++ con *watershed* y BiLSTM/CNN-1D en Keras; capítulos {doc}`12_seg_modelos` y {doc}`22_nlp_modelos` |
| O3 | Comparar modelos con un protocolo controlado e inferencia estadística | Capítulos {doc}`13_seg_resultados` y {doc}`23_nlp_resultados` |
| O4 | Medir con ablaciones qué componentes explican el rendimiento | Capítulos {doc}`14_seg_ablaciones_robustez` y {doc}`21_nlp_representaciones` |
| O5 | Evaluar robustez, calibración, incertidumbre y equidad | Capítulos {doc}`14_seg_ablaciones_robustez` y {doc}`24_nlp_robustez_equidad` |
| O6 | Empaquetar y desplegar el mejor modelo de cada dominio con pruebas, CI/CD y detección de deriva | Capítulos {doc}`30_mlops` y {doc}`31_monitoreo_deriva` |

## Preguntas de investigación

| ID | Pregunta | Capítulo |
|---|---|---|
| RQ1 | ¿Cómo varía el desempeño de los paradigmas de segmentación según tipo celular, densidad y tamaño de objeto, y qué diferencias son estadística y prácticamente relevantes? | {doc}`13_seg_resultados` |
| RQ2 | ¿Cuál es la contribución de la función de pérdida y del preentrenamiento, y cómo escala el rendimiento con la fracción de datos de entrenamiento? | {doc}`14_seg_ablaciones_robustez` |
| RQ3 | ¿Son consistentes las métricas de píxel (Dice, AUC, *balanced accuracy*) con las de instancia (AP, PQ)? ¿Cuándo se contradicen? | {doc}`13_seg_resultados` |
| RQ4 | ¿Qué tan robustos y calibrados son los modelos ante un tipo celular no visto y ante perturbaciones de imagen? | {doc}`14_seg_ablaciones_robustez` |
| RQ5 | Con el mismo presupuesto de búsqueda de hiperparámetros y controlando la varianza entre semillas, ¿justifica el costo de un *transformer* su ganancia frente a modelos clásicos? | {doc}`23_nlp_resultados` |
| RQ6 | ¿Qué parte del rendimiento se debe a atajos léxicos o a ruido de etiquetas, y cómo cambia con perturbaciones controladas? | {doc}`24_nlp_robustez_equidad` |
| RQ7 | ¿Están calibradas las probabilidades y hay disparidades de desempeño entre subgrupos? | {doc}`24_nlp_robustez_equidad` |
| RQ8 | ¿Cómo se degrada el rendimiento ante deriva de datos, y con qué rapidez y potencia la detecta el monitoreo? | {doc}`31_monitoreo_deriva` |

Las hipótesis H1–H5, con su criterio de refutación numérico, están en el capítulo {doc}`03_declaracion_apriori`. Refutar una hipótesis con evidencia sólida es un resultado tan informativo como confirmarla.

## Alcance

Las conclusiones de este libro valen para las particiones descritas en el capítulo {doc}`02_datos_fichas`: imágenes de contraste de fase de los tres tipos celulares de Sartorius y currículos en inglés de las 24 categorías de Jarvis. No evaluamos los datos sin etiqueta de Sartorius (`train_semi_supervised/`, LIVECell) ni otras modalidades de microscopía, y no evaluamos el uso del clasificador de currículos para decidir contrataciones.
