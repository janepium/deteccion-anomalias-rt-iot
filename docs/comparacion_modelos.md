# Comparación de Modelos de Clasificación Binaria (IDS)

**Responsable:** Daniel  
**Notebook:** `notebooks/08_comparacion_modelos.ipynb`  
**Dataset:** `data/processed/dataset_limpio.csv`

---

## 1. Objetivo

El objetivo de esta etapa es comparar diferentes algoritmos de clasificación para determinar cuál ofrece el mejor desempeño en la detección binaria de tráfico normal y tráfico de ataque en el dataset RT_IOT2022.

Los modelos evaluados son:

- Decision Tree Baseline
- Random Forest
- Logistic Regression
- KNN
- SVM

Todos los modelos fueron evaluados utilizando el mismo conjunto de entrenamiento y prueba y bajo el mismo protocolo experimental.

---

## 2. Configuración experimental

### 2.1 Variable objetivo

Se utilizó la variable binaria:

```text
0 = Normal
1 = Ataque
```

La clase **Normal** corresponde al tráfico legítimo de:

- `MQTT_Publish`
- `Thing_Speak`
- `Wipro_bulb`

La clase **Ataque** corresponde al resto de categorías presentes en `Attack_type`.

`Attack_type` se conservó únicamente para trazabilidad y no fue utilizado como variable objetivo del modelo.

---

### 2.2 División de datos

Se utilizó una división:

```text
80% Train
20% Test
```

La partición se realizó mediante estratificación:

```python
train_test_split(
    X,
    y_bin,
    test_size=0.20,
    stratify=y_bin,
    random_state=42
)
```


---

### 2.3 Variables potencialmente leaky

Se excluyeron las siguientes variables:

```text
id.orig_p
id.resp_p
proto
service
```

Estas variables fueron identificadas previamente como potencialmente leaky y podían proporcionar información demasiado directa sobre la clase.

Por esta razón, todos los modelos de esta comparación se entrenaron sin estas variables.

---

### 2.4 Escalamiento

La estrategia de escalamiento dependió del algoritmo:

| Modelo | Escalamiento |
|---|---|
| Decision Tree | No |
| Random Forest | No |
| Logistic Regression | Sí |
| KNN | Sí |
| SVM | Sí |

Los modelos basados en árboles no requieren escalamiento.

Logistic Regression, KNN y SVM utilizaron `StandardScaler` mediante `Pipeline`.

---

## 3. Modelos evaluados

### 3.1 Decision Tree Baseline

Se utilizó como referencia el baseline desarrollado en `07_baseline_desicion_tree.ipynb` .

Resultados del baseline sin variables leaky:

- Accuracy: **0.9983**
- Precision: **0.9989**
- Recall: **0.9992**
- F1-score: **0.9990**
- F1-Macro: **0.9952**
- PR-AUC: **0.9988**

Este resultado establece el punto de referencia para los demás modelos.

---

### 3.2 Random Forest

Configuración utilizada:

```python
RandomForestClassifier(
    n_estimators=100,
    class_weight="balanced",
    random_state=42,
    n_jobs=-1
)
```

El uso de múltiples árboles permite combinar diferentes reglas de decisión y obtener un modelo más robusto que un único Decision Tree.

---

### 3.3 Logistic Regression

Se utilizó:

```python
Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(
        max_iter=2000,
        class_weight="balanced"
    ))
])
```

Este modelo representa una alternativa lineal frente a los modelos basados en árboles.

---

### 3.4 KNN

Se utilizó:

```python
Pipeline([
    ("scaler", StandardScaler()),
    ("model", KNeighborsClassifier(
        n_neighbors=5
    ))
])
```

El escalamiento es necesario porque KNN utiliza distancias entre observaciones.

---

### 3.5 SVM

Se utilizó un SVM con kernel RBF:

```python
Pipeline([
    ("scaler", StandardScaler()),
    ("model", SVC(
        kernel="rbf",
        probability=True,
        class_weight="balanced",
        random_state=42
    ))
])
```

`probability=True` permitió obtener probabilidades para calcular PR-AUC y analizar calibración.

---

## 4. Métricas utilizadas

