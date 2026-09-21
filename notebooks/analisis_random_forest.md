# ANÁLISIS Y OPTIMIZACIÓN DEL MODELO RANDOM FOREST


1. OBJETIVO
--------------------------------------------------------------------------------
El objetivo de este experimento es optimizar el modelo Random Forest utilizado 
para la detección binaria de tráfico normal y tráfico de ataque en el dataset 
del proyecto. El análisis busca comparar el modelo inicial con una versión 
optimizada mediante validación cruzada, estudiar la importancia de las 
características y evaluar específicamente el efecto de la variable flow_duration.

Las preguntas principales a responder son:
- ¿El Random Forest optimizado mejora respecto al inicial?
- ¿Qué hiperparámetros tienen mayor impacto?
- ¿Cuáles son las características más importantes?
- ¿flow_duration domina el Random Forest?
- ¿Qué sucede cuando se elimina flow_duration?
- ¿Qué impacto tiene esta eliminación sobre el rendimiento?


2. PREPARACIÓN DE LOS DATOS
--------------------------------------------------------------------------------
Se utilizó el archivo dataset_limpio.csv, que cuenta con 62.338 registros y 
37 columnas.

La variable objetivo binaria y_bin se construyó a partir de Attack_type, 
considerando tres clases como normales (MQTT_Publish, Thing_Speak, Wipro_bulb). 
La codificación fue:
- 0 = Normal
- 1 = Ataque

DISTRIBUCIÓN DE LA VARIABLE OBJETIVO:
--------------------------------------------------
Clase        Registros       Porcentaje
--------------------------------------------------
Ataque       50.323          80,73 %
Normal       12.015          19,27 %
Total        62.338          100,00 %
--------------------------------------------------

Para evitar posibles fugas de información (data leakage), se excluyeron de X 
las variables: id.orig_p, id.resp_p, proto y service.
También se eliminó Attack_type, ya que fue utilizada exclusivamente para 
construir el objetivo binario.


3. CONFIGURACIÓN EXPERIMENTAL
--------------------------------------------------------------------------------
Se utilizó la misma partición para todos los experimentos:
- 80 % Train
- 20 % Test
- stratify=y_bin
- random_state=42

El conjunto Test se mantuvo estrictamente reservado para la evaluación final. 
La selección de hiperparámetros se realizó exclusivamente sobre Train mediante 
5-fold Stratified Cross Validation.

El modelo Random Forest utilizó:
- class_weight="balanced"
- random_state=42
- n_jobs=-1

No se aplicó escalado de variables dado que Random Forest es un algoritmo basado 
en árboles de decisión, insensible a las transformaciones monótonas de escala.


4. RANDOM FOREST INICIAL
--------------------------------------------------------------------------------
El modelo inicial se definió con la configuración por defecto de la librería:

RandomForestClassifier(
    n_estimators=100,
    class_weight="balanced",
    random_state=42,
    n_jobs=-1
)

RESULTADOS EN TEST (MODELO INICIAL):
- Accuracy:                  0,997514
- Precision:                 0,998213
- Recall:                    0,998708
- F1-score:                  0,998460
- F1-Macro:                  0,996002
- PR-AUC:                    0,999980
- Train time:                2,692787 s
- Predict time:              0,070842 s
- Falsos Negativos (FN):     13
- Falsos Positivos (FP):     18

El modelo inicial presenta un rendimiento sobresaliente desde el inicio, 
registrando únicamente 13 Falsos Negativos y 18 Falsos Positivos sobre el 
conjunto de prueba.


5. OPTIMIZACIÓN CON GRIDSEARCHCV
--------------------------------------------------------------------------------
Se implementó GridSearchCV para evaluar sistemáticamente distintas combinaciones 
de hiperparámetros sobre el conjunto de entrenamiento.

ESPACIO DE BÚSQUEDA:
param_grid = {
    "n_estimators": [100, 200],
    "max_depth": [None, 15, 25],
    "min_samples_split": [2, 10],
    "min_samples_leaf": [1, 4],
    "max_features": ["sqrt", None]
}

El grid contempla 48 combinaciones que, multiplicadas por los 5 folds, representan 
240 ajustes de modelo durante la validación cruzada. La métrica guía para la 
selección fue F1-Macro.

MEJOR COMBINACIÓN ENCONTRADA:
- max_depth: None
- max_features: sqrt
- min_samples_leaf: 1
- min_samples_split: 2
- n_estimators: 100

Mejor F1-Macro promedio en Cross Validation: 0,996676


6. IMPACTO DE LOS HIPERPARÁMETROS
--------------------------------------------------------------------------------
El análisis de las 10 mejores combinaciones del grid muestra un patrón claro: 
las configuraciones con min_samples_split=2 y min_samples_leaf=1 concentran los 
mejores resultados de rendimiento.

