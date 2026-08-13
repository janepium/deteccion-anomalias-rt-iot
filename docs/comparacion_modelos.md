# Comparación Inicial de Modelos

**Responsable:** Daniel  
**Notebook:** `notebooks/08_comparacion_modelos.ipynb`  
**Dataset:** `data/processed/dataset_limpio.csv`

---

## 1. Objetivo

Comparar el desempeño de cuatro algoritmos de clasificación utilizando exactamente el mismo conjunto Train/Test:

- Decision Tree
- Random Forest
- Logistic Regression
- KNN

XGBoost estaba disponible en el entorno, pero no fue ejecutado debido a las limitaciones de tiempo del proyecto.

Se utilizaron las métricas:

- Accuracy
- Precision
- Recall
- F1-score
- F1-Macro
- Tiempo de entrenamiento
- Tiempo de predicción

---

## 2. Configuración

El dataset contiene **117.922 registros** y tiene como variable objetivo:

```text
Attack_type
```

Las variables potencialmente leaky fueron excluidas:

```text
id.orig_p
id.resp_p
proto
service
```

Se utilizó una división:

```text
80% Train
20% Test
```

con estratificación y `random_state=42`.

```python
train_test_split(
    X,
    y,
    test_size=0.20,
    stratify=y,
    random_state=42
)
```

Para Logistic Regression y KNN se utilizó `StandardScaler`.

Decision Tree y Random Forest no requirieron escalamiento.

---

## 3. Resultados

| Modelo | Accuracy | Precision | Recall | F1-score | F1-Macro | Train (s) | Predict (s) |
|---|---:|---:|---:|---:|---:|---:|---:|
| Decision Tree | 0.995718 | 0.962499 | 0.982527 | 0.995743 | 0.971244 | 0.675386 | **0.006383** |
| **Random Forest** | **0.996184** | **0.964506** | **0.987651** | **0.996212** | **0.974786** | 6.338418 | 0.200624 |
| Logistic Regression | 0.957346 | 0.709240 | 0.931045 | 0.957574 | 0.744948 | 94.812547 | 0.027115 |
| KNN | 0.994149 | 0.953223 | 0.973892 | 0.994165 | 0.962462 | **0.493749** | 25.088361 |

---

## 4. Análisis

### Decision Tree

Obtuvo un F1-Macro de **0.971244** y fue el modelo más rápido durante la predicción.

Su principal desventaja es el posible sobreajuste observado previamente en el baseline.

### Random Forest

Obtuvo el mejor desempeño:

- Accuracy: **0.996184**
- Recall: **0.987651**
- F1-Macro: **0.974786**

Superó al Decision Tree en F1-Macro por **0.003542** (aprox. 0.3542 puntos porcentuales).

### Logistic Regression

Obtuvo un F1-Macro de **0.744948**, considerablemente inferior al de los modelos de árboles.

Además, tuvo el mayor tiempo de entrenamiento: **94.812547 s**.

### KNN

Obtuvo un F1-Macro de **0.962462** y fue el más rápido entrenando (**0.493749 s**).

Sin embargo, su tiempo de predicción fue el mayor: **25.088361 s**.

---

## 5. Respuestas

### ¿Cuál modelo obtuvo mejor desempeño?

**Random Forest**, con el mayor F1-Macro, Recall, Precision, F1-score y Accuracy.

### ¿Cuál fue el más rápido?

- **Entrenamiento:** KNN, con 0.493749 s.
- **Predicción:** Decision Tree, con 0.006383 s.

### ¿Cuál manejó mejor el desbalance?

**Random Forest**, debido a su mayor Recall y F1-Macro.

### ¿Qué modelo será el candidato principal?

**Random Forest** es el candidato principal para continuar con el proyecto, ya que obtuvo el mejor equilibrio entre desempeño y tiempo de ejecución.

---

## 6. Conclusiones

La comparación muestra que los modelos basados en árboles obtuvieron el mejor desempeño.

Random Forest fue el modelo con mejores resultados generales y superó ligeramente al Decision Tree baseline.

KNN presentó un buen desempeño, pero su tiempo de predicción fue demasiado alto en comparación con los demás modelos.

Logistic Regression obtuvo un desempeño inferior, especialmente en F1-Macro.

Por lo tanto, **Random Forest se selecciona como candidato principal para la siguiente etapa**, donde se realizará validación y ajuste de hiperparámetros.

XGBoost no fue ejecutado debido a las limitaciones de tiempo del proyecto, aunque su librería estaba disponible.