Se utilizaron:

- **Accuracy:** proporción total de predicciones correctas.
- **Precision:** proporción de predicciones positivas que fueron correctas.
- **Recall:** proporción de ataques correctamente detectados.
- **F1-score:** equilibrio entre Precision y Recall.
- **F1-Macro:** promedio del F1 de ambas clases dando el mismo peso a Normal y Ataque.
- **PR-AUC:** medida del comportamiento conjunto de Precision y Recall.
- **Tiempo de entrenamiento:** tiempo necesario para ajustar el modelo.
- **Tiempo de predicción:** tiempo necesario para generar las predicciones.

Debido al desbalance entre Normal y Ataque, F1-Macro, Recall y PR-AUC tienen especial importancia.

---

## 5. Resultados

Los resultados obtenidos fueron:

| Modelo | Accuracy | Precision | Recall | F1-Score | F1-Macro | PR-AUC | FN | FP | Train (s) | Predict (s) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| **Random Forest** | **0.9985** | **0.9989** | **0.9995** | **0.9992** | **0.9959** | **0.9998** | **11** | **24** | 6.23 | 0.55 |
| Decision Tree Baseline | 0.9983 | 0.9989 | 0.9992 | 0.9990 | 0.9952 | 0.9988 | 18 | 23 | **0.41** | **0.01** |
| KNN | 0.9973 | 0.9976 | 0.9993 | 0.9985 | 0.9925 | 0.9993 | 14 | 50 | **0.08** | 15.81 |
| SVM | 0.9823 | 0.9988 | 0.9815 | 0.9901 | 0.9547 | 0.9996 | 391 | 26 | 860.05 | 5.53 |
| Logistic Regression | 0.9632 | 0.9979 | 0.9610 | 0.9791 | 0.9118 | 0.9991 | 826 | 43 | 6.45 | **0.01** |

---

## 6. Análisis de los resultados

### 6.1 Random Forest

Random Forest obtuvo el mejor desempeño general:

- Accuracy: **0.9985**
- Recall: **0.9995**
- F1-Macro: **0.9959**
- PR-AUC: **0.9998**
- FN: **11**

Superó ligeramente al Decision Tree baseline y redujo los falsos negativos de 18 a 11.

Su costo fue mayor que el del Decision Tree, pero continuó siendo razonable:

- Entrenamiento: **6.23 s**
- Predicción: **0.55 s**

---

### 6.2 Decision Tree

El baseline obtuvo:

- Accuracy: **0.9983**
- Recall: **0.9992**
- F1-Macro: **0.9952**
- PR-AUC: **0.9988**
- FN: **18**

Fue el modelo más rápido durante la predicción:

```text
0.01 s
```

Su principal desventaja es una mayor sensibilidad al sobreajuste frente a un conjunto de múltiples árboles.

---

### 6.3 KNN

KNN obtuvo:

- Accuracy: **0.9973**
- Recall: **0.9993**
- F1-Macro: **0.9925**
- PR-AUC: **0.9993**
- FN: **14**

Fue el modelo más rápido durante el entrenamiento:

```text
0.08 s
```

Sin embargo, tuvo un tiempo de predicción de:

```text
15.81 s
```

Esto limita considerablemente su utilidad para un sistema IDS que deba procesar tráfico de forma continua.

---

### 6.4 SVM

SVM obtuvo:

- Accuracy: **0.9823**
- Recall: **0.9815**
- F1-Macro: **0.9547**
- PR-AUC: **0.9996**
- FN: **391**

Aunque obtuvo una Precision alta, dejó pasar muchos más ataques que Random Forest, Decision Tree y KNN.

Además, presentó el mayor costo computacional:

```text
Train = 860.05 s
Predict = 5.53 s
```

Por estas razones, no resulta conveniente para continuar bajo esta configuración.

---

### 6.5 Logistic Regression

Logistic Regression obtuvo:

- Accuracy: **0.9632**
- Recall: **0.9610**
- F1-Macro: **0.9118**
- PR-AUC: **0.9991**
- FN: **826**

Aunque su Precision fue elevada, dejó pasar 826 ataques.

