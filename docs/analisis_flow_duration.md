# Análisis de `flow_duration` en el baseline Decision Tree

**Responsable:** Edwin.
**Notebook:** `notebooks/09_analisis_flow_duration.ipynb`
**Modelo guardado:** `models/decision_tree_sin_flow_duration.pkl` (experimento de trazabilidad;
no reemplaza al baseline oficial `models/decision_tree_baseline.pkl`).

## Objetivo

En `docs/baseline_decision_tree.md` se documentó que `flow_duration` concentra ~91% de la
importancia del Decision Tree binario, señalándolo como "una observación a vigilar" sin
investigarla a fondo. Este análisis cierra ese pendiente: caracteriza `flow_duration` en detalle,
verifica si el modelo realmente aprende de múltiples características del tráfico o depende casi
exclusivamente de esta variable, y evalúa el riesgo de que el resultado esté demasiado
condicionado por ella antes de que el equipo optimice Random Forest (Daniel) y audite errores por
`Attack_type` (Carlos).

## Datos y variable objetivo

- Archivo: `data/processed/dataset_limpio.csv` (117.922 filas, 37 columnas, sin nulos).
- **`y_bin`** (0 = Normal, 1 = Ataque) se reconstruye con la misma fórmula oficial que en
  `07_baseline_decision_tree.ipynb`:
  ```python
  NORMAL_CLASSES = {"MQTT_Publish", "Thing_Speak", "Wipro_bulb"}
  y_bin = (~df["Attack_type"].isin(NORMAL_CLASSES)).astype(int)
  ```
  Distribución: **89.81% Ataque (105.907 filas) / 10.19% Normal (12.015 filas)**.
- `Attack_type` se usa **únicamente para análisis** (agrupaciones y gráficas); nunca entra como
  feature del modelo.
- Mismo split que 07: 80/20 estratificado sobre `y_bin`, `random_state=42` — mismo `X_test`
  (23.585 filas), directamente comparable fila a fila con el baseline oficial.

## Análisis de `flow_duration`

### Estadísticos globales y por clase

| | count | mean | median | std | min | max |
|---|---:|---:|---:|---:|---:|---:|
| Global | 117.922 | 3.8085 | 0.000004 | 127.06 | 0 | 21728.34 |
| Normal | 12.015 | 26.3049 | 0.8882 | 387.36 | 0 | 21728.34 |
| Ataque | 105.907 | 1.2563 | 0.000004 | 29.86 | 0 | 5341.39 |

- Proporción de `flow_duration == 0`: **12.90% global**, pero muy distinta por clase —
  **0.73% en Normal vs. 14.28% en Ataque**.
- Por `Attack_type` (ordenado por media descendente), el patrón es claro: `Wipro_bulb` (media
  587.71 s, muy afectada por unos pocos flujos larguísimos), `MQTT_Publish` (43.44 s) y
  `ARP_poisioning` (16.15 s, pero mediana 0.0007 s — también muy sesgada) tienen las duraciones
  más largas. En el extremo opuesto, **`DOS_SYN_Hping` (90.089 filas, 85.1% de todo "Ataque" y
  76.4% del dataset completo) tiene duración prácticamente cero** (media y mediana ≈0), igual que
  los cinco tipos de escaneo NMAP (`NMAP_OS_DETECTION`, `NMAP_TCP_scan`, `NMAP_XMAS_TREE_SCAN`,
  `NMAP_FIN_SCAN`, `NMAP_UDP_SCAN`; 7.624 filas en conjunto). En total, **~92.3% de las filas de
  Ataque (97.713 de 105.907) tienen `flow_duration` esencialmente nulo**.

### Outliers y cola extrema

- Una regla clásica de IQR (1.5·IQR sobre Q3) es poco informativa aquí: Q1=0.000001, Q3=0.000005,
  IQR=0.000004 → límite superior ≈0.000011, con lo que **15.93% de las filas (18.783) quedan
  marcadas como "outlier"** solo por ser mayores a unos microsegundos. Se usa en su lugar un
  análisis por cuantiles.
