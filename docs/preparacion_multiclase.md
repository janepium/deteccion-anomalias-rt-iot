# Preparación del Pipeline Multiclase — RT_IOT2022

**Responsable:** Edwin  
**Dataset utilizado:** `data/processed/dataset_limpio.csv`  
**Notebook:** `notebooks/12_preparacion_multiclase.ipynb`  
**Semana:** 6 — Clasificación multiclase

---

# Objetivo

Adaptar el pipeline de la clasificación binaria (Normal vs. Ataque) para trabajar directamente con `Attack_type` como
variable objetivo (12 clases), manteniendo las mismas decisiones de preparación de datos, y dejar listo el conjunto
Train/Test que se utilizará para entrenar el Random Forest multiclase (`13_random_forest_multiclase.ipynb`).

---

# Cambios Realizados al Pipeline Binario

| Aspecto | Pipeline binario | Pipeline multiclase |
|---|---|---|
| Variable objetivo | `y_bin` (0 = Normal, 1 = Ataque) | `Attack_type` (12 clases, etiquetas de texto) |
| Estratificación | `stratify=y_bin` | `stratify=y` (12 clases) |
| Variables leaky | Excluidas | Excluidas (sin cambio) |
| Features | 32 numéricas | Las mismas 32 (sin cambio) |
| Escalamiento | No (árboles) | No (sin cambio) |
| Split | 80/20, `random_state=42` | 80/20, `random_state=42` (sin cambio) |
| Índice de filas | Índice original del CSV | `row_id` único (posición en `dataset_limpio.csv`) |
| Métricas recomendadas | F1, F1-Macro, PR-AUC | Accuracy, Precision/Recall macro, F1-Macro, F1-Weighted, métricas por clase |

### 1. Variable objetivo

En binario se construía:

```python
NORMAL_CLASSES = {"MQTT_Publish", "Thing_Speak", "Wipro_bulb"}
y_bin = (~df["Attack_type"].isin(NORMAL_CLASSES)).astype(int)
```

En multiclase se usa directamente:

```python
y = df["Attack_type"]
```

Las etiquetas se conservan como texto: Random Forest las acepta y la matriz de confusión queda legible.