Esto representa el peor resultado entre los modelos evaluados en términos de falsos negativos.

Su tiempo de entrenamiento fue de:

```text
6.45 s
```

Por lo tanto, presenta un peor equilibrio entre detección y costo que los modelos basados en árboles.

---

## 7. Matrices de confusión

La matriz de confusión se interpreta como:

| | Predicción Normal | Predicción Ataque |
|---|---:|---:|
| **Real Normal** | TN | FP |
| **Real Ataque** | FN | TP |

Los resultados fueron:

| Modelo | TN | FP | FN | TP |
|---|---:|---:|---:|---:|
| **Random Forest** | 2.379 | 24 | **11** | 21.171 |
| Decision Tree | 2.380 | 23 | 18 | 21.164 |
| KNN | 2.353 | 50 | 14 | 21.168 |
| SVM | 2.377 | 26 | 391 | 20.791 |
| Logistic Regression | 2.360 | 43 | 826 | 20.356 |

En un IDS, los falsos negativos son especialmente importantes porque representan ataques que fueron clasificados como tráfico normal.

Random Forest presentó el menor número de falsos negativos.

---

## 8. Precision-Recall

La curva Precision-Recall permite analizar el compromiso entre detectar ataques y generar falsas alarmas.

Los valores de PR-AUC fueron:

| Modelo | PR-AUC |
|---|---:|
| **Random Forest** | **0.9998** |
| SVM | 0.9996 |
| KNN | 0.9993 |
| Logistic Regression | 0.9991 |
| Decision Tree | 0.9988 |

Random Forest obtuvo el mayor PR-AUC.

Por lo tanto, presentó el mejor comportamiento general en Precision-Recall.

---

## 9. Calibración

La calibración evalúa qué tan bien las probabilidades predichas representan la frecuencia real de los eventos.

Los resultados mostraron:

- **KNN:** comportamiento más cercano a la diagonal ideal.
- **Logistic Regression:** tendencia a subestimar probabilidades en ciertos rangos.
- **Random Forest:** desviaciones principalmente en probabilidades intermedias.
- **SVM:** comportamiento más irregular en algunos rangos.

La calibración no determina por sí sola cuál es el mejor clasificador; evalúa la calidad de sus probabilidades.

---

## 10. Comparación de tiempos

### Entrenamiento

| Modelo | Tiempo |
|---|---:|
| KNN | **0.08 s** |
| Decision Tree | 0.41 s |
| Random Forest | 6.23 s |
| Logistic Regression | 6.45 s |
| SVM | 860.05 s |

KNN fue el más rápido entrenando.

### Predicción

| Modelo | Tiempo |
|---|---:|
| Decision Tree | **0.01 s** |
| Logistic Regression | **0.01 s** |
| Random Forest | 0.55 s |
| SVM | 5.53 s |
| KNN | 15.81 s |

Decision Tree y Logistic Regression fueron los más rápidos prediciendo.

---

## 11. Relación entre desempeño y costo computacional

La selección del modelo no debe basarse solamente en Accuracy.

También deben considerarse:

- F1-Macro.
- Recall.
- PR-AUC.
- Falsos negativos.
- Tiempo de entrenamiento.
- Tiempo de predicción.

KNN presenta un buen desempeño, pero su tiempo de predicción es demasiado elevado.

SVM tiene un costo computacional muy alto y un número elevado de falsos negativos.

Logistic Regression tiene tiempos moderados, pero un F1-Macro y Recall inferiores.

Decision Tree es extremadamente rápido y presenta un desempeño excelente, pero tiene mayor riesgo de sobreajuste.

Random Forest ofrece el mejor equilibrio entre desempeño, detección de ataques y costo computacional.

---

## 12. Respuestas a las preguntas del proyecto

### ¿Cuál modelo obtuvo el mejor desempeño?

**Random Forest**, con:

- Accuracy: **0.9985**
- F1-Macro: **0.9959**
- Recall: **0.9995**
- PR-AUC: **0.9998**

---

### ¿Cuál detecta mejor los ataques?

**Random Forest**, debido a que obtuvo el mayor Recall:

```text
0.9995
```

y detectó correctamente 21.171 de los 21.182 ataques.

