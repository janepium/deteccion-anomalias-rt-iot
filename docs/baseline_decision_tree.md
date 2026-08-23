# Baseline oficial de clasificación binaria — Decision Tree

**Responsable:** Edwin.
**Notebook:** `notebooks/07_baseline_decision_tree.ipynb`
**Modelo guardado:** `models/decision_tree_baseline.pkl`

## Objetivo

Construir el **baseline oficial del proyecto** para clasificación binaria (Normal/Ataque),
comparando una variante **con** variables potencialmente leaky y una variante **sin** ellas,
y explorando una búsqueda de hiperparámetros sobre el árbol recomendado. Reemplaza al baseline
multiclase inicial (`docs/baseline_decision_tree_old.md`), tras la decisión del equipo de fijar
la clasificación binaria como el problema oficial.

## Datos y variable objetivo

- Archivo: `data/processed/dataset_limpio.csv` (117.922 filas, 37 columnas, sin nulos).
- **`y_bin`** (0 = Normal, 1 = Ataque) no está materializada en ningún CSV — se construye en
  este notebook siguiendo la fórmula oficial documentada en `reports/decisiones_preparacion.md`
  (C1) y `docs/conceptos_proyecto.md`:
  ```python
  NORMAL_CLASSES = {"MQTT_Publish", "Thing_Speak", "Wipro_bulb"}
  y_bin = (~df["Attack_type"].isin(NORMAL_CLASSES)).astype(int)
  ```
- Distribución verificada explícitamente: **89.81% Ataque (105.907 filas) / 10.19% Normal (12.015 filas)**.
- `Attack_type` se conserva en el DataFrame solo para trazabilidad; no se usa como feature ni como target.
- Variables leaky (mismas que en el baseline multiclase): `id.orig_p`, `id.resp_p`, `proto`, `service`.

## Metodología

- **Split:** 80/20 (`train_test_split(..., stratify=y_bin, random_state=42)`) — estratificado sobre `y_bin`, no sobre `Attack_type`.
- **Preprocesamiento:** mismo `build_preprocessor(include_leaky)` del baseline multiclase (`ColumnTransformer` con `OneHotEncoder` para `proto`/`service` ajustado solo en train, dentro de un `Pipeline`).
- **Modelo base:** `DecisionTreeClassifier(class_weight="balanced", random_state=42)`, sin límite de profundidad en la etapa inicial (carácter diagnóstico).
- **Validación cruzada:** `StratifiedKFold(n_splits=5, shuffle=True, random_state=42)` sobre train, con `cross_validate` (`accuracy`, `precision`, `recall`, `f1`, `f1_macro`, `average_precision`, `return_train_score=True`).
- **Tiempos:** se miden **entrenamiento y predicción** por separado (`time.perf_counter()`), a diferencia del baseline multiclase que solo medía entrenamiento.

### Búsqueda de hiperparámetros

- **Alcance:** solo sobre la variante **sin leaky** (decisión confirmada con el equipo) — evita duplicar el paso más costoso en una variante que de todas formas no se recomienda para producción.
- **Herramienta — `GridSearchCV` en vez de Optuna:** (a) ya viene con `scikit-learn`, no agrega una dependencia nueva al proyecto; (b) el grid de 4 hiperparámetros (224 combinaciones) es pequeño y perfectamente abarcable de forma exhaustiva, no se necesita una búsqueda bayesiana; (c) es determinista y reproducible sin hiperparámetros propios que ajustar, a diferencia de un `study` de Optuna.
- **Grid:** `max_depth ∈ {3,5,8,10,15,20,None}`, `min_samples_split ∈ {2,10,20,50}`, `min_samples_leaf ∈ {1,5,10,20}`, `criterion ∈ {gini, entropy}` — 224 combinaciones × 5 folds.
- **Scoring — `f1_macro`:** da igual peso a ambas clases, coherente con `class_weight="balanced"`, y evita premiar un modelo degenerado que prediga "todo ataque" (maximizaría `recall` trivialmente pero no `f1_macro`).
- El test **no se toca** durante la búsqueda — solo `X_train`/`y_train`.

## Resultados

### Baseline inicial: con leaky vs sin leaky (test reservado, 23.585 filas)