### 2. Estratificación por las 12 clases

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.20, stratify=y, random_state=42
)
```

Con la estratificación binaria (`stratify=y_bin`) solo se garantizaba la proporción Normal/Ataque; las clases internas
quedaban repartidas al azar. Comparación del número de muestras en Test:

| Attack_type | Test esperado (20 %) | `stratify=y` | `stratify=y_bin` |
|---|---|---|---|
| DOS_SYN_Hping | 18.017,8 | 18.018 | 17.893 |
| Thing_Speak | 1.530,8 | 1.531 | 1.564 |
| ARP_poisioning | 1.525,0 | 1.525 | 1.542 |
| MQTT_Publish | 828,4 | 828 | 792 |
| NMAP_UDP_SCAN | 516,8 | 517 | 551 |
| NMAP_XMAS_TREE_SCAN | 402,0 | 402 | 451 |
| NMAP_OS_DETECTION | 400,0 | 400 | 409 |
| NMAP_TCP_scan | 200,4 | 200 | 210 |
| DDOS_Slowloris | 106,6 | 107 | 116 |
| Wipro_bulb | 43,8 | 44 | 47 |
| Metasploit_Brute_Force_SSH | 7,2 | 7 | 6 |
| NMAP_FIN_SCAN | 5,6 | 6 | 4 |

### 3. Índice único `row_id`

El índice guardado en `dataset_limpio.csv` proviene del CSV original, que **reinicia la numeración en cada tipo de
tráfico**, por lo que no es único (27.833 valores repetidos). Esto hacía que filas distintas de Train y Test compartieran
índice. Se reemplazó por `row_id` = posición de la fila en `dataset_limpio.csv` (0 … 117.921), que es el índice con el
que se exportan los archivos y el que permite rastrear cada flujo en el análisis de errores.

### 4. Métricas

La PR-AUC binaria deja de aplicar. Debido al desbalance, la Accuracy queda dominada por `DOS_SYN_Hping` (76,4 % de los
datos); deben priorizarse F1-Macro y las métricas por clase.

---

# Características Utilizadas

Se conservaron **exactamente las mismas 32 características** del Random Forest binario seleccionado
(`08_comparacion_modelos.ipynb`).

### Variables excluidas

| Variable | Motivo |
|---|---|
| `id.orig_p` | Leaky — puerto de origen |
| `id.resp_p` | Leaky — puerto de destino (identifica casi directamente el servicio/ataque) |
| `proto` | Leaky — protocolo |
| `service` | Leaky — servicio |
| `Attack_type` | Variable objetivo |

Las 46 variables redundantes y la variable constante ya habían sido eliminadas en `03_dataset_limpio.ipynb`
(clustering Spearman |ρ| ≥ 0,9, ver `docs/preprocesamiento_limpieza.md`); no se eliminó ninguna variable adicional.

### Variables conservadas (32)

`flow_duration`, `fwd_pkts_tot`, `bwd_pkts_tot`, `fwd_data_pkts_tot`, `flow_pkts_per_sec`, `down_up_ratio`,
`fwd_header_size_max`, `bwd_header_size_max`, `flow_FIN_flag_count`, `flow_SYN_flag_count`, `flow_RST_flag_count`,
`fwd_PSH_flag_count`, `flow_ACK_flag_count`, `fwd_URG_flag_count`, `flow_CWR_flag_count`, `fwd_pkts_payload.avg`,
`bwd_pkts_payload.min`, `bwd_pkts_payload.avg`, `flow_pkts_payload.min`, `flow_pkts_payload.max`,
`flow_pkts_payload.tot`, `flow_pkts_payload.avg`, `flow_pkts_payload.std`, `fwd_iat.std`, `fwd_subflow_bytes`,
`fwd_bulk_bytes`, `active.std`, `idle.tot`, `idle.std`, `fwd_init_window_size`, `bwd_init_window_size`,
`fwd_last_window_size`.

**`flow_duration` se conserva**: el modelo binario de referencia la utiliza. El experimento sin ella
(`10_analisis_random_forest.ipynb`) fue un análisis de sensibilidad y no modificó el pipeline. Así Daniel puede
comparar su importancia en multiclase con la obtenida en binario.

---

# Distribución de las 12 Clases en Train y Test

Split 80/20 estratificado: **Train = 94.337 filas**, **Test = 23.585 filas**.

| Attack_type | Total | Train | Test | % Total | % Train | % Test |
|---|---|---|---|---|---|---|
| DOS_SYN_Hping | 90.089 | 72.071 | 18.018 | 76,397 | 76,397 | 76,396 |
| Thing_Speak | 7.654 | 6.123 | 1.531 | 6,491 | 6,491 | 6,491 |
| ARP_poisioning | 7.625 | 6.100 | 1.525 | 6,466 | 6,466 | 6,466 |
| MQTT_Publish | 4.142 | 3.314 | 828 | 3,512 | 3,513 | 3,511 |
| NMAP_UDP_SCAN | 2.584 | 2.067 | 517 | 2,191 | 2,191 | 2,192 |
| NMAP_XMAS_TREE_SCAN | 2.010 | 1.608 | 402 | 1,705 | 1,705 | 1,704 |
| NMAP_OS_DETECTION | 2.000 | 1.600 | 400 | 1,696 | 1,696 | 1,696 |
| NMAP_TCP_scan | 1.002 | 802 | 200 | 0,850 | 0,850 | 0,848 |
| DDOS_Slowloris | 533 | 426 | 107 | 0,452 | 0,452 | 0,454 |
| Wipro_bulb | 219 | 175 | 44 | 0,186 | 0,186 | 0,187 |
| Metasploit_Brute_Force_SSH | 36 | 29 | 7 | 0,031 | 0,031 | 0,030 |
| NMAP_FIN_SCAN | 28 | 22 | 6 | 0,024 | 0,023 | 0,025 |

- Las **12 clases están presentes** en Train y Test (verificado con `assert`).
- La diferencia máxima de proporción entre Train y Test es de **0,002 puntos porcentuales**.
- Train y Test no comparten ningún `row_id`.

---

# Dificultades Encontradas

## 1. Desbalance extremo

- `DOS_SYN_Hping` concentra el **76,4 %** de los datos; la clase minoritaria `NMAP_FIN_SCAN` solo tiene 28 registros.
- **Ratio de desbalance: 3.217 : 1.**
- Un modelo que siempre predijera `DOS_SYN_Hping` tendría 76 % de Accuracy, por lo que esta métrica es poco informativa.

## 2. Clases con muy pocas muestras en Test

| Attack_type | n_test | Impacto de 1 error en el recall |
|---|---|---|
| Wipro_bulb | 44 | 2,3 pp |
| Metasploit_Brute_Force_SSH | 7 | 14,3 pp |
| NMAP_FIN_SCAN | 6 | 16,7 pp |

Las métricas individuales de `Metasploit_Brute_Force_SSH` y `NMAP_FIN_SCAN` serán **muy inestables**: un solo error
mueve el recall más de 14 puntos. Además, al tener tan pocos datos, el F1-Macro (promedio simple de las 12 clases) puede
variar bastante por pocos errores en estas clases.

## 3. Duplicados al eliminar las variables leaky

`dataset_limpio.csv` no tiene filas duplicadas considerando todas sus columnas, pero al eliminar puertos, protocolo y
servicio **100.034 de 117.922 filas (84,8 %) quedan duplicadas** en las 32 features.

| Attack_type | Registros | Filas únicas en X | % duplicado |
|---|---|---|---|
| DOS_SYN_Hping | 90.089 | 57 | 99,9 |
| NMAP_OS_DETECTION | 2.000 | 75 | 96,2 |
| NMAP_XMAS_TREE_SCAN | 2.010 | 80 | 96,0 |
| NMAP_TCP_scan | 1.002 | 44 | 95,6 |
| NMAP_UDP_SCAN | 2.584 | 193 | 92,5 |
| NMAP_FIN_SCAN | 28 | 5 | 82,1 |
| ARP_poisioning | 7.625 | 5.035 | 34,0 |
| Thing_Speak | 7.654 | 7.487 | 2,2 |
| MQTT_Publish, DDOS_Slowloris, Wipro_bulb, Metasploit | — | — | 0,0 |

Consecuencia: **20.071 de las 23.585 filas de Test (85,1 %) tienen una copia idéntica en Train**. En `DOS_SYN_Hping`
y `NMAP_FIN_SCAN` es el 100 % y en los escaneos NMAP (TCP, XMAS, OS) más del 96 %. Las métricas de esas clases serán
optimistas, porque el modelo "memoriza" flujos que vuelve a ver en Test.

**Decisión:** se mantuvieron las 117.922 filas para conservar la comparabilidad directa con los resultados binarios
(obtenidos con este mismo dataset). Se recomienda evaluar en una semana posterior una variante deduplicada o un split
por grupos (`GroupShuffleSplit` sobre la combinación de features).

## 4. Flujos idénticos con etiquetas distintas

Existen **14 combinaciones exactas de features que aparecen con más de una etiqueta** (3.809 filas afectadas).
Ningún modelo puede clasificar correctamente todas esas filas.

| Attack_type | Filas en combinaciones ambiguas | % de la clase |
|---|---|---|
| NMAP_UDP_SCAN | 2.235 | 86,5 |
| ARP_poisioning | 1.559 | 20,4 |
| Thing_Speak | 13 | 0,2 |
| Metasploit_Brute_Force_SSH | 1 | 2,8 |
| DDOS_Slowloris | 1 | 0,2 |

Pares de clases que comparten flujos idénticos: `ARP_poisioning / NMAP_UDP_SCAN` (5 combinaciones),
`ARP_poisioning / Thing_Speak` (3), `ARP_poisioning / NMAP_UDP_SCAN / Thing_Speak` (2), `NMAP_UDP_SCAN / Thing_Speak` (2),
`Metasploit_Brute_Force_SSH / NMAP_UDP_SCAN` (1) y `ARP_poisioning / DDOS_Slowloris / Thing_Speak` (1).

Esto **anticipa confusiones entre `ARP_poisioning`, `NMAP_UDP_SCAN` y `Thing_Speak`** en el modelo multiclase y es
coherente con que `ARP_poisioning` y `NMAP_UDP_SCAN` concentraran los falsos negativos en la clasificación binaria
(`docs/analisis_errores.md`). Una hipótesis plausible es que ARP poisoning opera en capa 2 y el dataset solo describe
flujos de capa 3/4, por lo que algunos de sus flujos son indistinguibles de otro tráfico.

## 5. Índice no único

Descrito arriba: el índice original se repite entre clases. Se resolvió con `row_id`.

---

# Respuestas a las Preguntas del Enunciado

## ¿Qué cambios fue necesario realizar al pipeline binario para adaptarlo a la clasificación multiclase?

1. Reemplazar la variable objetivo binaria `y_bin` por `Attack_type` directamente (12 clases como texto).
2. Estratificar el split por las 12 clases (`stratify=y`) en lugar de por `y_bin`, para que las clases minoritarias
   conserven su proporción en Test (por ejemplo `NMAP_FIN_SCAN` pasa de 4 a 6 muestras en Test).
3. Crear un índice único `row_id`, ya que el índice original de `dataset_limpio.csv` se repite entre clases.
4. Cambiar las métricas de evaluación recomendadas a métricas macro/weighted y por clase (PR-AUC binaria ya no aplica).

El resto del pipeline (dataset, exclusión de variables leaky, features, ausencia de escalamiento, proporción 80/20 y
`random_state=42`) se mantuvo sin cambios.

## ¿Se conservaron las mismas características utilizadas en la clasificación binaria? ¿Cuáles se excluyeron y por qué?

Sí, se conservaron las **mismas 32 características** del Random Forest binario seleccionado, incluida `flow_duration`.
Se excluyeron `id.orig_p`, `id.resp_p`, `proto` y `service` por ser variables leaky (identifican el tipo de tráfico de
forma casi directa y producirían resultados artificialmente optimistas), además de `Attack_type` por ser la variable
objetivo. Las variables redundantes ya habían sido eliminadas en la etapa de limpieza.

## ¿Con qué distribución están representadas las 12 clases en los conjuntos de entrenamiento y prueba?

Con la misma proporción que en el dataset completo (diferencia máxima de 0,002 pp entre Train y Test). Train tiene
94.337 filas y Test 23.585. La distribución va desde `DOS_SYN_Hping` (72.071 en Train / 18.018 en Test, 76,4 %) hasta
`NMAP_FIN_SCAN` (22 / 6, 0,024 %). Las 12 clases están presentes en ambos conjuntos (ver tabla de distribución).

## ¿Qué dificultades surgieron al preparar el conjunto de datos multiclase?

1. **Desbalance extremo** (3.217 : 1), con una clase que acapara el 76,4 % de los datos.
2. **Clases con muy pocas muestras en Test** (`Metasploit_Brute_Force_SSH` = 7, `NMAP_FIN_SCAN` = 6): sus métricas
   individuales serán inestables.
3. **Duplicados masivos al quitar las variables leaky** (84,8 % de las filas): el 85,1 % del Test tiene una copia
   idéntica en Train, lo que infla las métricas, sobre todo en `DOS_SYN_Hping` y los escaneos NMAP.
4. **Flujos idénticos con etiquetas distintas** (14 combinaciones, 3.809 filas), concentrados en `NMAP_UDP_SCAN` (86,5 %
   de la clase) y `ARP_poisioning` (20,4 %): imponen un techo al desempeño en esas clases.
5. **Índice no único** en `dataset_limpio.csv`, resuelto con `row_id`.

---

# Archivos Generados

Carpeta `data/processed/multiclase/`:

| Archivo | Contenido |
|---|---|
| `X_train.csv` | 94.337 × 32 features, índice `row_id` |
| `X_test.csv` | 23.585 × 32 features, índice `row_id` |
| `y_train.csv` | `Attack_type` de Train, índice `row_id` |
| `y_test.csv` | `Attack_type` de Test, índice `row_id` |
| `feature_list.json` | Lista ordenada de features, clases, variables excluidas y parámetros del split |

Carga recomendada para `13_random_forest_multiclase.ipynb`:

```python
import pandas as pd