---

### ¿Cuál presenta menos falsos negativos?

**Random Forest**, con únicamente:

```text
11 falsos negativos
```

frente a:

```text
Decision Tree = 18
KNN = 14
SVM = 391
Logistic Regression = 826
```

---

### ¿Cuál maneja mejor el desbalance?

**Random Forest**, debido a su mayor F1-Macro y Recall.

Además, utiliza `class_weight="balanced"` para considerar la diferencia entre las clases.

---

### ¿Cuál es el más rápido?

Depende de la fase:

- **Entrenamiento:** KNN, con **0.08 s**.
- **Predicción:** Decision Tree y Logistic Regression, con aproximadamente **0.01 s**.

---

### ¿Cuál tiene mejor relación entre desempeño y costo computacional?

**Random Forest**.

Aunque requiere más tiempo que Decision Tree y KNN en algunas etapas, obtiene mejores métricas generales y reduce los falsos negativos a solamente 11.

---

### ¿Qué modelo debería pasar a la siguiente etapa de optimización?

**Random Forest**.

Es el modelo con el mejor equilibrio entre desempeño, detección de ataques y costo computacional.

---

### ¿Qué modelo presenta peor comportamiento y por qué?

En términos de desempeño y detección de ataques, **Logistic Regression** presentó el peor resultado, con un F1-Macro de **0.9118** y **826 falsos negativos**.

SVM también presentó un comportamiento poco conveniente debido a su elevado costo computacional de **860.05 s** de entrenamiento y **391 falsos negativos**.

---

## 13. Conclusiones

Los resultados obtenidos no aparecen de manera aleatoria. Se explican principalmente por cómo quedó construido el dataset después de la limpieza, por las características de las variables que se conservaron y por las diferencias entre los algoritmos utilizados.

Durante la limpieza se eliminaron variables redundantes, pero se conservaron características que describen directamente el comportamiento del tráfico de red, como duración de los flujos, cantidad de paquetes, velocidad del tráfico, payload, flags TCP y parámetros de las conexiones. Estas variables contienen información suficiente para diferenciar con mucha precisión el tráfico normal del tráfico de ataque.

Los modelos basados en árboles, especialmente **Decision Tree y Random Forest**, se benefician de estas características porque pueden encontrar relaciones no lineales entre las variables. Conceptualmente, pueden aprender reglas que combinen variables como cantidad de paquetes por segundo, presencia de determinados flags TCP y duración del flujo para determinar si una observación corresponde a tráfico normal o a un ataque.

Esto ayuda a explicar por qué los modelos basados en árboles obtuvieron los mejores resultados. **Random Forest** fue el mejor modelo de la comparación, alcanzando un F1-Macro de **0.9959**, un Recall de **0.9995** y un PR-AUC de **0.9998**. Además, redujo los falsos negativos del Decision Tree baseline de 18 a solamente 11.

El **Decision Tree** presentó un desempeño muy similar a Random Forest y destacó por su bajo costo computacional, especialmente durante la predicción. Sin embargo, Random Forest ofrece una ligera mejora en las métricas principales y mayor robustez potencial al combinar múltiples árboles.

**KNN** obtuvo buenos resultados y fue el modelo más rápido durante el entrenamiento, pero su tiempo de predicción fue de **15.81 segundos**, lo que limita su utilidad para un IDS que deba realizar predicciones continuamente.

**Logistic Regression** presentó un desempeño inferior, especialmente en F1-Macro (**0.9118**), y dejó pasar **826 ataques**. Esto demuestra que una Accuracy relativamente alta no es suficiente para evaluar este problema, debido al desbalance de las clases y a la importancia de minimizar los falsos negativos.

**SVM** presentó el mayor costo computacional, con **860.05 segundos** de entrenamiento, y dejó pasar **391 ataques**. Por esta razón, no resulta conveniente para continuar bajo la configuración utilizada.

En conclusión, **Random Forest se selecciona como candidato principal para la siguiente etapa del proyecto**, ya que obtuvo el mejor desempeño general, el mayor Recall y PR-AUC, además de presentar solamente 11 falsos negativos con un costo computacional razonable.