TOP 10 DE COMBINACIONES EVALUADAS:
----------------------------------------------------------------------------------------------------
n_estimators   max_depth   min_samples_split   min_samples_leaf   max_features   F1-Macro CV   Std
----------------------------------------------------------------------------------------------------
100            None        2                   1                  sqrt           0,996676      0,000218
100            25          2                   1                  sqrt           0,996676      0,000218
200            25          2                   1                  sqrt           0,996610      0,000269
200            None        2                   1                  sqrt           0,996578      0,000237
100            15          2                   1                  sqrt           0,996354      0,000298
200            15          2                   1                  sqrt           0,996354      0,000363
200            None        2                   1                  None           0,995710      0,000639
200            25          2                   1                  None           0,995710      0,000639
200            15          2                   1                  None           0,995613      0,000692
100            25          2                   1                  None           0,995548      0,000482
----------------------------------------------------------------------------------------------------

Observaciones Clave:
1. max_features: Es el parámetro más determinante. Usar "sqrt" supera 
   consistentemente a None (que utiliza todas las variables), haciendo 
   descender el F1-Macro CV a la franja de 0,9955–0,9957.
2. max_depth: La diferencia entre None y 25 es prácticamente imperceptible en 
   los mejores modelos.
3. n_estimators: Genera variaciones mínimas; 100 árboles demostraron ser 
   suficientes sin necesidad de subir a 200.


7. COMPARACIÓN: RANDOM FOREST INICIAL VS. OPTIMIZADO
--------------------------------------------------------------------------------
Métrica                     RF Inicial         RF Optimizado
--------------------------------------------------------------------------------
Accuracy                    0,997514           0,997514
Precision                   0,998213           0,998213
Recall                      0,998708           0,998708
F1-score                    0,998460           0,998460
F1-Macro                    0,996002           0,996002
PR-AUC                      0,999980           0,999980
Falsos Negativos (FN)       13                 13
Falsos Positivos (FP)       18                 18
--------------------------------------------------------------------------------

Interpretación:
El Random Forest optimizado no presenta una mejora en el conjunto Test respecto 
al modelo inicial debido a que GridSearchCV eligió exactamente la misma 
combinación de hiperparámetros que el modelo base. Esto demuestra que la 
configuración predeterminada ya era la óptima dentro del espacio de búsqueda.


8. COSTO COMPUTACIONAL DE LA OPTIMIZACIÓN
--------------------------------------------------------------------------------
- Tiempo de GridSearchCV: 2643,59 segundos (aprox. 44,06 minutos).
- Tiempo de entrenamiento del RF Inicial: 2,69 segundos.

Nota: El costo del GridSearchCV abarca las 48 combinaciones evaluadas en 5 folds 
(240 ajustes en total). Este experimento demuestra que la búsqueda masiva en 
malla resultó computacionalmente costosa sin generar incrementos en el rendimiento 
final sobre Test.


9. IMPORTANCIA DE VARIABLES EN EL RANDOM FOREST OPTIMIZADO
--------------------------------------------------------------------------------
Extraído mediante el atributo feature_importances_:

Posición    Variable                      Importancia Absoluta    Porcentaje
--------------------------------------------------------------------------------
1           flow_duration                 0,198252                19,83 %
2           fwd_pkts_payload.avg          0,102805                10,28 %
3           bwd_pkts_payload.avg          0,101339                10,13 %
4           fwd_data_pkts_tot             0,092109                9,21 %
5           fwd_pkts_tot                  0,078496                7,85 %
6           flow_pkts_per_sec             0,053210                5,32 %
7           bwd_pkts_tot                  0,051543                5,15 %
8           fwd_subflow_bytes             0,040398                4,04 %
9           flow_ACK_flag_count           0,040262                4,03 %
10          flow_pkts_payload.min         0,024520                2,45 %
11          fwd_PSH_flag_count            0,021617                2,16 %
12          fwd_iat.std                   0,021001                2,10 %
13          bwd_init_window_size          0,020473                2,05 %
14          bwd_pkts_payload.min          0,019722                1,97 %
15          flow_pkts_payload.tot         0,018571                1,86 %
--------------------------------------------------------------------------------

Aunque flow_duration es la más relevante (19,83 %), el modelo reparte 
sustancialmente la decisión en otras variables de volumen, carga útil y banderas 
del protocolo.


10. COMPARACIÓN CON EL DECISION TREE
--------------------------------------------------------------------------------
- Decision Tree (Análisis previo): flow_duration concentraba el 90,94 % de la 
  importancia.
- Random Forest Optimizado: flow_duration representa el 19,83 % de la importancia.