DIR = "../data/processed/multiclase"
X_train = pd.read_csv(f"{DIR}/X_train.csv", index_col=0)
X_test = pd.read_csv(f"{DIR}/X_test.csv", index_col=0)
y_train = pd.read_csv(f"{DIR}/y_train.csv", index_col=0)["Attack_type"]
y_test = pd.read_csv(f"{DIR}/y_test.csv", index_col=0)["Attack_type"]
```

Para el análisis de errores, `row_id` permite recuperar la fila original con
`pd.read_csv("../data/processed/dataset_limpio.csv", index_col=0).reset_index(drop=True).loc[row_id]`.

---

# Recomendaciones para Daniel y Carlos

- Usar la configuración de referencia `RandomForestClassifier(n_estimators=100, class_weight="balanced", random_state=42, n_jobs=-1)`;
  `class_weight="balanced"` es aún más relevante en multiclase por el desbalance.
- Priorizar **F1-Macro** y las métricas por clase; no apoyarse en Accuracy ni en F1-Weighted, dominadas por `DOS_SYN_Hping`.
- Interpretar con cautela los resultados de `Metasploit_Brute_Force_SSH` y `NMAP_FIN_SCAN` (≤ 7 muestras en Test).
- Prestar atención a las confusiones entre `ARP_poisioning`, `NMAP_UDP_SCAN` y `Thing_Speak`, anticipadas por los flujos
  con features idénticas.
- Tener presente que las métricas casi perfectas en `DOS_SYN_Hping` y los escaneos NMAP están favorecidas por los duplicados
  entre Train y Test.

---

# Conclusiones

El pipeline binario se adaptó a la clasificación multiclase con cambios mínimos: la variable objetivo pasa a ser
`Attack_type` y la estratificación se hace sobre las 12 clases, manteniendo el resto de decisiones (exclusión de variables
leaky, mismas 32 features, sin escalamiento, split 80/20 con `random_state=42`). Las 12 clases quedan representadas en
Train y Test con proporciones prácticamente idénticas.

La preparación reveló tres problemas que condicionarán la interpretación del modelo multiclase: el desbalance extremo
(3.217 : 1), la presencia masiva de duplicados que se reparten entre Train y Test (85 % del Test) y la existencia de flujos
idénticos con distinta etiqueta, que afectan principalmente a `NMAP_UDP_SCAN` y `ARP_poisioning`. Se mantuvo el dataset
completo para asegurar la comparabilidad con la clasificación binaria, y se recomienda evaluar más adelante una variante
deduplicada para obtener una estimación más realista del desempeño.
