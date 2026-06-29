# Registro de decisiones — Preparación de datos (Fase 2 → 3)

> **Qué es esto:** el registro **trazable** de **todas** las decisiones que tomamos para
> preparar los datos de RT-IoT2022 antes de modelar. Para cada decisión está **qué**
> decidimos, **por qué** y **en qué evidencia nos basamos**. Si alguien pregunta "¿por
> qué botaron tal columna?" o "¿por qué este mapeo?", la respuesta está aquí.
>
> Documento complementario: `docs/conceptos_proyecto.md` (explica los *conceptos*; este
> explica las *decisiones*). Proyecto: detección de anomalías RT-IoT2022 (CapiData).

**Convención de reproducibilidad:** `random_state = 42` en todo (split, CV, modelos).

---

## A. Enfoque y alcance

### A1. Enfoque supervisado, binario primero y multiclase después
- **Decisión:** clasificación **supervisada**; empezar con binario (normal/ataque) y
  luego extender a multiclase (tipo de ataque).
- **Por qué:** el dataset trae etiquetas (`Attack_type`) → tiene sentido supervisado.
  Binario primero es más simple, da un baseline interpretable y detecta problemas de
  datos (leakage) antes de complicar con 12 clases.
- **Base:** decisión de equipo + buenas prácticas de modelado incremental.

---

## B. El dataset

### B1. Usar la versión **oficial de UCI** (123,117 filas, sin `Amazon-Alexa`)
- **Decisión:** trabajar con el archivo `data/rt-iot2022.zip` → `RT_IOT2022`
  (123,117 filas × 84 columnas), que **no** incluye la clase `Amazon-Alexa`.
- **Por qué:** la página de UCI **describe** una clase `Amazon-Alexa` con 86,842 filas,
  pero el **archivo descargable no la trae**. Verificamos con `value_counts()`: las 12
  clases reales suman exactamente 123,117. La cuenta `123,117 + 86,842 = 209,959` muestra
  que la versión publicada = experimento original **menos** Alexa (que se publicó como
  dataset aparte). Usamos la versión oficial porque es la que **citan todos los papers**
  → resultados comparables y reproducibles.
- **Base / evidencia:**
  - `df["Attack_type"].value_counts()` → 12 clases, total 123,117.
  - Página UCI: declara "123,117 instances" pero lista clases que suman >180,000 (incluso
    solo `DOS_SYN_Hping` 94,659 + `Amazon-Alexa` 86,842 ya exceden el total) → su lista de
    conteos es metadata inconsistente.
  - Publicación separada "Amazon Alexa traffic traces" (mismos autores).
- **Lección de método:** cuando documentación y archivo se contradicen, **gana el
  archivo** (`value_counts`).
- **Consecuencia:** sin Alexa, el dataset queda **muy desbalanceado** (ver D1); con Alexa
  habría sido casi 50/50.

---

## C. Etiqueta (target)

### C1. Construir la etiqueta binaria normal/ataque
- **Decisión:** crear `y_bin` con 1 = ataque, 0 = normal, agrupando como **normal** solo
  estas 3 clases: `MQTT_Publish`, `Thing_Speak`, `Wipro_bulb`.
- **Por qué:** el dataset **no trae** una columna "Normal"; el tráfico legítimo viene
  etiquetado con el **nombre del dispositivo IoT** que lo generó. Esas 3 son dispositivos
  legítimos; las otras 9 son técnicas/herramientas de ataque (NMAP, hping, Metasploit…).
- **Base:** `value_counts()` real (las 12 clases) + significado de cada clase
  (`docs/ataques.md`, `docs/variables_dataset.md`).
- **Implementación:** `y_bin = (~df["Attack_type"].isin(NORMAL_CLASSES)).astype(int)`.

### C2. Conservar **además** la etiqueta multiclase
- **Decisión:** guardar `y_multi = Attack_type` (las 12 clases) sin sobrescribir.
- **Por qué:** la extensión posterior necesita saber **qué tipo** de ataque, no solo
  ataque sí/no.

| Mapeo binario | Clases | Filas |
|---|---|---|
| Normal (0) | MQTT_Publish, Thing_Speak, Wipro_bulb | 12,507 |
| Ataque (1) | DOS_SYN_Hping, ARP_poisioning*, NMAP_* (5), DDOS_Slowloris, Metasploit_Brute_Force_SSH | 110,610 |

> *`ARP_poisioning` está **mal escrito** en el dataset original — se usa el string tal cual.

