# Baseline de clasificación — Decision Tree

**Responsable:** Edwin.
**Notebook:** `notebooks/07_baseline_decision_tree.ipynb`
**Modelo guardado:** `models/decision_tree_baseline.pkl`

## Objetivo

Entrenar el primer modelo baseline del proyecto para clasificar `Attack_type` (12 clases),
comparando una variante **con** variables potencialmente leaky y una variante **sin** ellas,
siguiendo el protocolo ya definido por el equipo en `reports/decisiones_preparacion.md`
(secciones H e I).

## Dataset de entrada

- Archivo: `data/processed/dataset_limpio.csv`
- Registros: 117.922 · Columnas: 37 (36 features + `Attack_type`)
- Sin nulos, sin columnas constantes (ya validado en `03_dataset_limpio.ipynb` / `05_validacion_variables.ipynb`)
- Variables leaky (`docs/validacion_variables.md`, `docs/encoding.md`): `id.orig_p`, `id.resp_p`, `proto`, `service`

## Metodología

- **Split:** 80/20 estratificado (`train_test_split(..., stratify=y, random_state=42)`), compartido por ambas variantes. Test reservado solo para el reporte final.
- **Preprocesamiento:** `build_preprocessor(include_leaky)` — implementa el toggle mencionado en `reports/decisiones_preparacion.md` (H1) y nunca antes construido. `include_leaky=True` aplica `OneHotEncoder` sobre `proto`/`service` y conserva `id.orig_p`/`id.resp_p`; `include_leaky=False` los excluye todos (32 features numéricas restantes). El encoder se ajusta solo con train, dentro de un `Pipeline` (orden anti-leakage, protocolo I3).
- **Modelo:** `DecisionTreeClassifier(class_weight="balanced", random_state=42)`, **sin límite de profundidad** — decisión deliberada: un baseline diagnóstico no debe podarse, porque eso ocultaría la señal de sobreajuste/leakage que el protocolo pide vigilar.
- **Validación cruzada:** `StratifiedKFold(n_splits=5, shuffle=True, random_state=42)` sobre train, con `cross_validate` (`return_train_score=True`) para accuracy, F1-macro/weighted, precision-macro, recall-macro.
- **Piso de referencia:** `DummyClassifier(strategy="most_frequent")`.
- **Tiempos:** medidos con `time.perf_counter()` en el `fit` final de cada variante, y por fold vía `cross_validate` (`fit_time`).

## Resultados

### Tabla comparativa (test reservado, 23.585 filas)

| Variante | Accuracy | F1-macro | F1-weighted | Recall-macro | PR-AUC-macro | CV F1-macro (train, 5-fold) | Fit time (s) | Profundidad | Hojas |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Dummy floor | 0.7640 | 0.0722 | 0.6617 | 0.0833 | — | — | 0.05 | — | — |
| **Con leaky** | 0.9975 | 0.9871 | 0.9975 | 0.9924 | 0.9864 | 0.9568 ± 0.0201 | 1.10 | 33 | 306 |
| **Sin leaky** (modelo guardado) | 0.9957 | 0.9712 | 0.9957 | 0.9825 | 0.9696 | 0.9333 ± 0.0090 | 0.64 | 24 | 360 |

La columna "CV F1-macro" es la medida más honesta de desempeño (no toca el test); las de test son sobre una sola partición.

### Diagnóstico de sobreajuste (train vs. CV vs. test, F1-macro)

| Variante | Train (CV) | Validación (CV) | Test | Brecha train–test |
|---|---:|---:|---:|---:|
| Con leaky | 0.9988 | 0.9568 | 0.9871 | 0.0114 |
| Sin leaky | 0.9942 | 0.9333 | 0.9712 | 0.0226 |

### Importancia de variables (top 5)

**Sin leaky:** `fwd_last_window_size` (22.5%), `fwd_PSH_flag_count` (13.6%), `flow_pkts_per_sec` (9.9%), `fwd_URG_flag_count` (9.1%), `flow_pkts_payload.max` (7.9%).

