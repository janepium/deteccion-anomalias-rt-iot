# Análisis de Errores y Evaluación por Attack_type

A continuación presenta el análisis detallado de los errores de clasificación generados por el modelo **Random Forest binario** (entrenado en la semana 4). El objetivo es desglosar el rendimiento a través de la variable `Attack_type` para identificar vulnerabilidades específicas y proponer mejoras en la detección de amenazas de red.

---

## 📊 1. Perspectivas clave

*   **Ataque más crítico:** **`ARP_poisioning`** es el más difícil de detectar. Concentra el **90.91% de todos los Falsos Negativos** del sistema (10 de 11 omisiones).

*   **Falsas Alarmas en IoT:** Las clases minoritarias de tráfico normal sufren de baja representación. El dispositivo **`Wipro_bulb`** presenta la tasa de error relativa más alta con un **6.38% de Falsos Positivos**.

*   **Efecto del Desbalanceo:** El volumen general oculta problemas específicos. Aunque el Recall global parece perfecto (>99.9%), el error real está concentrado en ataques de capa de enlace (ARP) y en tráfico benigno de dispositivos IoT específicos.

---

## 📈 2. Matriz de Confusión Global

| Estado Real | Predicción: Normal (0) | Predicción: Ataque (1) | Total Real |

| **Real: Normal (0)** | **2,379** (TN) | **24** (FP) ⚠️ | 2,403 |
| **Real: Ataque (1)** | **11** (FN) 🚨 | **21,171** (TP) | 21,182 |

*   **Falsos Negativos (FN):** **11** ataques críticos omitidos (el modelo los consideró flujos seguros).
*   **Falsos Positivos (FP):** **24** flujos normales que generaron falsas alarmas de seguridad.

---

## 🚨 3. Análisis de Falsos Negativos (Ataques Omitidos)

De las 21,182 instancias de ataque en el conjunto de prueba, **solo 11 no fueron detectadas**. Estos errores no son aleatorios; se concentran exclusivamente en dos categorías de ataque:

| Attack_type        | Total Casos | Casos FN | Porcentaje de FN (%) | Severidad / Diagnóstico |

| **ARP_poisioning** |     1,542   |    10    |       **0.65%**      | 🔴 **Alta** (Representa el 90.91% del error total de omisión) |
| **NMAP_UDP_SCAN** |      551     |     1    |       **0.18%**      | 🟡 **Baja** (Error residual aislado) |

---

## ⚠️ 4. Análisis de Falsos Positivos (Alertas Falsas)

Evaluación del comportamiento del modelo sobre las clases de tráfico benigno. Los dispositivos con menos datos sufren la mayor tasa de error relativo:

| Tráfico Normal | Total Casos | Casos FP | Porcentaje FP (%) | Diagnóstico Operativo |

| **Wipro_bulb** |      47     |     3    |     **6.38%**     | ❌ **Alta tasa de error relativo** debido a una baja representatividad. |
| **Thing_Speak** |   1,564    |    21    |     **1.34%**     | ⚠️ **Mayor volumen absoluto** de falsas alarmas operativas. |
| **MQTT_Publish** |    792    |     0    |     **0.00%**     | ✅ **Rendimiento perfecto** sin alertas falsas. |

---

## ✅ 5. Rendimiento Detallado por Clase de Attack_type

A continuación se presenta la evaluación completa del modelo ordenada por efectividad. Siete de las nueve categorías de ataque alcanzan una detección perfecta:

|    Attack_type     | Total Datos | Clasificados OK | Falsos Negativos (FN) | Recall (%) | Estado del Arte |

| **DDOS_Slowloris**             |   116  |   116  | 0 | **100.00%** | ✅ Perfecto |

| **DOS_SYN_Hping**              | 17,893 | 17,893 | 0 | **100.00%** | ✅ Perfecto |

| **Metasploit_Brute_Force_SSH** |    6   |    6   | 0 | **100.00%** | ✅ Perfecto |

| **NMAP_FIN_SCAN**              |    4   |    4   | 0 | **100.00%** | ✅ Perfecto |

| **NMAP_OS_DETECTION**          |   409  |   409  | 0 | **100.00%** | ✅ Perfecto |

| **NMAP_TCP_scan**              |   210  |   210  | 0 | **100.00%** | ✅ Perfecto |

| **NMAP_XMAS_TREE_SCAN**        |   451  |   451  | 0 | **100.00%** | ✅ Perfecto |

| **NMAP_UDP_SCAN**              |   551  |   550  | 1 | **99.82%**  | 🔄 Excelente |

| **ARP_poisioning**             |  1,542 | 1,532  | 10 | **99.35%** | ⚠️ Requiere Ajuste |

---

## 🛠️ 6. Plan de Acción Recomendado

1. **Ingeniería de Características en Capa 2 (Red):** El ataque `ARP_poisioning` actúa a nivel de enlace de datos. Se sugiere incorporar o revisar variables como el conteo de paquetes ARP por segundo o discrepancias en la tabla MAC-IP para mitigar el 0.65% de omisión.

2. **Estrategia de Balanceo para Clases Minoritarias:** Aplicar técnicas como SMOTE o *Data Augmentation* enfocadas en el tráfico de **`Wipro_bulb`** y **`Thing_Speak`** para corregir los patrones falsamente asociados a firmas de ataques y reducir las alertas en producción.
