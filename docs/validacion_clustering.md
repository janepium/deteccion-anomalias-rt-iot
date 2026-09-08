# Validación del clustering jerárquico y del punto de corte

## 1. Objetivo

El objetivo de esta validación es evaluar la metodología utilizada en `03_matriz_correlacion.ipynb` para agrupar variables numéricas redundantes mediante clustering jerárquico.

La agrupación se realizó con base en la correlación de Spearman entre variables, utilizando una transformación de la correlación a una medida de distancia. Posteriormente, se evaluó la calidad de los grupos obtenidos mediante índices internos de clustering y se compararon diferentes puntos de corte.

La validación busca determinar si el punto de corte utilizado por el equipo, correspondiente a:

\[
|\rho| = 0.9
\]

produce grupos suficientemente coherentes y útiles para la reducción de redundancia.

---

## 2. Metodología utilizada

### 2.1 Variables utilizadas

La matriz de correlación se construyó utilizando únicamente variables numéricas.

Antes de calcular las correlaciones se eliminó la variable constante:

- `bwd_URG_flag_count`

Por tanto, se utilizaron:

- 80 variables numéricas
- 123.117 registros

La variable objetivo `Attack_type` no se utilizó para construir los grupos.

Las variables potencialmente *leaky* (`id.orig_p`, `id.resp_p`, `service` y `proto`) se mantienen fuera de la poda definitiva y son controladas posteriormente mediante el parámetro `include_leaky`.

---

## 3. Spearman y Pearson: resultados observados

Al utilizar un umbral de:

\[
|\rho| > 0.9
\]

se encontraron:

- **277 pares** de variables considerando la unión de Spearman y Pearson.
- **202 pares** fueron detectados únicamente por Spearman.
- **54 pares** fueron detectados únicamente por Pearson.
- **239 pares** presentaron una diferencia absoluta entre ambos coeficientes superior a 0.2.

La diferencia observada entre ambos métodos proporciona evidencia de que una parte importante de las relaciones presentes en las variables numéricas no puede describirse únicamente mediante relaciones lineales.

Por esta razón, el equipo adoptó Spearman como criterio principal para la agrupación.

---

## 4. Distancia utilizada

Para realizar el clustering jerárquico se utilizó la transformación:

\[
d_{ij}=1-|\rho_{ij}|
\]

donde:

- \(\rho_{ij}\) corresponde a la correlación de Spearman entre las variables \(i\) y \(j\).
- \(d_{ij}\) representa la distancia entre ambas variables.

Esta transformación permite interpretar una correlación absoluta alta como una distancia pequeña:

| \(|\rho|\) | Distancia \(1-|\rho|\) |
|---:|---:|
| 1.00 | 0.00 |
| 0.99 | 0.01 |
| 0.95 | 0.05 |
| 0.90 | 0.10 |
| 0.80 | 0.20 |

Por lo tanto, el punto de corte elegido:

\[
|\rho|=0.9
\]

es equivalente a:

\[
d=0.1
\]

La elección de este valor se realizó como criterio de reducción de redundancia. No corresponde a un valor obtenido automáticamente a partir de los índices de validación, sino a una decisión metodológica del equipo que posteriormente se sometió a validación.

### 4.1 Resultado clustering

Con este corte se obtuvieron **30 grupos**, considerando tanto los grupos redundantes como las variables que permanecieron como grupos individuales.

Dentro del procedimiento de reducción de redundancia se identificaron familias de variables altamente correlacionadas y posteriormente se aplicó la regla de selección de representantes acordada por el equipo:

1- No leaky.
2- Mayor densidad: menor proporción de ceros.
3- Mayor interpretabilidad.

El resultado final fue una reducción de la redundancia mediante el descarte de 46 columnas, manteniendo las variables representantes y las variables que no presentaban redundancia superior al umbral.

---

## 5. Validación de índices internos

Para evaluar la calidad de las agrupaciones se utilizaron cuatro índices internos:

- **Silhouette Score:** mide la calidad y la cohesión de los grupos formados. Rango de entre -1 a +1, siendo mejor un resultado cercano a +1.
- **Davies-Bouldin Index:** mide la similitud entre cada grupo y el grupo más parecido a él. Rango de 0 a infinito. Valores menores indican grupos más compactos y mejor separados.
- **Calinski-Harabasz Index:** compara la dispersión entre grupos con la dispersión dentro de los grupos. Rango de 0 a infinito. Un valor alto indica grupos con una mayor separación respecto a su dispersión interna.
- **Dunn Index:**  busca identificar grupos que estén bien compactos por dentro y bien separados entre sí. Rango de 0 a infinito. Un valor alto significa grupos separados y compactos.