**Con leaky:** `fwd_last_window_size` (20.4%), `id.resp_p` (15.3%), `flow_pkts_per_sec` (9.8%), `fwd_URG_flag_count` (9.1%), `service_mqtt` (9.1%).

## Respuestas

### ¿Qué desempeño obtiene el baseline?

La variante sin leaky (modelo oficial guardado) da accuracy 0.9957 y F1-macro 0.9712 en test; la medida más confiable, F1-macro en CV, es **0.9333 ± 0.009**. Muy por encima del piso trivial (Dummy: accuracy 0.764 pero F1-macro 0.0722), confirmando que el modelo aprende señal real y que la accuracy por sí sola sería engañosa con este desbalance de clases.

### ¿Existe diferencia entre usar variables leaky y no usarlas?

Sí, moderada pero consistente: la variante con leaky mejora F1-macro en +1.6 pp (test) y +2.3 pp (CV), y `id.resp_p` entra como 2ª variable más importante (15.3%) en cuanto se incluye; los dummies de `service`/`proto` también aparecen en su top 10. Ninguna variante llega a F1≈1.0 exacto (la señal de alarma extrema de H2 no se dispara), pero la mejora sistemática + la alta importancia de `id.resp_p` confirman que esas columnas aportan información que un IDS real no tendría disponible. Se guarda como modelo oficial la variante **sin leaky**, siguiendo la recomendación de `docs/encoding.md`.

### ¿Qué variables fueron las más importantes?

`fwd_last_window_size` domina en ambas variantes (~20-22%). Le siguen features de conteo de flags TCP (`fwd_PSH_flag_count`, `fwd_URG_flag_count`) y de tasa/tamaño de payload (`flow_pkts_per_sec`, `flow_pkts_payload.max`) en la variante sin leaky; en la variante con leaky, `id.resp_p` y el dummy `service_mqtt` desplazan a algunas de esas.

### ¿El modelo presenta señales de sobreajuste?

Sí. El árbol, sin poda a propósito, alcanza profundidad 24–33 y 306–360 hojas. En CV, el F1-macro de train (0.994–0.999) supera al de validación (0.933–0.957) por 4-6 puntos porcentuales — sobreajuste típico de un árbol sin regularizar que memoriza el train. La brecha train-test es menor porque train y test comparten distribución similar; la CV es la comparación honesta y sí lo muestra. Próximo paso sugerido: limitar `max_depth`/`min_samples_leaf`, o probar un ensemble (Random Forest).

### ¿Qué dificultades aparecieron durante el entrenamiento?

- El entorno no tenía `scikit-learn`, `joblib`, `matplotlib` ni `seaborn` instalados; hubo que instalarlos (`scikit-learn` 1.9.0, que actualizó `numpy` de 1.26.4 a 2.5.1).
- Clases extremadamente raras (`NMAP_FIN_SCAN`=28 filas, `Metasploit_Brute_Force_SSH`=36) quedan con muy pocas muestras por fold en el 5-fold CV (~5-6 en train por fold); `StratifiedKFold` no falla, pero el resultado en esas clases es más inestable.
- `cross_validate(..., n_jobs=-1)` en Windows generó ruido inofensivo del *resource tracker* de `joblib`/`loky` al finalizar (no afectó el resultado).
- Un árbol sin límite de profundidad es sensible a la partición exacta de los datos, lo que se refleja directamente en la brecha train/CV documentada arriba.

## Artefactos generados

- `notebooks/07_baseline_decision_tree.ipynb` — notebook completo (carga, preprocesamiento, entrenamiento, CV, evaluación, importancia de variables, guardado del modelo).
- `models/decision_tree_baseline.pkl` — pipeline (`ColumnTransformer` + `DecisionTreeClassifier`) de la variante **sin leaky**, entrenado sobre el 80% de train. Verificado con `joblib.load(...)`, reproduce accuracy = 0.9957 en el mismo test.
- `docs/baseline_decision_tree.md` — este documento.
