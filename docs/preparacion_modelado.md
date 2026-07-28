# Preparación para el Modelado — RT_IOT2022

**Responsable:** Juan David (Hecho por Daniel)  
**Dataset utilizado:** data/processed/dataset_limpio.csv  
**Notebook:** notebooks/06_preparacion_modelado.ipynb

---

# Objetivo

Preparar el dataset limpio para la etapa de entrenamiento de modelos de Machine Learning.
Esta fase incluye la separación de variables predictoras y variable objetivo, el análisis de la distribución de clases,
la división de los datos en conjuntos de entrenamiento y prueba utilizando estratificación y la definición de una estrategia de escalamiento
adecuada para el proyecto.

---

# Separación de Variables Predictoras y Variable Objetivo

La variable objetivo del proyecto es:

```python
Attack_type
```

Esta variable representa la categoría de tráfico observada en cada flujo de red y será la que el modelo intentará predecir.

Se realizó la separación entre:

- **X:** variables predictoras.
- **y:** variable objetivo (`Attack_type`).

Código utilizado:

```python
y = df["Attack_type"]
X = df.drop(columns=["Attack_type"])
```

## Resultado

- Variables predictoras: 36
- Variable objetivo: Attack_type

---

# Distribución de la Variable Objetivo

Se analizó la distribución de las clases presentes en la variable objetivo utilizando:

```python
y.value_counts()
```

y

```python
y.value_counts(normalize=True) * 100
```

También se generó una gráfica de barras para visualizar la distribución de las diferentes categorías de ataque.

## Hallazgos

Se observó que el dataset presenta un desbalance importante entre clases.

Existen categorías con una gran cantidad de registros y otras con una participación mucho menor dentro del conjunto de datos.

Este comportamiento ya había sido identificado durante las etapas previas del EDA y debe ser considerado durante el entrenamiento y evaluación de los modelos.

## Implicaciones

Debido al desbalance de clases, no es recomendable utilizar únicamente la métrica Accuracy para evaluar el rendimiento de los modelos.

Se recomienda complementar la evaluación utilizando:

- Precision
- Recall
- F1-Score
- F1-Macro
- Matriz de Confusión

---

# División de Datos en Entrenamiento y Prueba

Para realizar una evaluación adecuada del modelo se dividió el dataset en conjuntos de entrenamiento y prueba.

Código utilizado:

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.20,
    random_state=42,
    stratify=y
)
```

---

# Proporción Train/Test Utilizada

| Conjunto | Porcentaje |
|-----------|-----------|
| Entrenamiento | 80% |
| Prueba | 20% |

## Justificación

La división 80/20 permite:

- Disponer de una cantidad suficiente de datos para entrenar el modelo.
- Mantener un conjunto independiente para evaluar el desempeño.
- Reducir el riesgo de sobreajuste.
- Facilitar comparaciones futuras entre diferentes algoritmos.

---

# Uso de Estratificación

Durante la división de los datos se utilizó el parámetro:

```python
stratify=y
```

## ¿Por qué se utilizó estratificación?

El dataset presenta un desbalance significativo entre las clases de la variable objetivo.

La estratificación garantiza que las proporciones originales de cada categoría se mantengan tanto en el conjunto de entrenamiento como en el conjunto de prueba.

Sin estratificación podrían producirse problemas como:

- Ausencia de clases minoritarias en alguno de los conjuntos.
- Evaluaciones poco representativas.
- Resultados inconsistentes entre ejecuciones.

Por esta razón la estratificación es necesaria para garantizar una evaluación más confiable del modelo.

---

# Evaluación del Escalamiento

Las variables numéricas del dataset presentan rangos de valores muy diferentes.

Algunos ejemplos son:

- flow_duration
- flow_pkts_per_sec
- idle.tot
- fwd_init_window_size
- bwd_init_window_size

Esta diferencia de escalas puede afectar el rendimiento de ciertos algoritmos de Machine Learning.

---

# Métodos de Escalamiento Analizados

## StandardScaler

Estandariza las variables para que tengan:

- Media = 0
- Desviación estándar = 1

### Ventajas

- Muy utilizado en Machine Learning.
- Funciona correctamente con Regresión Logística.
- Funciona correctamente con SVM.
- Funciona correctamente con Redes Neuronales.

### Desventajas

- Sensible a valores extremos.

---

## MinMaxScaler

Transforma las variables al rango:

```text
[0,1]
```

### Ventajas

- Conserva la forma original de la distribución.
- Es útil para algunos modelos neuronales.

### Desventajas

- Muy sensible a outliers.

---

## RobustScaler

Utiliza la mediana y el rango intercuartílico para transformar los datos.

### Ventajas

- Más resistente a valores extremos.
- Adecuado cuando existen outliers.

### Desventajas

- Menos interpretativo que StandardScaler.

---

# Modelos que Requieren Escalamiento

Generalmente requieren escalamiento:

- Regresión Logística
- Support Vector Machine (SVM)
- K-Nearest Neighbors (KNN)
- Redes Neuronales
- PCA

Generalmente no requieren escalamiento:

- Árboles de Decisión
- Random Forest
- XGBoost
- LightGBM
- CatBoost

---

# Estrategia de Escalamiento del Proyecto

Para los primeros experimentos se utilizarán modelos basados en árboles de decisión.

Debido a que estos algoritmos no dependen de la escala de las variables, inicialmente no será necesario aplicar escalamiento.

Sin embargo, se mantendrá disponible una versión del pipeline utilizando StandardScaler para futuras pruebas con algoritmos sensibles a la escala.

Esta estrategia permitirá comparar diferentes modelos sin modificar la estructura principal del proyecto.

---

# Respuestas a las Preguntas del Enunciado

## ¿Qué proporción Train/Test se utilizará?

Se utilizará una división de:

- 80% para entrenamiento.
- 20% para prueba.

---

## ¿Por qué se utilizó estratificación?

Porque el dataset presenta un desbalance importante entre clases y es necesario conservar la misma distribución de 
categorías tanto en entrenamiento como en prueba.

---

## ¿Qué modelos necesitan escalamiento?

Principalmente:

- Regresión Logística
- SVM
- KNN
- Redes Neuronales
- PCA

---

## ¿Cuál será la estrategia de escalamiento del proyecto?

Inicialmente no se aplicará escalamiento para modelos basados en árboles.

Para modelos sensibles a la escala se utilizará StandardScaler dentro del pipeline de entrenamiento.

---

# Conclusiones

Se preparó correctamente el dataset para la fase de modelado mediante la separación de variables predictoras y variable objetivo.

Se confirmó la existencia de un desbalance entre clases, por lo que se utilizó estratificación durante la división de los datos para 
preservar la distribución original.

Se estableció una partición de 80% para entrenamiento y 20% para prueba, garantizando una evaluación adecuada del rendimiento de los modelos.

Finalmente, se definió una estrategia de escalamiento flexible que permitirá trabajar tanto con modelos basados en árboles como con 
algoritmos que requieren normalización o estandarización de los datos.
