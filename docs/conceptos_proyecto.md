# Guía de conceptos del proyecto — Detección de anomalías RT-IoT2022

> **Para quién es esto:** cualquier integrante del equipo, sin importar su nivel.
> El objetivo es que entiendas **el porqué** de cada decisión que tomamos, con
> explicaciones sencillas y analogías. No hace falta saber mucho de redes ni de
> machine learning para leerlo. Es el "manual de a bordo" del proyecto.

Índice:
1. [El dataset RT-IoT2022](#1-el-dataset-rt-iot2022)
2. [La historia de Amazon-Alexa](#2-la-historia-de-amazon-alexa-paper-vs-archivo)
3. [La etiqueta binaria (normal vs ataque)](#3-la-etiqueta-binaria-normal-vs-ataque)
4. [El desbalance de clases](#4-el-desbalance-de-clases)
5. [Data leakage (fuga de información)](#5-data-leakage-fuga-de-información)
6. [`fit` vs `transform` y la analogía de la balanza](#6-fit-vs-transform-y-la-analogía-de-la-balanza)
7. [El orden correcto para evitar leakage](#7-el-orden-correcto-para-evitar-leakage)
8. [Variables categóricas y escalado](#8-variables-categóricas-y-escalado)
9. [Correlación: Spearman vs Pearson](#9-correlación-spearman-vs-pearson)
10. [Redundancia y representantes](#10-redundancia-y-representantes)
11. [Familias casi-constantes y la regla del 1%](#11-familias-casi-constantes-y-la-regla-del-1)
12. [Features "leaky" (puertos, service, proto)](#12-features-leaky-puertos-service-proto)
13. [`fwd` vs `bwd` (direcciones del tráfico)](#13-fwd-vs-bwd-direcciones-del-tráfico)
14. [Métricas cuando hay desbalance](#14-métricas-cuando-hay-desbalance)
15. [Explicación del notebook 03_matriz_correlacion](#15-explicación-del-notebook-03_matriz_correlacionipynb)
16. [Glosario](#16-glosario)

---

## 1. El dataset RT-IoT2022

Es un conjunto de datos de **tráfico de red** de dispositivos IoT (Internet de las
Cosas: bombillos inteligentes, sensores, Alexa, etc.), capturado en un laboratorio
junto con **ataques simulados** contra esos dispositivos.

- **123,117 filas**: cada fila es un **flujo de red** (una "conversación" entre dos
  máquinas: un resumen de los paquetes que se enviaron).
- **84 columnas**: características de cada flujo (cuántos paquetes, cuántos bytes,
  cuánto duró, qué banderas TCP llevaba, etc.) + la etiqueta `Attack_type`.
- **Sin valores faltantes** (ningún hueco que rellenar).

La columna que queremos predecir es **`Attack_type`**, que tiene **12 valores**:

| Clase | Filas | ¿Qué es? |
|---|---|---|
| DOS_SYN_Hping | 94,659 | Ataque de denegación de servicio (inundación SYN) |
| Thing_Speak | 8,108 | **Normal** (dispositivo IoT legítimo) |
| ARP_poisioning | 7,750 | Ataque (envenenamiento ARP) *(sí, está mal escrito en el dataset)* |
| MQTT_Publish | 4,146 | **Normal** (sensor publicando por MQTT) |
| NMAP_UDP_SCAN | 2,590 | Ataque (escaneo de puertos) |
| NMAP_XMAS_TREE_SCAN | 2,010 | Ataque (escaneo) |
| NMAP_OS_DETECTION | 2,000 | Ataque (escaneo) |
| NMAP_TCP_scan | 1,002 | Ataque (escaneo) |
| DDOS_Slowloris | 534 | Ataque (denegación de servicio lenta) |
| Wipro_bulb | 253 | **Normal** (bombillo inteligente) |
| Metasploit_Brute_Force_SSH | 37 | Ataque (fuerza bruta de contraseña SSH) |
| NMAP_FIN_SCAN | 28 | Ataque (escaneo) |

---

## 2. La historia de Amazon-Alexa (paper vs archivo)

Esto es importante y es una **gran lección de método**.

La página oficial del dataset (UCI) **menciona** una clase `Amazon-Alexa` con **86,842
filas** de tráfico normal. Pero cuando abrimos **el archivo real** y contamos las clases
(`value_counts()`), **Amazon-Alexa NO aparece**. Las 12 clases reales suman exactamente
123,117.

¿Por qué la contradicción? Hicimos la cuenta:

```
Nuestro archivo (12 clases, sin Alexa) ........ 123,117
+ Amazon-Alexa (lo que dice el paper) .........  86,842
                                              ----------
Dataset original completo .....................  209,959 filas
```

O sea: **el experimento original tenía ~209,959 flujos, pero la versión que publicaron
en UCI (la que tenemos) se quedó en 123,117 — justo el total SIN el tráfico de Alexa.**
El tráfico de Amazon-Alexa se publicó como un **dataset aparte**. La descripción de la
página quedó escrita según el experimento completo, por eso menciona Alexa aunque el
archivo descargable no lo traiga.

**Decisión del equipo:** usar la **versión oficial de UCI (123,117 filas, sin Alexa)**,
porque es la que citan todos los papers → nuestros resultados son comparables y
reproducibles.

> 🎓 **La lección:** cuando la documentación y el archivo se contradicen, **gana el
> archivo**. Siempre corre `value_counts()` y confía en eso, no en lo que diga la web.

**Consecuencia importante:** como Alexa era la clase normal más grande, al excluirla el
dataset queda **muy desbalanceado** (ver siguiente sección). Con Alexa habría sido casi
50/50; sin Alexa, lo normal es apenas el 10%.

---

## 3. La etiqueta binaria (normal vs ataque)

El dataset **no trae** una columna que diga "normal" o "ataque". Solo trae `Attack_type`
con los 12 nombres. El comportamiento normal está **disfrazado con el nombre del
dispositivo** que lo generó:

- `MQTT_Publish`, `Thing_Speak`, `Wipro_bulb` → tráfico **normal** (dispositivos legítimos).
- Las otras 9 → **ataques**.

Por eso construimos nosotros la etiqueta binaria:

```python
NORMAL_CLASSES = {"MQTT_Publish", "Thing_Speak", "Wipro_bulb"}
y_bin = (~df["Attack_type"].isin(NORMAL_CLASSES)).astype(int)   # 1 = ataque, 0 = normal
```

> **Importante:** guardamos **además** la etiqueta original `Attack_type` (multiclase),
> porque más adelante queremos no solo decir "es ataque" sino **qué tipo** de ataque.
> No la sobreescribimos.

---

## 4. El desbalance de clases

"Desbalance" significa que una clase tiene **muchísimas más filas** que la otra:

| Clase | Filas | % |
|---|---|---|
| Ataque (9 clases) | 110,610 | **89.8%** |
| Normal (3 clases) | 12,507 | **10.2%** |

Hay casi **9 ataques por cada 1 normal**. ¿Por qué importa? Porque un modelo perezoso
puede decir "**todo es ataque**" y acertar el 89.8% de las veces sin aprender nada útil.
Por eso necesitamos cuidados especiales (métricas adecuadas, ver sección 14).

**Tres formas de lidiar con el desbalance:**

1. **`class_weight="balanced"`** (la que usamos para empezar): no toca los datos; solo le
   dice al modelo "préstale **más atención** a la clase rara (normal)". Simple y seguro.
2. **Undersampling (submuestreo):** **botar** filas de la clase grande hasta igualar.
   Ejemplo: bajar los 110,610 ataques a ~12,507. *Problema:* tiras datos a la basura.
3. **Oversampling / SMOTE:** **inventar** filas sintéticas de la clase pequeña hasta
   igualar. Ejemplo: subir los 12,507 normales a ~110,610. *Problema:* son datos
   "fabricados", no reales.

> **Decisión:** arrancamos con `class_weight` (sin tocar los datos). SMOTE/undersampling
> quedan como **experimentos posteriores**, no como punto de partida.

---

## 5. Data leakage (fuga de información)

**Leakage** = cuando el modelo "ve", mientras entrena, información que en la vida real
**no tendría** al momento de predecir. El resultado es traicionero: métricas
**espectaculares** en tu computador, pero el modelo **fracasa** con datos reales.

**Analogía del examen:** imagina que el examen final se te filtra mientras estudias.
Sacarás 5.0, pero ese 5.0 **no mide si aprendiste** — mide que viste las respuestas.

En nuestro proyecto el leakage puede entrar por dos caminos:

1. **Identificadores** (índice, timestamps): no aportan generalización; el modelo podría
   "memorizar" filas.
2. **Variables que separan demasiado bien**: por ejemplo el puerto destino o el `service`.
   El SSH (puerto 22) puede coincidir casi 1:1 con el ataque de fuerza bruta. Usarlas
   **infla las métricas** y no refleja un IDS (sistema de detección) real.

> 🚨 **Señal de alarma:** si el primer modelo simple ya da un **F1 ≈ 1.0** (casi
> perfecto), casi seguro hay leakage. Un baseline demasiado bueno es sospechoso, no
> motivo de celebración.

---

## 6. `fit` vs `transform` y la analogía de la balanza

Esta es **la confusión más común** al empezar. Aclarémosla bien.

Primero, dos tipos de operaciones distintas:

| | **Limpieza** | **Transformación** |
|---|---|---|
| Ejemplos | Botar columnas, quitar duplicados | Escalar números, codificar categóricas, rellenar nulos |
| ¿"Aprende" algo de los datos? | **No** (botar una columna es botar una columna) | **Sí** (para escalar necesita calcular media y desviación) |

Solo lo que **aprende** puede causar leakage. Por eso la limpieza es inofensiva, pero la
transformación hay que hacerla con cuidado.

**Lo que confunde:** ¿se escalan todos los datos? **Sí, todos** — entrenamiento Y prueba.
Los dos terminan escalados. Lo especial es **de dónde salen los números** para escalar.

Para escalar una columna se usa: `valor_escalado = (valor − media) / desviación`. La
pregunta clave es: **¿la media la calculo con el train solo, o con train + test juntos?**

- **`fit`** = *calcular* la media y la desviación. → **SOLO con el train.**
- **`transform`** = *aplicar* la fórmula. → a **train Y a test**, a los dos.

```python
scaler.fit(X_train)                    # 1. CALCULA media/std mirando SOLO train
X_train = scaler.transform(X_train)    # 2. ESCALA el train
X_test  = scaler.transform(X_test)     # 3. ESCALA el test (con la media/std del train)
```

> 🎯 **La analogía de la balanza:** el train es donde **calibras la balanza** (la pones en
> cero, defines cuánto pesa un kilo). Una vez calibrada, **pesas todo** con ella: las
> frutas del train y las del test. **No recalibras** con las frutas del test — usas la
> misma calibración. Si recalibraras con el test, sería "hacer trampa", porque el test ya
> no sería una medición independiente.

Resumen en una frase: **se escala todo, pero el "patrón de medida" (media, std,
categorías) se saca solo del train y se aplica igual al test.**

---

## 7. El orden correcto para evitar leakage

El preprocesamiento debe seguir **este orden** o el leakage se cuela:

1. **Split primero**: separa train (80%) y test (20%), **estratificado** (que la
   proporción normal/ataque sea igual en ambos lados). El test se **guarda en una caja
   fuerte** y no se mira hasta el reporte final.
2. **`fit` solo con train, `transform` a ambos** (ver sección 6).
3. **Resampling (SMOTE/undersampling) solo sobre train**, nunca antes del split ni sobre
   el test. (El test debe quedar desbalanceado, porque la realidad llega desbalanceada.)

> **Por qué usamos `Pipeline` + `ColumnTransformer` de scikit-learn:** hacer esto a mano
> es fácil de equivocar (un día se te olvida y haces `fit` sobre el test). El `Pipeline`
> lo hace bien **por construcción**: al llamar `pipeline.fit(X_train)` ajusta todo solo
> con train; al llamar `pipeline.predict(X_test)` solo transforma. No te deja filtrar.

---

## 8. Variables categóricas y escalado

- **Categóricas** (texto, como `proto`=tcp/udp o `service`=mqtt/ssh): los modelos no
  entienden texto, hay que convertirlas a números. Usamos **one-hot encoding**: una
  columna 0/1 por cada categoría (ej. `proto_tcp`, `proto_udp`).
- **Numéricas:** se **escalan** para que todas estén en rangos parecidos. Esto importa
  para modelos lineales o basados en distancia. Tipos de escalado:
  - `StandardScaler`: resta la media y divide por la desviación. Bueno si la variable es
    más o menos simétrica.
  - `RobustScaler` o `log1p`: mejores cuando hay **muchos outliers** (valores extremos),
    como pasa en tráfico de red.
- **Los árboles (RandomForest, GradientBoosting) NO necesitan escalado** — funcionan
  igual con o sin él, porque solo comparan "¿es mayor o menor que X?".

> **Conclusión:** la elección del escalado **depende del modelo**. No hay uno "correcto"
> para todo.

---

## 9. Correlación: Spearman vs Pearson

**Correlación** mide qué tan relacionadas están dos variables (de -1 a +1). Cerca de +1 o
-1 = muy relacionadas; cerca de 0 = independientes.

Hay dos formas de calcularla:

| | **Pearson** | **Spearman** |
|---|---|---|
| Qué detecta | Relaciones **lineales** (línea recta) | Relaciones **monótonas** (suben/bajan juntas, aunque sea curvo) |
| Sensible a outliers | **Sí** (mucho) | **No** (usa el orden, no el valor) |
| ¿Bueno para tráfico de red? | Menos | **Sí** ✅ |

> **Por qué usamos Spearman:** el tráfico de red está **muy sesgado** (muchos ceros, unos
> pocos valores enormes) y lleno de outliers. Spearman es robusto a eso. Calculamos
> **ambos y los comparamos**: donde difieren mucho, es señal de una relación **no lineal**
> o dominada por outliers.

También hay dos usos distintos de la correlación, ojo no confundirlos:
- **Entre features** (lo que hicimos): detectar **redundancia** (variables que dicen lo
  mismo) → ver sección 10.
- **Con el target**: ver qué variables **discriminan** normal vs ataque. Esto lo dejamos
  para después porque tiene riesgo de leakage (una variable que correlaciona "demasiado"
  con el target es sospechosa).

---

## 10. Redundancia y representantes

Cuando dos variables están correlacionadas **> 0.9**, son casi **clones**: dicen casi lo
mismo. Conservar las dos no te da "más información para analizar" — te da **dos columnas
casi idénticas**. Y eso **perjudica** el análisis por dos razones:

1. **Multicolinealidad** (en modelos lineales): si dos variables dicen lo mismo, el modelo
   no sabe a cuál darle el peso y reparte el coeficiente de forma **inestable** → los
   coeficientes se vuelven imposibles de interpretar.
2. **Dilución de importancia** (en RandomForest): la importancia se **reparte a la mitad**
   entre las dos correlacionadas → **ambas parecen poco importantes** aunque la señal sea
   fuerte. Conservar duplicados **sabotea** justo el análisis de importancia.

**Qué hicimos:** agrupamos las variables muy correlacionadas (clustering jerárquico con
Spearman, corte en |ρ|>0.9) y de cada grupo conservamos **un representante**. Regla para
elegirlo:

> **no-leaky → más denso (menos ceros) → más interpretable**

Es decir: nunca elegir una variable leaky; entre las demás, preferir la que tiene menos
ceros (más información usable); y a igualdad, la de nombre más entendible.

**¿Cuándo conservar 2 representantes?** Solo cuando el grupo es "suelto" (correlación no
tan alta, < 0.95) **y** el segundo representante captura una **faceta distinta** (ej. uno
mide *volumen* de paquetes y el otro *payload*). Si el grupo son clones perfectos
(ρ=0.99+), conservar 2 es duplicar por duplicar — no se hace.

> **Idea clave:** podar redundancia **no borra análisis**. Las 84 columnas siguen en el
> notebook de correlación (ahí es donde analizas). La poda solo decide qué entra **al
> modelo**. Y no es permanente: siempre puedes traer una columna de vuelta.

---

## 11. Familias casi-constantes y la regla del 1%

Algunas columnas son **casi todo ceros** (99% o más). El reflejo sería botarlas por
"inútiles", pero ojo: **99% ceros no significa siempre inútil**.

> **La regla del 1%:** si ese 1% de valores **no-cero** se concentra en una clase
> concreta, puede ser una **señal rara pero valiosa** (un evento poco frecuente que
> delata un tipo de tráfico). No botar por reflejo — **investigar primero**.

Lo verificamos con datos. Mirando dónde caen los valores no-cero de cada familia:

| Familia | % no-cero | ¿Dónde se concentra? | Decisión |
|---|---|---|---|
| `idle.*` | 4.6% | **72% normal** (MQTT) | **Conservar** — señal de tráfico normal |
| `fwd_bulk.*` | 0.2% | **81% ataque** (ARP) | **Conservar 1** — marcador raro de ARP |
| `flow_CWR/ECE` | 0.06% | **100% normal** | **Conservar 1** (`flow_CWR`) — marcador normal |
| `bwd_bulk.*` | 0.9% | mezclado (46/54) | **Botar entera** — sin señal clara |

Lo único que se bota **siempre** sin discusión es `bwd_URG_flag_count`, que es **100%
ceros** (constante de verdad: no aporta absolutamente nada).

---

## 12. Features "leaky" (puertos, service, proto)

Cuatro columnas tienen **riesgo de leakage** porque podrían codificar el ataque casi 1:1:

- `id.orig_p` y `id.resp_p` (puertos de origen y destino)
- `service` (servicio: mqtt, ssh, dns…)
- `proto` (protocolo: tcp, udp…)

**Cómo las manejamos:** NO las podamos por redundancia. En su lugar, el preprocesador
tiene un **interruptor** `include_leaky`:
- `include_leaky=False` → el modelo entrena **sin** ellas (escenario realista de IDS).
- `include_leaky=True` → el modelo entrena **con** ellas.

Corremos el baseline en las **dos variantes y comparamos**. Si las métricas se disparan
"con leaky", es la prueba de que esas variables están haciendo trampa.

> **Dato curioso que encontramos:** `id.resp_p` (puerto destino) correlaciona >0.9 con
> `fwd_pkts_payload.avg`. O sea, el puerto "predice" el tamaño del payload → confirma que
> es una variable leaky y hay que reportarlo.

---

## 13. `fwd` vs `bwd` (direcciones del tráfico)

Muchas columnas vienen en pareja `fwd_` y `bwd_`:
- **`fwd`** (forward) = origen → destino (lo que **envía** quien inició la conexión).
- **`bwd`** (backward) = destino → origen (lo que **responde** el otro lado).

¿Por qué importa? Porque la **asimetría** delata ataques. Un escaneo o un DDoS suele ser
**muy asimétrico**: el atacante manda muchísimo (`fwd` alto) y casi no recibe respuesta
(`bwd` casi cero). Por eso muchas columnas `bwd_*` son casi todo ceros en el tráfico de
ataque — y eso, en sí mismo, es información.

---

## 14. Métricas cuando hay desbalance

Con desbalance, la **accuracy (exactitud) engaña**. Si el 89.8% es ataque, un modelo que
diga "todo ataque" tiene 89.8% de accuracy… sin servir para nada. Por eso usamos otras:

- **Matriz de confusión:** tabla de aciertos/errores (verdaderos positivos, falsos
  positivos, etc.). La foto completa.
- **Precision (precisión):** de lo que el modelo marcó como ataque, ¿cuánto era ataque de
  verdad? (evita falsas alarmas).
- **Recall (sensibilidad):** de todos los ataques reales, ¿cuántos detectó? (evita
  ataques que se escapan).
- **F1:** balance entre precision y recall (un solo número resumen).
- **PR-AUC:** área bajo la curva precision-recall; resume el desempeño en todos los
  umbrales. Buena con desbalance.

> 🎯 **En un IDS (detector de intrusiones), el error caro es el FALSO NEGATIVO**: un
> ataque que **no** se detecta. Por eso priorizamos **recall** de la clase ataque (atrapar
> todos los ataques posibles) sin destruir la precision (sin llenar de falsas alarmas).

**Piso de referencia:** usamos un `DummyClassifier(strategy="most_frequent")` (que siempre
predice la clase mayoritaria) para saber qué accuracy es "trivial". Si nuestro modelo no
le gana a eso, no sirve.

---

## 15. Explicación del notebook `03_matriz_correlacion.ipynb`

Este es el **primer notebook** del puente Fase 2 → Fase 3. Su único objetivo:
**detectar redundancia entre features** y dar una **vista global** de cómo se relacionan.
**No** analiza la relación con el target (eso se difiere). Va celda por celda así:

| Sección | Qué hace | Qué produce / cómo leerlo |
|---|---|---|
| **1. Carga** | Abre `data/rt-iot2022.zip`, lee `RT_IOT2022` con `index_col=0` | Confirma shape `(123117, 84)` |
| **2. Preparar numéricas** | Toma solo columnas numéricas, quita `Attack_type` y la constante `bwd_URG_flag_count` | ~78 columnas listas para correlacionar |
| **3. Matrices** | Calcula correlación **Spearman** y **Pearson** | Dos matrices ~78×78 |
| **4. Heatmaps** | Dibuja ambos mapas de calor (con máscara de triángulo) | Vista global: bloques rojos = grupos redundantes |
| **5. Clustermap** | Reordena las variables agrupando las parecidas | Se ven "cuadros" de familias (iat, payload, header…) |
| **6. Pares \|ρ\|>0.9** | Lista todas las parejas muy correlacionadas, en Spearman y Pearson | Tabla ordenada; aquí están los "clones" |
| **7. Spearman vs Pearson** | Marca dónde difieren mucho (\|Δ\|>0.2) | Señala relaciones no lineales / outliers |
| **8. Grupos y representantes** | Agrupa (clustering) y elige 1–2 representantes por grupo con la regla **no-leaky → denso → interpretable** | La lista `REDUNDANT_COLS_DROP`: qué columnas botar |
| **9. Familias casi-constantes** | Aplica la **regla del 1%**: ve dónde caen los no-cero | Decide conservar/botar `idle`, `fwd_bulk`, `bwd_bulk`, `CWR/ECE` |
| **10. Resumen** | Imprime las listas finales: conservar / botar / leaky | Es el insumo para `src/data_prep.py` |

**Cómo leer el resultado:** al final tendrás tres listas claras — las features que se
conservan, las que se botan por redundancia, y las leaky (que se manejan aparte). Esas
listas son **decisiones documentadas**, no acciones automáticas: alimentan el módulo de
preparación reproducible.

> Este notebook **confirma y extiende** lo que ya había visto Juan (ej.
> `fwd_iat.tot ↔ flow_iat.tot` = 0.9997), pero de forma **completa y en un solo lugar**,
> con los dos métodos de correlación.

---

## 16. Glosario

- **Flujo (flow):** resumen de una "conversación" de red entre dos máquinas (no un paquete
  suelto, sino el agregado).
- **Paquete (packet):** unidad mínima de datos que viaja por la red.
- **Payload:** el "contenido útil" de un paquete (los datos, sin las cabeceras).
- **Header (cabecera):** la "etiqueta del sobre" de un paquete (de dónde viene, a dónde
  va, etc.).
- **IAT (Inter-Arrival Time):** tiempo entre la llegada de un paquete y el siguiente. En
  ataques automáticos suele ser casi cero (mandan paquetes a ritmo constante).
- **Flag TCP:** banderitas de control de una conexión TCP (SYN, ACK, FIN, RST, PSH, URG…).
  Distintos ataques dejan firmas distintas de flags.
- **Ventana TCP (window size):** cuántos datos puede recibir un lado sin confirmar;
  parte de la negociación de la conexión.
- **IDS (Intrusion Detection System):** sistema que detecta ataques en una red. Es lo que
  nuestro modelo simula.
- **Estratificación:** repartir train/test manteniendo la misma proporción de clases en
  ambos.
- **CV (validación cruzada):** entrenar/evaluar varias veces rotando los datos, para
  comparar modelos de forma confiable.
- **Baseline (línea base):** modelo simple de referencia. Si algo complejo no le gana, no
  vale la pena.
- **Outlier:** valor extremo, muy lejos del resto.
- **One-hot encoding:** convertir una categoría de texto en columnas 0/1.
- **`class_weight`:** ajuste que le dice al modelo que preste más atención a la clase rara.
- **SMOTE:** técnica que genera ejemplos sintéticos de la clase minoritaria.

---

> 📌 Documento vivo: a medida que avancemos (síntesis, baseline, modelos), iremos
> ampliando esta guía. Si algo no se entiende, es un bug del documento — avísalo y lo
> arreglamos.
