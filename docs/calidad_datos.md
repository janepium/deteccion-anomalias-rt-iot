# Calidad de Datos — RT_IOT2022

**Responsable:** Edwin  
**Dataset:** `data/rt-iot2022.zip` → archivo interno `RT_IOT2022`  
**Fecha de análisis:** 2026-05-22  
**Notebook:** `notebooks/eda_edwin_calidad_datos.ipynb`

---

## Estructura general

| Métrica | Valor |
|---|---|
| Filas | 123,117 |
| Columnas | 84 |
| Variables numéricas | 81 |
| Variables categóricas | 3 (`proto`, `service`, `Attack_type`) |
| Valores nulos | **0** |
| Filas duplicadas exactas | Ver notebook (ejecutar celda 6) |

> El dataset se carga con `index_col=0`, por lo que la columna de índice residual del CSV queda como índice del DataFrame y no cuenta entre las 84 columnas de features.

---

## Variables principales

| Variable | Tipo | Descripción |
|---|---|---|
| `proto` | object / category | Protocolo de red (`tcp`, `udp`, etc.) |
| `service` | object / category | Servicio detectado (`mqtt`, `-`, etc.) |
| `Attack_type` | object / category | **Variable objetivo** — tipo de tráfico/ataque |
| `id.orig_p` | int64 | Puerto de origen (identificador) |
| `id.resp_p` | int64 | Puerto de respuesta (identificador) |

---

## Valores faltantes

No se detectaron valores nulos en ninguna columna. El dataset está completo.

---

## Duplicados

Revisar al ejecutar el notebook. Si existen, se recomienda eliminarlos con `df.drop_duplicates()` antes del modelado.

---

## Distribución de la variable objetivo (`Attack_type`)

| Clase | Conteo | % |
|---|---|---|
| DOS_SYN_Hping | 94,659 | 76.88% |
| Thing_Speak | 8,108 | 6.59% |
| ARP_poisioning | 7,750 | 6.29% |
| MQTT_Publish | 4,146 | 3.37% |
| NMAP_UDP_SCAN | 2,590 | 2.10% |
| NMAP_XMAS_TREE_SCAN | 2,010 | 1.63% |
| NMAP_OS_DETECTION | 2,000 | 1.62% |
| NMAP_TCP_scan | 1,002 | 0.81% |
| DDOS_Slowloris | 534 | 0.43% |
| Wipro_bulb | 253 | 0.21% |
| Metasploit_Brute_Force_SSH | 37 | 0.03% |
| NMAP_FIN_SCAN | 28 | 0.02% |

**Ratio de desbalance:** ~3,381:1 entre la clase mayoritaria y la minoritaria.  
Este desbalance extremo debe tenerse en cuenta en la estrategia de modelado (SMOTE, pesos de clase, métricas balanceadas como F1-macro o AUC).

---

## Columnas constantes o casi constantes

Se detecta mediante el notebook (sección 8). Columnas donde un único valor representa >99% de los registros son candidatas a eliminación en el pipeline de features.

---

## Columnas con tipos incorrectos

- `proto` y `service` están tipadas como `object`. Se recomienda convertir a `category` para reducir uso de memoria y habilitar encoding eficiente.
- `id.orig_p` e `id.resp_p` son enteros pero funcionan como identificadores de sesión, no como magnitudes continuas. Evaluar si incluirlos como features o excluirlos del modelo.

---

## Inconsistencias en categorías

- `service` incluye el valor `"-"` (guion) que representa ausencia de servicio — puede tratarse como `None` o codificarse como categoría propia.
- `Attack_type` tiene nombres consistentes en mayúsculas con guion bajo (`MQTT_Publish`, `DOS_SYN_Hping`). No se detectaron espacios extra ni variantes de capitalización.

---

## Limpiezas propuestas

| Prioridad | Acción | Justificación |
|---|---|---|
| Alta | El índice residual se gestiona con `index_col=0` al cargar | No requiere paso manual adicional |
| Alta | Eliminar filas duplicadas (si existen) | Evita sesgo en entrenamiento |
| Media | Convertir `proto`, `service`, `Attack_type` a `category` | Ahorro de memoria ~60-70% en esas columnas |
| Media | Documentar y gestionar desbalance en `Attack_type` | Clase dominante es 77% del dataset |
| Baja | Evaluar columnas casi constantes para feature selection | Reducir ruido en el modelo |
| Baja | Decidir rol de `id.orig_p` e `id.resp_p` | Puertos como feature vs. identificador |

---

## Preguntas respondidas

| Pregunta | Respuesta |
|---|---|
| ¿Hay valores nulos? | **No.** El dataset está completo. |
| ¿Hay filas duplicadas? | Ver notebook — ejecutar `df.duplicated().sum()` |
| ¿Hay columnas que no aportan? | Ninguna — `index_col=0` maneja el índice residual al cargar. |
| ¿Qué columnas son categóricas? | `proto`, `service`, `Attack_type` |
| ¿Qué columnas son numéricas? | Las 81 restantes (flujos, paquetes, tiempos, flags, ventanas) |
| ¿Hay columnas que deban eliminarse? | Ninguna obligatoria; evaluar `id.orig_p` / `id.resp_p` según uso |

---

*Documento generado con base en análisis exploratorio inicial del dataset RT_IOT2022 (123,117 registros de tráfico IoT).*
