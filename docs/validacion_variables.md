# Validación de Variables — RT_IOT2022

**Responsable:** Daniel
**Dataset analizado:** data/processed/dataset_limpio.csv  
**Notebook:** notebooks/05_validacion_variables.ipynb  
**Fecha:** 2026-07-14

---

# Objetivo

Verificar que las decisiones tomadas durante las etapas de EDA y limpieza de datos fueron aplicadas correctamente al dataset final.
Se revisó la presencia de variables constantes, posibles variables redundantes,
correlaciones entre características y la coherencia de las variables seleccionadas para el entrenamiento de modelos de clasificación.

---

# Resumen

| Métrica | Resultado |
|----------|----------|
| Columnas del dataset limpio | 37 |
| Variables constantes encontradas | 0 |
| Variables con alta correlación detectadas | 3 |
| Estado general del dataset | Apto para modelado |

---

# Verificación del Dataset

Durante la validación se cargó el dataset limpio generado en la fase de preprocesamiento.

Se identificó una columna denominada `Unnamed: 0`, correspondiente al índice exportado automáticamente por Pandas al guardar el archivo CSV.
Esta columna no representa una característica real del dataset y fue excluida del análisis.

Después de eliminar dicha columna, el dataset quedó compuesto por 37 variables, coincidiendo con la documentación generada durante la etapa de limpieza.

---

# Revisión de Variables Constantes

Se verificó el número de valores únicos presentes en cada columna mediante el método `nunique()`.

Código utilizado:

```python
constantes = [col for col in df.columns if df[col].nunique() <= 1]
print(constantes)
```

Resultado obtenido:

```python
[]
```

## Hallazgos

No se encontraron variables constantes dentro del dataset limpio.

Esto indica que todas las variables restantes presentan variabilidad y pueden aportar información útil durante el proceso de entrenamiento de modelos.

La limpieza realizada previamente eliminó correctamente las columnas sin capacidad discriminativa.

---

# Análisis de Correlaciones

Con el fin de detectar posibles redundancias entre variables, se calculó la matriz de correlación utilizando únicamente las variables numéricas.

Posteriormente se identificaron aquellas variables que presentaban correlaciones superiores a 0.95.

## Variables detectadas

- flow_ACK_flag_count
- flow_pkts_payload.tot
- idle.tot

## Interpretación

Estas variables presentan una relación muy fuerte con otras características del conjunto de datos.

Una correlación elevada puede indicar que dos variables están proporcionando información similar al modelo.

Por el momento no se recomienda eliminar automáticamente estas variables sin realizar pruebas de desempeño durante la fase de modelado.

---

# Revisión de Variables Seleccionadas

Se verificó que las variables consideradas importantes durante el EDA continúan presentes en el dataset final.

Entre ellas se encuentran las variables relacionadas con los flags TCP:

- flow_FIN_flag_count
- flow_SYN_flag_count
- flow_RST_flag_count
- flow_ACK_flag_count
- fwd_PSH_flag_count
- fwd_URG_flag_count
- flow_CWR_flag_count

Estas variables fueron identificadas previamente como potencialmente útiles para diferenciar tráfico normal de tráfico malicioso
y continúan disponibles para el entrenamiento de modelos.

También permanecen variables relacionadas con:

- Duración del flujo.
- Cantidad de paquetes.
- Payload.
- Ventanas TCP.
- Variables temporales.
- Variables de actividad e inactividad.

---

# Variables Potencialmente Leaky

Durante la revisión se identificaron algunas variables que podrían introducir fuga de información (Data Leakage):

- id.orig_p
- id.resp_p
- proto
- service

Estas variables pueden contener información muy relacionada con determinadas clases de ataque y producir métricas artificialmente elevadas.

Por esta razón se recomienda realizar pruebas comparando:

- Modelo con variables leaky.
- Modelo sin variables leaky.

De esta forma será posible evaluar el impacto real de estas características sobre el rendimiento del modelo.

---

# Respuestas 

## ¿Persisten variables redundantes?

Sí.

Se identificaron algunas variables con correlaciones superiores a 0.95:

- flow_ACK_flag_count
- flow_pkts_payload.tot
- idle.tot

No obstante, la cantidad de variables redundantes 

---

## ¿Existe alguna variable que todavía debería eliminarse?

No se encontraron columnas constantes ni problemas graves de redundancia.

Las variables altamente correlacionadas deberán evaluarse durante el modelado antes de tomar una decisión definitiva sobre su eliminación.

Asimismo, las variables:

- id.orig_p
- id.resp_p
- proto
- service

deben revisarse cuidadosamente debido al posible riesgo de fuga de información.

---

## ¿Las decisiones tomadas durante el EDA fueron suficientes?

Sí.

Los resultados muestran que:

- No existen variables constantes.
- La mayoría de las variables redundantes fueron eliminadas.
- Las variables consideradas importantes durante el EDA permanecen disponibles.

Por lo tanto, las decisiones tomadas durante las etapas de exploración y limpieza fueron adecuadas
para preparar el dataset para la fase de entrenamiento de modelos.

---

# Conclusiones

La validación realizada confirma que el dataset limpio cumple con los requisitos necesarios para iniciar la etapa de modelado.

No se encontraron variables constantes y únicamente permanecen unas pocas variables con correlaciones elevadas,
las cuales deberán evaluarse experimentalmente en futuras iteraciones.

Las variables seleccionadas durante el análisis exploratorio continúan presentes y conservan su utilidad potencial para la detección de ataques IoT.

Finalmente, se recomienda prestar especial atención a las variables potencialmente leaky durante la construcción de los modelos para evitar resultados artificialmente optimistas y garantizar una evaluación realista del desempeño del sistema.