---

## D. Desbalance de clases

### D1. Cuantificar y reportar el desbalance
- **Hecho:** 89.8% ataque / 10.2% normal (≈ 8.85 : 1).
- **Por qué importa:** un modelo que prediga "todo ataque" acierta 89.8% sin aprender
  nada → la **accuracy engaña**.

### D2. Empezar con `class_weight="balanced"` (no SMOTE)
- **Decisión:** manejar el desbalance con `class_weight="balanced"` en los modelos.
  SMOTE / undersampling quedan como **experimento posterior**, no de arranque.
- **Por qué:** `class_weight` no toca los datos (no inventa ni bota filas), es simple y
  seguro. Lo más importante con desbalance es elegir bien las **métricas** (ver H), no
  necesariamente rebalancear.
- **Base:** buenas prácticas; evitar artefactos de datos sintéticos en el baseline.

---

## E. Limpieza de columnas (no por correlación)

### E1. Botar la columna 100% constante `bwd_URG_flag_count`
- **Decisión:** eliminar siempre.
- **Por qué:** es **100% ceros** (varianza cero) → no aporta nada y rompe Pearson (NaN).
- **Base:** `nunique() == 1`; confirmado en `eda_daniel_flags_tcp.ipynb`.

### E2. Eliminar filas duplicadas
- **Decisión:** `drop_duplicates()`.
- **Por qué:** ~1,880 filas exactamente duplicadas (~1.5%) pueden sesgar e inflar
  métricas (la misma fila en train y test).
- **Base:** `eda_edwin_calidad_datos.ipynb`.

### E3. No imputar nulos
- **Decisión:** no hay imputación.
- **Por qué:** el dataset **no tiene valores faltantes**.
- **Base:** verificado en el EDA de calidad.

---

## F. Correlación y poda de redundancia

Este es el bloque más extenso porque fue la decisión central del notebook
`03_matriz_correlacion.ipynb`.

### F1. Analizar **redundancia entre features**, no relación con el target
- **Decisión:** la matriz mide correlación **entre variables** (para detectar
  duplicados), no con `Attack_type`.
- **Por qué:** el objetivo era reducir redundancia. La correlación con el target tiene
  riesgo de leakage y se difiere al baseline / importancia de RandomForest.

### F2. Usar **Spearman** como criterio principal (y comparar con Pearson)
- **Decisión:** calcular Spearman **y** Pearson; decidir con Spearman.
- **Por qué:** el tráfico de red está **muy sesgado** y lleno de **outliers**. Spearman
  usa el orden (robusto a outliers) y capta relaciones **monótonas no lineales** que
  Pearson no ve. Comparar ambos revela dónde la relación es no-lineal (|Δ| grande).
- **Base:** 277 pares |ρ|>0.9; varios con `abs_diff` alto (no-lineales).

### F3. Agrupar con clustering jerárquico (complete linkage, corte |ρ|=0.9)
- **Decisión:** agrupar variables redundantes con `linkage(method="complete")` sobre
  `1 − |Spearman|`, cortando en distancia 0.10.
- **Por qué:** decidir par-a-par (277 pares) es inmanejable. **Complete linkage**
  garantiza que **todos** los pares dentro de un grupo cumplen |ρ|>0.9 (sin
  encadenamientos espurios). Resultado: **13 grupos** redundantes (63 columnas) + **17
  solitarias**.

### F4. Regla del representante: **no-leaky → más denso → más interpretable**
- **Decisión:** de cada grupo se conserva 1 representante eligiendo, en orden: (1) que NO
  sea leaky, (2) el más denso (menos ceros), (3) el de nombre más interpretable.
- **Por qué:** conservar duplicados **perjudica** el análisis:
  - **Multicolinealidad:** en modelos lineales, coeficientes inestables e
    ininterpretables.
  - **Dilución de importancia:** en RandomForest, dos variables correlacionadas se
    reparten la importancia y **ambas parecen poco importantes**.
  Preferir la **densa** (menos ceros) conserva más señal usable; preferir la
  **interpretable** facilita explicar el modelo.
- **Nota:** la versión inicial del algoritmo elegía "la menos correlacionada con el resto"
  (unicidad estadística), pero producía elecciones poco interpretables y con muchos ceros
  → se cambió a esta regla.

### F5. Conservar **2 representantes** en los grupos "sueltos" (G1, G3, G5)
- **Decisión:** en grupos con `min|ρ| < 0.95`, conservar un 2º representante que capture
  una **faceta distinta** (el miembro menos correlacionado con el 1º).