- Tabla de cuantiles (Normal vs. Ataque vs. global):

  | Cuantil | Normal | Ataque | Global |
  |---|---:|---:|---:|
  | 0.50 | 0.8882 | 0.0000 | 0.0000 |
  | 0.75 | 31.9645 | 0.0000 | 0.0000 |
  | 0.90 | 61.9689 | 0.0000 | 0.0642 |
  | 0.95 | 62.0505 | 0.0002 | 4.8494 |
  | 0.99 | 62.1442 | 20.1236 | 62.0473 |
  | 0.999 | 653.4027 | 173.4828 | 194.1938 |
  | 1.000 | 21728.3356 | 5341.3923 | 21728.3356 |

- La cola extrema (`flow_duration` > P99 = 62.05 s, 1.180 filas) mezcla ambas clases: 600
  `MQTT_Publish` + 19 `Thing_Speak` (Normal) junto con 527 `ARP_poisioning` + 31 `Wipro_bulb`
  (Normal) + 3 `NMAP_UDP_SCAN` (Ataque) — la evidencia de que la separación no es perfecta en los
  extremos.

### ¿Las duraciones separan limpiamente las clases?

- **P90(Ataque) = 0.000006 s << P10(Normal) = 0.0267 s**: el 90% "más corto" de los ataques cae
  muy por debajo del 90% "más largo" de los normales — separación casi total en el grueso de la
  distribución.
- Usando `flow_duration` como única variable: **ROC-AUC = 0.9739, PR-AUC = 0.9959**. La mejor
  regla de un solo umbral (`flow_duration ≤ 0.00277 s → Ataque`) logra **F1-macro = 0.9150** —
  fuerte, pero 8 puntos por debajo del árbol completo (F1-macro 0.9952), confirmando que
  `flow_duration` sola no basta para igualar al modelo completo.

## Metodología

- Se generaliza `build_preprocessor(include_leaky: bool)` de `07_baseline_decision_tree.ipynb` a
  una versión con `exclude_cols`, porque aquí se necesita quitar una columna numérica específica
  (`flow_duration`), no un bloque leaky completo. Ambas variantes comparadas son "sin leaky"
  (mismo alcance recomendado en 07); la única diferencia entre ellas es la presencia o ausencia de
  `flow_duration`.
- Mismos hiperparámetros del árbol que el baseline inicial de 07: `DecisionTreeClassifier(class_weight="balanced", random_state=42)`, sin restricción de profundidad.
- Mismo split (80/20, `stratify=y_bin`, `random_state=42`) y misma validación cruzada
  (`StratifiedKFold(n_splits=5, shuffle=True, random_state=42)` + `cross_validate`) que 07.

## Resultados

### Comparación con vs. sin `flow_duration` (test reservado, 23.585 filas)

| Variante | Accuracy | Precision | Recall | F1 | F1-macro | PR-AUC | CV F1-macro | Fit (s) | Predict (s) | Profundidad | Hojas |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Con `flow_duration` | 0.9983 | 0.9989 | 0.9992 | 0.9990 | 0.9952 | 0.9988 | 0.9951 ± 0.0014 | 0.418 | 0.0146 | 23 | 207 |
| Sin `flow_duration` | 0.9982 | 0.9988 | 0.9992 | 0.9990 | 0.9950 | 0.9988 | 0.9954 ± 0.0007 | 0.307 | 0.0135 | 19 | 228 |
| **Delta (sin − con)** | −0.0001 | −0.0000 | −0.0000 | −0.0000 | **−0.0002** | −0.0000 | +0.0003 | −0.111 | −0.0011 | −4 | +21 |

### Matrices de confusión (test)

| Con `flow_duration` | Predicho Normal | Predicho Ataque |
|---|---:|---:|
| **Real Normal** | 2.379 (TN) | 24 (FP) |
| **Real Ataque** | 17 (FN) | 21.165 (TP) |