| Variante | Accuracy | Precision | Recall | F1 | F1-macro | PR-AUC | CV F1-macro (val) | Fit (s) | Predict (s) | Profundidad | Hojas |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Con leaky | 0.9984 | 0.9988 | 0.9994 | 0.9991 | 0.9957 | 0.9988 | 0.9955 ± 0.0009 | 1.12 | 0.030 | 21 | 196 |
| **Sin leaky** | 0.9983 | 0.9989 | 0.9992 | 0.9990 | 0.9952 | 0.9988 | 0.9951 ± 0.0014 | 0.41 | 0.012 | 23 | 207 |

### Matriz de confusión (test, variante sin leaky)

| | Predicho Normal | Predicho Ataque |
|---|---:|---:|
| **Real Normal** | 2.379 (TN) | 24 (FP) |
| **Real Ataque** | 17 (FN) | 21.165 (TP) |

Solo 17 ataques no detectados de 21.182 (0.08% de falsos negativos) y 24 falsas alarmas de 2.403 normales (1.0% de falsos positivos).

### Diagnóstico de sobreajuste (F1-macro)

| Variante | Train (CV) | Validación (CV) | Test | Brecha train–CV |
|---|---:|---:|---:|---:|
| Con leaky | 1.0000 | 0.9955 | 0.9957 | 0.0045 |
| Sin leaky | 0.9997 | 0.9951 | 0.9952 | 0.0046 |

Brecha mucho menor que en el baseline multiclase (donde era de 0.04–0.06): el problema binario es más separable y el árbol, aunque crece libre, apenas memoriza ruido.

### Importancia de variables (top 5, sin leaky)

`flow_duration` (90.94%), `flow_pkts_payload.min` (1.87%), `bwd_init_window_size` (1.62%), `bwd_pkts_tot` (1.15%), `fwd_subflow_bytes` (1.03%). Prácticamente idéntico en la variante con leaky (`flow_duration` 90.86%); `id.resp_p` apenas entra al top 10 con 0.16%.

### Búsqueda de hiperparámetros

- 224 combinaciones × 5 folds, **128.3 s** de cómputo total.
- **Mejor combinación:** `max_depth=None, min_samples_split=2, min_samples_leaf=1, criterion="gini"` — idéntica a la configuración inicial sin restricciones. Mejor F1-macro (CV, train): 0.9951.
- **Efecto marginal por hiperparámetro** (rango de F1-macro promedio entre su mejor y peor valor): `max_depth` **0.0535** (con diferencia el más influyente) · `min_samples_leaf` 0.0038 · `criterion` 0.0038 · `min_samples_split` 0.0021.

### Baseline inicial (sin leaky) vs modelo ajustado

| Modelo | Accuracy | F1-macro | PR-AUC | Train F1-macro | Brecha train–test | Fit (s) | Predict (s) | Profundidad | Hojas |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Baseline inicial | 0.9983 | 0.9952 | 0.9988 | 0.9997 | 0.0044 | 0.41 | 0.012 | 23 | 207 |
| Modelo ajustado | 0.9983 | 0.9952 | 0.9988 | 0.9997 | 0.0044 | 0.48 | 0.013 | 23 | 207 |

Son el mismo modelo — ver respuesta más abajo.

## Respuestas

### ¿Cuál es el desempeño del Decision Tree para clasificación binaria?

El baseline sin leaky obtiene accuracy 0.9983, F1-macro 0.9952 y PR-AUC 0.9988 en test, con CV F1-macro 0.9951 ± 0.0014 — la medida más confiable. Solo 17 ataques no detectados de 21.182 (0.08%) y 24 falsas alarmas de 2.403 normales (1.0%). Es notablemente más alto que el baseline multiclase anterior (F1-macro ~0.93–0.97): separar 2 clases es un problema bastante más fácil que 12.

### ¿Qué diferencia existe entre utilizar y no utilizar variables leaky?

Mínima (+0.05 pp en F1-macro test, +0.04 pp en CV), muy distinto al caso multiclase (+1.6–2.3 pp). La razón: `flow_duration` domina con ~91% de la importancia en ambas variantes y ya "resuelve" el problema casi solo, sin necesitar el atajo de `id.resp_p`/`proto`/`service` (que apenas aporta 0.16% de importancia cuando se incluye). Esto refuerza, con más evidencia aún que en el caso multiclase, la recomendación de usar la variante **sin leaky**.

### ¿Qué variables son más importantes?