- **Por qué:** en grupos "clones" (|ρ|≥0.99) el 2º sería duplicado puro. En grupos
  sueltos, el 2º aporta un **ángulo real** distinto, no un duplicado. La poda **no borra
  análisis** (las 84 columnas siguen en el notebook) y **no es permanente** (se puede
  revisar con RF).
- **Base (firmeza por grupo, `min|ρ|`):**

| Grupo | min\|ρ\| | ¿2º rep? |
|---|---|---|
| G7, G10, G11 | 1.000 | No (clones) |
| G12 | 0.997 | No |
| G13 | 0.993 | No |
| G2, G4, G6, G9 | 0.960 | No |
| **G3** | 0.945 | **Sí** |
| **G1** | 0.934 | **Sí** |
| **G8** | 0.911 | **No** (ver F6) |
| **G5** | 0.903 | **Sí** |

- **2º representante elegido:** G1 → `bwd_pkts_payload.avg` (payload, ≠ volumen); G3 →
  `fwd_iat.std` (variabilidad de tiempos, ≠ ventana); G5 → `fwd_init_window_size`
  (ventana TCP, ≠ cabecera).

### F6. G8 se queda con **1** representante
- **Decisión:** no conservar 2º en G8.
- **Por qué:** aunque es "suelto" (0.911), su único candidato no-leaky es
  `fwd_pkts_payload.min`, que es el **mismo facet** (payload) que el 1º
  (`fwd_pkts_payload.avg`) → sería el duplicado sin valor que queríamos evitar. El otro
  miembro del grupo, `id.resp_p`, es leaky (va por el toggle).

### F7. Overrides de interpretabilidad del equipo (G6 y G3)
- **G6 → conservar AMBAS** (`bwd_header_size_max` + `flow_ACK_flag_count`).
  - **Por qué:** el algoritmo eligió `bwd_header_size_max` por densidad, pero el conteo
    de flags **ACK** es más interpretable y relevante para un IDS (distingue conexiones
    establecidas vs escaneos). El equipo prefirió conservar las dos.
- **G3 → representante = `bwd_init_window_size`** (en vez de `fwd_iat.std`).
  - **Por qué:** la ventana TCP inicial tiene un significado de red más claro; G3 es un
    grupo heterogéneo ("Frankenstein") que se agrupó solo por compartir ~91% de ceros.

### F8. Lista final de poda por redundancia (`REDUNDANT_COLS_DROP`)
- **Resultado:** **46 columnas** a botar en `clean()`. Lista completa y trazable en
  `notebooks/03_matriz_correlacion.ipynb` (celda con `assert` de existencia) y en el plan.
- **Representantes conservados (16):** `fwd_pkts_tot`, `bwd_pkts_payload.avg`,
  `flow_duration`, `bwd_init_window_size`, `fwd_iat.std`, `flow_pkts_per_sec`,
  `fwd_header_size_max`, `fwd_init_window_size`, `bwd_header_size_max`,
  `flow_ACK_flag_count`, `idle.tot`, `fwd_pkts_payload.avg`, `fwd_subflow_bytes`,
  `fwd_bulk_bytes`, `bwd_pkts_tot`, `flow_CWR_flag_count`.
- **Caveat documentado:** muchas de estas correlaciones son por **ceros compartidos**
  (variables 90%+ ceros en las mismas filas). Spearman las ve "idénticas" por el patrón
  de ceros, no porque su parte no-cero sea igual → por eso aplicamos la regla del 1% (G).

---

## G. Familias casi-constantes (la "regla del 1%")

### G1. No botar por reflejo las columnas ~99% ceros
- **Decisión:** antes de botar una familia casi-constante, **investigar** si su 1% no-cero
  se concentra en una clase (señal rara valiosa).
- **Por qué:** 99% ceros no = inútil. Un valor no-cero que solo aparece en una clase puede
  ser un **detector** de esa clase.
- **Base / evidencia (dónde caen los no-cero):**

| Familia | % no-cero | Concentración | Decisión | Razón |
|---|---|---|---|---|
| `idle.*` | 4.63% | **72% normal** (MQTT) | **Conservar** (`idle.tot`) | señal de tráfico normal |
| `fwd_bulk.*` | 0.20% | **81% ataque** (ARP) | **Conservar 1** (`fwd_bulk_bytes`) | marcador raro de ARP |
| `flow_CWR/ECE` | 0.06% | **100% normal** | **Conservar 1** (`flow_CWR`) | marcador de normal |
| `bwd_bulk.*` | 0.90% | mezclado (46/54) | **Botar familia** | sin señal clara |

