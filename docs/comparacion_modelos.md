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

Los resultados obtenidos no aparecen de manera aleatoria. Se explican principalmente por cómo quedó construido nuestro dataset después de la limpieza, por las características de las variables que se conservaron y por las diferencias entre los algoritmos utilizados.

Durante el proceso de limpieza se redujo considerablemente la cantidad de variables redundantes, pero se conservaron características relacionadas directamente con el comportamiento del tráfico de red, como `flow_duration`, cantidad de paquetes, `flow_pkts_per_sec`, variables de payload, flags TCP y tamaños de ventana. Estas variables contienen información útil para diferenciar los diferentes tipos de tráfico y ataques presentes en `Attack_type`.

Los modelos basados en árboles, especialmente **Decision Tree y Random Forest**, se benefician de estas características porque pueden encontrar relaciones no lineales entre las variables. Conceptualmente, un modelo puede aprender reglas como:

```text
Si la cantidad de paquetes por segundo es alta
y ciertos flags TCP aparecen con determinada frecuencia
y la duración del flujo se encuentra dentro de cierto rango,
entonces aumenta la probabilidad de pertenecer a una determinada clase.
```

Este tipo de relaciones es difícil de representar mediante un modelo estrictamente lineal, pero puede ser aprendido de manera natural por los árboles de decisión.

Esto ayuda a explicar por qué los modelos basados en árboles obtuvieron los mejores resultados. **Random Forest** fue el modelo con mejor desempeño general, alcanzando un Accuracy de **0.996184**, un Recall de **0.987651** y un F1-Macro de **0.974786**. Además, superó ligeramente al Decision Tree baseline, que obtuvo un F1-Macro de **0.971244**.

La mejora de Random Forest puede explicarse por su utilización de múltiples árboles, lo que permite combinar diferentes reglas de decisión y obtener un modelo más robusto que un único árbol. Aunque la diferencia respecto al Decision Tree fue pequeña, Random Forest obtuvo mejores resultados en las principales métricas.

El **Decision Tree** también obtuvo un desempeño muy alto y fue el modelo más rápido durante la predicción, con **0.006383 segundos**. Sin embargo, el baseline mostró previamente señales de sobreajuste, por lo que se considera menos robusto que Random Forest para continuar el proyecto.

El comportamiento de **KNN** también puede explicarse por las características del dataset. Aunque fue el modelo más rápido durante el entrenamiento, con **0.493749 segundos**, necesita calcular distancias entre las observaciones durante la predicción. Debido al tamaño del dataset, esto produjo un tiempo de predicción de **25.088361 segundos**, considerablemente mayor que el de los demás modelos.

Por su parte, **Logistic Regression** obtuvo un Accuracy de **0.957346**, pero un F1-Macro de solamente **0.744948**. Esto se relaciona con el fuerte desbalance existente en `Attack_type`. Una Accuracy alta puede estar influenciada por la clase mayoritaria y no reflejar correctamente el comportamiento sobre las clases minoritarias. Por esta razón, F1-Macro y Recall son métricas más representativas para este problema.

El desbalance del dataset es especialmente importante, ya que la clase mayoritaria representa una proporción mucho mayor de los registros que las clases minoritarias. Por ello, el uso de `class_weight="balanced"` en los modelos que lo permiten y la evaluación mediante F1-Macro permiten obtener una visión más equilibrada del desempeño.

Finalmente, **Random Forest se selecciona como candidato principal para la siguiente etapa**, debido a que presentó el mejor equilibrio entre desempeño predictivo y costo computacional.

**XGBoost no fue ejecutado debido a las limitaciones de tiempo del proyecto**, aunque la librería estaba disponible en el entorno. Por esta razón, no forma parte de la comparación ni de la selección realizada en esta etapa.