| Sin `flow_duration` | Predicho Normal | Predicho Ataque |
|---|---:|---:|
| **Real Normal** | 2.378 (TN) | 25 (FP) |
| **Real Ataque** | 18 (FN) | 21.164 (TP) |

Un error adicional de cada tipo sobre 23.585 filas de test.

### Diagnóstico de sobreajuste (F1-macro)

| Variante | Train (CV) | Validación (CV) | Test | Brecha train–CV |
|---|---:|---:|---:|---:|
| Con `flow_duration` | 0.9997 | 0.9951 | 0.9952 | 0.0046 |
| Sin `flow_duration` | 0.9997 | 0.9954 | 0.9950 | 0.0042 |

Sin señales de sobreajuste adicional al quitar la variable; la brecha incluso se reduce
levemente.

### Importancia de variables: antes vs. después

Top 5 con `flow_duration`: `flow_duration` (90.94%), `flow_pkts_payload.min` (1.87%),
`bwd_init_window_size` (1.62%), `bwd_pkts_tot` (1.15%), `fwd_subflow_bytes` (1.03%).

Top 5 sin `flow_duration`: **`bwd_pkts_payload.avg` (87.35%)**, `flow_pkts_per_sec` (3.12%),
`fwd_data_pkts_tot` (1.57%), `flow_pkts_payload.avg` (1.49%), `fwd_subflow_bytes` (1.18%).

Tabla de variables que más importancia ganan (`sin − con`):

| Variable | Con `flow_duration` | Sin `flow_duration` | Delta |
|---|---:|---:|---:|
| `bwd_pkts_payload.avg` | 0.0006 | 0.8735 | **+0.8729** |
| `flow_pkts_per_sec` | 0.0008 | 0.0312 | +0.0304 |
| `fwd_data_pkts_tot` | 0.0000 | 0.0157 | +0.0157 |
| `flow_pkts_payload.avg` | 0.0012 | 0.0149 | +0.0137 |
| `fwd_last_window_size` | 0.0004 | 0.0088 | +0.0084 |
| `fwd_pkts_tot` | 0.0015 | 0.0055 | +0.0041 |
| `flow_pkts_payload.tot` | 0.0007 | 0.0030 | +0.0023 |
| `fwd_pkts_payload.avg` | 0.0012 | 0.0030 | +0.0017 |
| `bwd_pkts_payload.min` | 0.0018 | 0.0035 | +0.0017 |
| `fwd_subflow_bytes` | 0.0103 | 0.0118 | +0.0015 |

`bwd_pkts_payload.avg` tiene correlación de Pearson de solo **0.0274** con `flow_duration` —
señal genuinamente distinta, no un proxy. Sus estadísticos por clase repiten el mismo patrón:
mediana 52.0 / media 93.72 en Normal vs. **mediana 0.0** / media 10.83 en Ataque — tráfico de
ataque que no recibe payload de vuelta (flujos de una sola vía) frente a sesiones IoT legítimas
con payload bidireccional real.

## Respuestas

### ¿Por qué `flow_duration` tiene tanta importancia?

Porque separa casi perfectamente el grueso de las dos clases en un solo corte: ~92.3% de las
filas de Ataque (`DOS_SYN_Hping` + los 5 tipos de escaneo NMAP) tienen duración esencialmente
nula, mientras que el tráfico Normal corresponde a sesiones sostenidas (mediana 0.888 s, P75 ≈32
s). Un árbol de Gini sin restricción de profundidad encuentra ese punto de corte (~2.77 ms) casi
de inmediato y le asigna casi todo el presupuesto de importancia.

### ¿La diferencia entre Normal y Ataque puede explicarse principalmente por esta variable?

En gran parte, pero no del todo. Sola, `flow_duration` logra ROC-AUC 0.9739, PR-AUC 0.9959 y
F1-macro 0.9150 con el mejor umbral único — fuerte pero 8 puntos por debajo del árbol completo
(0.9952). La cola extrema (>P99, 1.180 filas) mezcla Normal (`MQTT_Publish`, `Thing_Speak`,
`Wipro_bulb`) y Ataque (`ARP_poisioning`, `NMAP_UDP_SCAN`): esa zona requiere otras variables.