> Decisión del equipo: variante **"guiado por datos"** — conservar los marcadores con
> señal y botar solo la familia mezclada.

---

## H. Features con riesgo de leakage

### H1. Tratar puertos / `service` / `proto` con un toggle, no podarlas
- **Decisión:** `id.orig_p`, `id.resp_p`, `service`, `proto` se manejan con
  `build_preprocessor(include_leaky=...)`; el baseline corre **con y sin** ellas y se
  compara. **No** entran a la poda por redundancia.
- **Por qué:** pueden codificar el ataque casi 1:1 (p. ej. servicio SSH → fuerza bruta) →
  inflan métricas y no reflejan un IDS real. Si las botáramos en `clean()`, desaparecerían
  de **las dos** variantes y no podríamos medir su impacto.
- **Base / evidencia de leakage:** `id.resp_p` (puerto destino) correlaciona **>0.9** con
  `fwd_pkts_payload.avg` → el puerto "predice" el payload, confirmando que es leaky.

### H2. Señal de alarma a vigilar
- **Regla:** si el baseline da **F1 ≈ 1.0**, sobre todo en la variante *con leaky*, es
  indicio de que esas variables están "haciendo trampa" → documentarlo.

---

## I. Protocolo de evaluación (definido ANTES de modelar)

### I1. Partición estratificada + validación cruzada
- **Decisión:** train/test 80/20 **estratificado**; `StratifiedKFold` sobre train para
  comparar modelos; test reservado para el reporte final.
- **Por qué:** estratificar mantiene la proporción 89.8/10.2 en ambos lados; CV compara de
  forma confiable sin tocar el test.

### I2. Métricas adecuadas al desbalance
- **Decisión:** matriz de confusión + **precision/recall/F1 de la clase** + **PR-AUC**.
  Énfasis en **recall** de ataque.
- **Por qué:** la accuracy engaña con desbalance. En un IDS el error caro es el **falso
  negativo** (ataque no detectado) → priorizar recall sin destruir precision.
- **Piso de referencia:** `DummyClassifier(strategy="most_frequent")` para saber qué
  accuracy es "trivial".

### I3. Orden anti-leakage en el preprocesamiento
- **Decisión:** (1) split primero, (2) `fit` solo con train y `transform` a ambos, (3)
  cualquier resampling solo sobre train. Implementado con `Pipeline` + `ColumnTransformer`.
- **Por qué:** evita que el test "se filtre" en el cálculo de medias/escalas. El
  `Pipeline` lo garantiza por construcción.

---

## J. Escalado (decisión diferida al modelo)
- **Decisión:** numéricas → `StandardScaler` por defecto; revisar `RobustScaler`/`log1p`
  para las muy sesgadas. Categóricas → one-hot. Árboles no requieren escalado.
- **Por qué:** la elección **depende del modelo**; se confirma al comparar baselines.

---

## Resumen cuantitativo

| Concepto | Valor |
|---|---|
| Filas (versión oficial) | 123,117 |
| Columnas totales | 84 |
| Clases `Attack_type` | 12 (3 normal, 9 ataque) |
| Desbalance | 89.8% ataque / 10.2% normal |
| Constante botada | 1 (`bwd_URG_flag_count`) |
| Botadas por redundancia | 46 |
| Familia casi-constante botada | `bwd_bulk.*` (3 cols) |
| Representantes conservados | 16 |
| Solitarias conservadas | 16 (17 − `id.orig_p` leaky) |
| Features numéricas que sobreviven | **32** |
| Leaky (toggle, no podadas) | 4 (`id.orig_p`, `id.resp_p`, `service`, `proto`) |

---

## Decisiones aún abiertas (para cerrar al modelar)
1. ¿Se usan finalmente las features leaky? → decidir con la comparación con-vs-sin del
   baseline.
2. Scaler definitivo por variable (Standard vs Robust/log1p) → confirmar con el modelo.
3. Pares correlados >0.9 que se conservaron: re-evaluar con **importancia de RandomForest**
   tras el baseline antes de podar más.

---

> Documento vivo. Cada decisión nueva (al modelar) se agrega aquí con su porqué y su
> evidencia, para mantener la trazabilidad completa del proyecto.