`flow_duration` domina de forma abrumadora (~91%), muy por encima de todo lo demás — un cambio marcado respecto al baseline multiclase, donde la importancia estaba más repartida (`fwd_last_window_size` lideraba con solo ~22%). Esta concentración en una sola variable es una observación a vigilar: probablemente refleja que el tráfico normal (dispositivos IoT legítimos, flujos sostenidos) tiene una duración de flujo estructuralmente distinta a la de los ataques (escaneos/floods, típicamente más cortos) en este dataset — no es un identificador tipo leaky, pero conviene tenerlo presente por la fragilidad que implica depender tanto de una sola señal.

### ¿El baseline presenta sobreajuste?

Muy poco. La brecha train(CV)–validación(CV) en F1-macro es de solo 0.0045–0.0046 (vs 0.04–0.06 en el baseline multiclase), a pesar de que el árbol sigue sin restricción de profundidad (23 niveles, 207 hojas). El problema binario es lo bastante separable como para que el árbol, aunque crece libre, no memorice ruido de forma significativa.

### ¿Qué hiperparámetros influyen más en el desempeño?

`max_depth` domina claramente (rango de 0.0535 en F1-macro promedio entre su mejor y peor valor) — muy por encima de `min_samples_leaf` (0.0038), `criterion` (0.0038) y `min_samples_split` (0.0021), que apenas mueven el resultado en este problema tan separable.

### ¿El modelo ajustado mejora respecto al baseline inicial?

No — y es un resultado real, no un error. `GridSearchCV` eligió como mejores hiperparámetros exactamente la configuración sin restricciones (`max_depth=None, min_samples_split=2, min_samples_leaf=1, criterion="gini"`), idéntica al baseline inicial (misma profundidad, mismas hojas, mismas métricas de test). Como el sobreajuste del baseline inicial ya era mínimo, no había margen para que podar el árbol mejorara la generalización — de hecho cualquier `max_depth` restrictivo bajaba el F1-macro promedio en la búsqueda. La búsqueda de hiperparámetros sirvió para **confirmar empíricamente** que el baseline sin restricciones ya es prácticamente óptimo para este problema, no para superarlo.

### ¿Qué dificultades aparecieron durante el entrenamiento?

- Ruido inofensivo de `joblib`/`loky` (*resource tracker*, `KeyError` en limpieza de carpetas de *memmapping*) en Windows con `n_jobs=-1`, tanto en `cross_validate` como en `GridSearchCV` — no afectó el resultado.
- El `GridSearchCV` (224 combinaciones × 5 folds = 1.120 ajustes) tomó ~128 segundos — manejable, no hizo falta reducir el grid.
- A diferencia del baseline multiclase, no hubo clases extremadamente raras en la CV (89.8%/10.2% es un desbalance mucho menos extremo que las clases de 28–36 filas del caso de 12 clases), aunque la clase Normal se sigue manejando con `class_weight="balanced"` + `stratify=y_bin`.
- Confirmar que el "modelo ajustado" resultó idéntico al baseline inicial exigió revisar cuidadosamente `grid_search.best_params_` y las métricas hasta varios decimales, para descartar un error de implementación antes de aceptarlo como resultado legítimo.

## Nota sobre trabajo relacionado del equipo

`docs/comparacion_modelos.md` / `notebooks/08_comparacion_modelos.ipynb` (Daniel) fueron construidos sobre la versión **multiclase** anterior — quedarán desalineados con el nuevo target binario oficial y necesitarán actualizarse. No se modificaron esos archivos como parte de esta tarea (fuera del alcance de Edwin).

## Artefactos generados

- `notebooks/07_baseline_decision_tree.ipynb` — notebook completo (carga, `y_bin`, comparación leaky/no-leaky, CV, evaluación, importancia de variables, `GridSearchCV`, comparación inicial-vs-ajustado, guardado del modelo).
- `models/decision_tree_baseline.pkl` — pipeline (`ColumnTransformer` + `DecisionTreeClassifier`) del modelo ajustado (variante sin leaky, ganador de `GridSearchCV`). Verificado con `joblib.load(...)`, reproduce accuracy = 0.9983 en el mismo test.
- `docs/baseline_decision_tree.md` — este documento.
- El baseline multiclase anterior queda preservado como referencia en `notebooks/07_baseline_decision_tree_old.ipynb`, `models/decision_tree_baseline_old.pkl` y `docs/baseline_decision_tree_old.md`.