Esta diferencia evidencia la ventaja del ensamble: mientras un árbol individual 
genera una dependencia casi exclusiva de una sola variable, Random Forest logra 
una representación mucho más distribuida y robusta.


11. EXPERIMENTO ELIMINANDO FLOW_DURATION
--------------------------------------------------------------------------------
Se entrenó un nuevo modelo excluyendo flow_duration manteniendo idénticas las 
demás condiciones: división 80/20, random_state=42, estratificación e 
hiperparámetros óptimos.

COMPARATIVA: CON VS. SIN FLOW_DURATION
----------------------------------------------------------------------------------------------------
Métrica                     RF Optimizado (Con FD)     RF Optimizado (Sin FD)     Cambio Absoluto
----------------------------------------------------------------------------------------------------
Accuracy                    0,997514                   0,997754                   +0,000240
Precision                   0,998213                   0,998213                   =
Recall                      0,998708                   0,999006                   +0,000298
F1-score                    0,998460                   0,998610                   +0,000150
F1-Macro                    0,996002                   0,996387                   +0,000385
PR-AUC                      0,999980                   0,999867                   -0,000113
FN                          13                         10                         -23,1 %
FP                          18                         18                         =
Train time                  2,692787 s                 2,179621 s                 -0,513166 s
Predict time                0,102726 s                 0,069146 s                 -0,033580 s
----------------------------------------------------------------------------------------------------


12. INTERPRETACIÓN DEL EXPERIMENTO SIN FLOW_DURATION
--------------------------------------------------------------------------------
A pesar de que flow_duration era la característica con mayor peso individual, su 
eliminación no degrada la capacidad general del modelo. Al contrario:
1. Reducción de errores graves: Los Falsos Negativos cayeron de 13 a 10 (reducción 
   del 23,1 % en amenazas no detectadas).
2. Leve mejora general: Subieron las métricas globales de Accuracy, Recall, F1 y 
   F1-Macro.
3. Leve caída en PR-AUC: El área bajo la curva cayó ligeramente de 0,999980 a 
   0,999867, mostrando un impacto menor en el comportamiento probabilístico global.

El resto de las características de flujo de red compensan ampliamente la ausencia 
de la duración.


13. RESPUESTAS A LAS PREGUNTAS DEL TRABAJO
--------------------------------------------------------------------------------
1. ¿El Random Forest optimizado mejora respecto al inicial?
   No en el conjunto de prueba. Las métricas fueron exactamente idénticas debido 
   a que GridSearchCV volvió a seleccionar la misma combinación que ya tenía 
   el modelo base.

2. ¿Qué hiperparámetros tienen mayor impacto?
   max_features demostró ser el más influyente, donde la opción "sqrt" superó 
   claramente a None. min_samples_split=2 y min_samples_leaf=1 también se mantuvieron 
   constantes en el TOP 10.

3. ¿Cuáles son las características más importantes?
   La más relevante es flow_duration (19,83 %), seguida por fwd_pkts_payload.avg 
   (10,28 %) y bwd_pkts_payload.avg (10,13 %).

4. ¿flow_duration domina el Random Forest?
   No. Aunque lidera la lista, su importancia es del 19,83 %, a diferencia del 
   árbol de decisión donde concentraba el 90,94 %.

5. ¿Qué pasa al eliminar flow_duration?
   El modelo mantiene un rendimiento excepcional e incluso reduce sus Falsos 
   Negativos de 13 a 10, mejorando ligeramente su F1-Macro.

6. ¿Vale la pena mantener flow_duration?
   No es indispensable. Aunque aporta información valiosa, el experimento de 
   remoción demostró que el Random Forest puede funcionar de forma equivalente o 
   ligeramente mejor sin ella, evitando depender de una métrica que puede ser 
   alterada en entornos reales de red.


14. CONCLUSIONES
--------------------------------------------------------------------------------
- Rendimiento Excepcional: Random Forest alcanza un desempeño cercano a la 
  perfección (Accuracy > 99,75%, F1 > 99,84%) en la tarea de clasificación binaria.
- Costo-Beneficio de GridSearchCV: La optimización en malla consumió 44 minutos 
  para seleccionar los mismos parámetros iniciales, indicando que el modelo 
  base era sumamente adecuado.
- Robustez de Ensamble: A diferencia de modelos individuales de árbol, Random 
  Forest dispersa el conocimiento entre múltiples variables, eliminando puntos 
  únicos de falla conceptual dentro del modelo.


15. CONSIDERACIONES PARA LA ENTREGA
--------------------------------------------------------------------------------
- Notebook implementado: notebooks/10_analisis_random_forest.ipynb
- Documento de análisis: docs/analisis_random_forest.md
================================================================================