### ¿Qué ocurre con el desempeño al eliminarla?

Casi nada cambia: F1-macro pasa de 0.9952 a 0.9950 (−0.0002), PR-AUC se mantiene en 0.9988, y la
matriz de confusión suma solo un FN y un FP adicionales sobre 23.585 filas de test. En CV el
F1-macro incluso mejora levemente (0.9951→0.9954) con menor varianza.

### ¿El modelo sigue funcionando adecuadamente?

Sí. La brecha train–CV no crece (0.0046→0.0042); no hay señales de que el árbol dependa de un
atajo frágil que se rompa al quitar `flow_duration`.

### ¿Existe riesgo de que el resultado esté demasiado condicionado por esta característica?

A nivel de desempeño, el riesgo es bajo: las métricas apenas se mueven al quitarla. A nivel de
interpretación, el riesgo es real: al remover `flow_duration`, `bwd_pkts_payload.avg` salta de
0.06% a 87.35% de importancia (correlación con `flow_duration` de solo 0.0274 — no es un proxy,
es una señal distinta que codifica la misma idea: "flujo corto/unidireccional = ataque" vs.
"sesión sostenida/bidireccional = normal"). El árbol siempre concentra ~87–91% de su importancia
en una sola variable porque este dataset contiene varias señales casi-univariadas que separan
casi perfectamente esta mezcla particular de ataques (mayormente floods y escaneos automatizados
de un solo paquete). El riesgo a vigilar no es que el modelo "solo sepa de `flow_duration`", sino
que el ~99.5% de desempeño puede no generalizar a ataques que no compartan esta firma de flujo
corto / sin payload de retorno.

### ¿Qué variables adquieren importancia después de eliminarla?

En orden de ganancia: `bwd_pkts_payload.avg` (+0.8729), `flow_pkts_per_sec` (+0.0304),
`fwd_data_pkts_tot` (+0.0157), `flow_pkts_payload.avg` (+0.0137), `fwd_last_window_size`
(+0.0084), `fwd_pkts_tot` (+0.0041), `flow_pkts_payload.tot` (+0.0023), `fwd_pkts_payload.avg`
(+0.0017), `bwd_pkts_payload.min` (+0.0017) y `fwd_subflow_bytes` (+0.0015). Casi todas describen
volumen/dirección de payload y tasa de paquetes — consistentes con la misma hipótesis de "flujo
corto y unidireccional" vs. "sesión sostenida y bidireccional".

## Nota sobre trabajo relacionado del equipo

Este análisis extiende directamente `notebooks/07_baseline_decision_tree.ipynb` /
`docs/baseline_decision_tree.md` (mismo autor, mismo protocolo, mismo split) y responde al
pendiente que ese documento dejaba abierto sobre la concentración de importancia en
`flow_duration`. La conclusión (el riesgo es de interpretación, no de desempeño; existen varias
señales casi-univariadas equivalentes) es relevante para las tareas paralelas de la semana:
Daniel (optimización de Random Forest) debería verificar si el mismo patrón aparece al usar un
ensamble, y Carlos (análisis de errores por `Attack_type`) puede usar la lista de `Attack_type`
con duración casi nula (`DOS_SYN_Hping`, escaneos NMAP) para interpretar mejor dónde se concentran
los aciertos y errores del modelo binario.

## Artefactos generados

- `notebooks/09_analisis_flow_duration.ipynb` — notebook completo (estadística descriptiva de
  `flow_duration`, distribuciones, outliers, separación de clases, comparación con/sin
  `flow_duration`, importancia de variables, diagnóstico de sobreajuste).
- `models/decision_tree_sin_flow_duration.pkl` — pipeline (`ColumnTransformer` + `DecisionTreeClassifier`) de la variante sin `flow_duration`, guardado solo para trazabilidad del experimento.
- `docs/analisis_flow_duration.md` — este documento.
