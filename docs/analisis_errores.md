# Análisis de Errores y Evaluación por Attack_type

Este documento contiene el análisis detallado de los errores de clasificación generados por el modelo Random Forest binario (entrenado en la semana 4), desglosando el rendimiento a través de la variable `Attack_type`.

---

## 1. Matriz de Confusión Global

| | Predicción: Normal (0) | Predicción: Ataque (1) | Total Real |

| **Real: Normal (0)** | **2,379** (TN) | **24** (FP)     | 2,403 |
| **Real: Ataque (1)** | **11** (FN)    | **21,171** (TP) | 21,182 |

- **Falsos Negativos (FN) Totales:** 11 ataques omitidos por el modelo.
- **Falsos Positivos (FP) Totales:** 24 flujos de tráfico normal etiquetados erróneamente como ataque.

---

## 2. Análisis de Falsos Negativos (FN)

De las 21,182 instancias de ataque en el conjunto de prueba, solo 11 no fueron detectadas. Los FN se concentran exclusivamente en dos categorías de ataque:

| Attack_type | Total Casos | Casos FN | Porcentaje de FN (%) |

| **ARP_poisioning** | 1,542 | 10 | 0.65% |
| **NMAP_UDP_SCAN**  | 551   | 1  | 0.18% |

---

## 3. Rendimiento Detallado por Clase de Attack_type

A continuación se presenta la evaluación del modelo para cada tipo de ataque registrado en el conjunto de prueba:

| Attack_type | Total Datos | Clasificados Correctamente | Falsos Negativos (FN) | Recall (%) |

| **ARP_poisioning**             | 1,542  | 1,532  | 10 | **99.35%**  |
| **NMAP_UDP_SCAN**              |  551   |   550  |  1 | **99.82%**  |
| **DDOS_Slowloris**             |  116   |   116  |  0 | **100.00%** |
| **DOS_SYN_Hping**              | 17,893 | 17,893 |  0 | **100.00%** |
| **Metasploit_Brute_Force_SSH** |    6   |    6   |  0 | **100.00%** |
| **NMAP_FIN_SCAN**              |    4   |    4   |  0 | **100.00%** |
| **NMAP_OS_DETECTION**          |   409  |   409  |  0 | **100.00%** |
| **NMAP_TCP_scan**              |   210  |   210  |  0 | **100.00%** |
| **NMAP_XMAS_TREE_SCAN**        |   451  |   451  |  0 | **100.00%** |

---

## 4. Análisis de Falsos Positivos (FP) en Tráfico Normal

Evaluación del comportamiento del modelo sobre las clases de tráfico benigno:

| Tráfico Normal | Total Casos | Casos FP | Porcentaje FP (%) |

| **Wipro_bulb**   |   47  | 3  | **6.38%** |
| **Thing_Speak**  | 1,564 | 21 | **1.34%** |
| **MQTT_Publish** |   792 |  0 | **0.00%** |

---

## 5. Respuestas a Preguntas Clave

### ¿Qué ataques son más difíciles de detectar?
El ataque con mayor dificultad de detección es **`ARP_poisioning`**, alcanzando un porcentaje de omisión del **0.65%** (10 Falsos Negativos), seguido de **`NMAP_UDP_SCAN`** con un **0.18%** (1 Falso Negativo). El resto de las 7 categorías de ataque alcanzaron un Recall perfecto del **100.00%**.

### ¿Dónde se concentran los Falsos Negativos (FN)?
Los Falsos Negativos se concentran casi en su totalidad en **`ARP_poisioning`**, que representa el **90.91%** de todas las omisiones del modelo (10 de los 11 FN totales).

### ¿Las clases minoritarias presentan mayor dificultad?
- **En clases de ataque: No.** 
- Las clases con menor volumen en el dataset (`NMAP_FIN_SCAN` con 28 muestras totales y `Metasploit_Brute_Force_SSH` con 36 muestras totales) lograron un **100.00% de Recall** sin ningún Falso Negativo.
- **En tráfico normal: Si.** 
- La clase de tráfico benigno con menor representación en el dataset, **`Wipro_bulb`** (219 muestras totales en el dataset), tuvo la tasa más alta de Falsos Positivos con un **6.38%**.

### ¿Qué tráfico normal genera más Falsos Positios (FP)?
- **En volumen absoluto:** 
- **`Thing_Speak`** produce la mayor cantidad de falsas alarmas (21 Falsos Positivos).
- **En proporción:** 
- **`Wipro_bulb`** presenta la tasa de error relativa más alta, con un **6.38%** de sus flujos clasificados como ataques.

### ¿El resultado binario está ocultando problemas en determinados tipos de ataque?
**No de manera crítica.** 
La tasa de detección (Recall) se mantiene por encima del **99.35%** en todas las clases de ataque. Sin embargo, el valor global de Recall (> 99.9%) enmascara dos aspectos específicos:
1. El error de detección no es aleatorio: se concentra en ataques a nivel de enlace de datos (`ARP_poisioning`).
2. Muestra una ligera degradación en la precisión sobre dispositivos IoT con menor representación (`Wipro_bulb`), generando alertas falsas innecesarias.