Estos indices primero se probaron de forma individual con \[|\rho|=0.9\] y luego para la validación del corte se contastó con los indices para los umbrales: 0.80, 0.85, 0.95 y 0.99.

Se obtuvieron los siguientes resultados:

| Índice                  |      Valor     |    Criterio    |
| ----------------------- | -------------- | -------------- |
| Silhouette Score        |   **0.663332** | Mayor es mejor |
| Davies-Bouldin Index    |   **0.217108** | Menor es mejor |
| Calinski-Harabasz Index | **225.687463** | Mayor es mejor |
| Dunn Index              |   **0.952666** | Mayor es mejor |

**Observaciones:**

- El Silhouette Score de 0.6633 indica una estructura de agrupamiento razonablemente compacta y separada. Las observaciones presentan, en promedio, mayor similitud con su propio grupo que con los grupos vecinos.

- El Davies-Bouldin de 0.2171 muestra una baja similitud relativa entre los grupos en comparación con configuraciones con valores superiores.

- El Calinski-Harabasz de 225.6875 proporciona evidencia de una separación considerable entre grupos respecto a su dispersión interna. Este valor resulta útil al compararlo con los demás puntos de corte evaluados.

- El Dunn Index de 0.9527 presenta un valor alto, lo que indica una relación favorable entre la separación de los grupos y la dispersión interna.

- En conjunto, los cuatro índices muestran que el agrupamiento obtenido con el corte de 0.9 presenta una estructura interna adecuada para el objetivo de identificar familias de variables redundantes.

Y cuando se comparó con los otros puntos de corte:

| Corte \(|\rho|\) | Distancia | Num grupos | Silhouette | Davies-Bouldin | Calinski-Harabasz | Dunn |
|---:|---:|---:|---:|---:|---:|---:|
| 0.80 | 0.20 | 26 | 0.712605 | 0.257594 | 156.421913 | 0.560704 |
| 0.85 | 0.15 | 28 | 0.688419 | 0.223124 | 204.134399 | 0.817303 |
| 0.90 | 0.10 | 30 | 0.663332 | 0.217108 | 225.687463 | 0.952666 |
| 0.95 | 0.05 | 35 | 0.604039 | 0.260190 | 334.829619 | 0.634254 |
| 0.99 | 0.01 | 52 | 0.425996 | 0.104180 | 1739.141390 | 0.744881 |

Observamos que el Silhouette disminuye progresivamente al aumentar el punto de corte, esto indica que los grupos son progresivamente menos compactos/separados cuando se exige una correlación más alta para formar un grupo.

- El mejor resultado de Silhouette se obtiene con \(|\rho|=0.80\).

Davies-Bouldin vemos que favorece configuraciones con un mayor número de grupos, especialmente el corte de 0.99. 

- El mejor resultado de Davies-Bouldin se obtiene con \(|\rho|=0.99\).

Calinski-Harabasz aumenta considerablemente al aumentar el punto de corte.

- El mejor resultado de Calinski-Harabasz se obtiene con \(|\rho|=0.99\).

El Dunn Index alcanza su máximo en el corte de \[|\rho|=0.9\].

### 5.1 Justificación del corte \[|\rho|=0.9\]

Los resultados muestran que no existe un punto de corte que maximice simultáneamente los cuatro índices.

El corte de \(|\rho|=0.80\) obtiene el mejor Silhouette, mientras que \(|\rho|=0.99\) obtiene los mejores valores de Davies-Bouldin y Calinski-Harabasz.

Sin embargo, el corte de \(|\rho|=0.90\) obtiene el mejor Dunn Index y proporciona un equilibrio entre el número de grupos y el nivel de redundancia exigido.

Además, el objetivo del procedimiento no era encontrar el clustering matemáticamente óptimo de las variables, sino identificar familias de variables suficientemente redundantes para realizar una poda controlada.

Por esta razón, el equipo mantiene \(|\rho|=0.90\) como punto de corte.

La validación permite concluir que este valor genera una estructura de agrupamiento internamente consistente y al mismo tiempo resulta adecuado para el objetivo específico de reducción de redundancia.