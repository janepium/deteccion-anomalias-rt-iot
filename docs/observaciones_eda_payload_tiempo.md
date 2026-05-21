# Registro de observaciones: EDA Payload y Tiempo
**Notebook:** `notebooks/eda_juan_payload_tiempo.ipynb`
**Analista:** Juan Florez
**Fecha de ejecución:** 21-05-26

---

## Sección 1 — Estadísticas descriptivas

### 1.1 Resumen general (`describe`)

**¿En qué variables la media es mucho mayor que la mediana?**
> (Indica valores extremos jalando la media hacia arriba — señal de comportamiento anómalo)

| Variable | Media | Mediana (50%) | ¿Diferencia notable? | Interpretación |
|---|---|---|---|---|
| `fwd_iat.min` | 8,843.27 | 0.00 | ✅ Sí — extrema | La mayoría de flujos tiene tiempo mínimo entre paquetes de 0; unos pocos con pausas largas elevan la media. |
| `fwd_iat.max` | 1,721,565.57 | 0.00 | ✅ Sí — extrema | Distribución casi binaria: mayoría en 0, outliers con tiempos máximos de cientos de miles de µs. |
| `fwd_iat.tot` | 3,780,208.46 | 0.00 | ✅ Sí — extrema | Tiempo total entre paquetes forward; la mediana en 0 indica flujos de un solo paquete predominantes. |
| `fwd_iat.avg` | 237,357.50 | 0.00 | ✅ Sí — extrema | Promedio fuertemente jalado por flujos con grandes intervalos de inactividad. |
| `fwd_iat.std` | 577,557.39 | 0.00 | ✅ Sí — extrema | Alta desviación estándar en flujos excepcionales; mayoría tiene variabilidad nula (un solo paquete). |
| `bwd_iat.min` | 3,764.85 | 0.00 | ✅ Sí — notable | Patrón similar al forward: dominancia de flujos sin respuesta backward o de un solo paquete. |
| `bwd_iat.max` | 407,726.69 | 0.00 | ✅ Sí — extrema | Outliers con tiempos máximos elevados en la dirección backward; mayoría no tiene tráfico de retorno. |
| `bwd_iat.tot` | 1,779,892.55 | 0.00 | ✅ Sí — extrema | Flujos backward inexistentes en la mayoría de muestras; total acumulado solo en flujos bidireccionales atípicos. |
| `bwd_iat.avg` | 87,652.13 | 0.00 | ✅ Sí — extrema | Mismo patrón: promedio backward inflado por pocos flujos con alta latencia de respuesta. |
| `bwd_iat.std` | 147,480.26 | 0.00 | ✅ Sí — extrema | Variabilidad backward prácticamente nula en la mayoría; alta solo en flujos con múltiples paquetes de retorno. |
| `flow_iat.min` | 4,283.08 | 3.81 | ✅ Sí — notable | La mediana es cercana a cero pero no nula; la media aún duplica con creces la mediana por outliers. |
| `flow_iat.max` | 1,725,999.06 | 4.05 | ✅ Sí — extrema | Brecha de ~425,000x entre media y mediana; valores máximos de hasta 3×10⁸ distorsionan completamente la distribución. |
| `flow_iat.tot` | 3,810,575.01 | 4.05 | ✅ Sí — extrema | Igual que el máximo; el total del flujo refleja los mismos outliers de alta duración. |
| `flow_iat.avg` | 139,654.49 | 4.05 | ✅ Sí — extrema | Promedio de flujo ~34,000x mayor que la mediana; sesgo extremo hacia la derecha. |
| `flow_iat.std` | 450,136.21 | 0.00 | ✅ Sí — extrema | La desviación estándar del flujo sigue el mismo patrón: nula en la mayoría, enorme en outliers. |
| `fwd_pkts_payload.min` | 96.26 | 120.00 | ⚠️ Inversa | La media es menor que la mediana; algunos paquetes con payload mínimo (0) jalan la media hacia abajo. |
| `fwd_pkts_payload.max` | 120.75 | 120.00 | ➖ Sin diferencia notable | Media y mediana casi idénticas; distribución concentrada alrededor de 120 bytes. |
| `fwd_pkts_payload.tot` | 221.52 | 120.00 | ⚠️ Moderada | La media duplica la mediana; flujos con múltiples paquetes grandes elevan el total. |
| `fwd_pkts_payload.avg` | 100.52 | 120.00 | ⚠️ Inversa | La media cae bajo la mediana por flujos con payload reducido. |
| `fwd_pkts_payload.std` | 8.11 | 0.00 | ✅ Sí — notable | La mayoría de flujos tiene payload constante (std = 0); pocos con variación elevan la media. |
| `bwd_pkts_payload.min` | 3.82 | 0.00 | ✅ Sí — notable | Mayoría sin tráfico backward; pocos flujos con payload mínimo no nulo. |
| `bwd_pkts_payload.max` | 52.41 | 0.00 | ✅ Sí — extrema | Tráfico backward ausente en la mayoría de flujos; media elevada por respuestas con payloads grandes. |
| `bwd_pkts_payload.tot` | 513.00 | 0.00 | ✅ Sí — extrema | Patrón idéntico: flujos unidireccionales predominan; bidireccionales con gran payload acumulado jalan la media. |
| `bwd_pkts_payload.avg` | 18.79 | 0.00 | ✅ Sí — extrema | Promedio backward elevado únicamente por los flujos que sí tienen respuesta. |
| `bwd_pkts_payload.std` | 20.55 | 0.00 | ✅ Sí — notable | Variabilidad en backward solo donde existe tráfico de retorno. |
| `flow_pkts_payload.min` | 13.55 | 0.00 | ✅ Sí — notable | Flujos con payload mínimo de 0 predominan; media jalada por flujos con paquetes pequeños no nulos. |
| `flow_pkts_payload.max` | 148.51 | 120.00 | ⚠️ Moderada | Diferencia moderada; algunos flujos con payloads máximos superiores a 120 bytes elevan la media. |
| `flow_pkts_payload.tot` | 734.52 | 120.00 | ⚠️ Moderada | Media ~6x la mediana; flujos con muchos paquetes grandes acumulan totales elevados. |
| `flow_pkts_payload.avg` | 65.01 | 60.00 | ➖ Sin diferencia notable | Media y mediana muy cercanas; distribución relativamente simétrica alrededor de 60 bytes. |
| `flow_pkts_payload.std` | 76.04 | 84.85 | ⚠️ Inversa | Media ligeramente menor que la mediana; distribución algo sesgada a la izquierda en variabilidad de payload. |

**Observación general:**

> La mayoría de las variables de **inter-arrival time (IAT)** — tanto en dirección forward (`fwd_iat.*`), backward (`bwd_iat.*`) como de flujo (`flow_iat.*`) — presentan una media enormemente superior a la mediana, que en casi todos los casos es **0.00 o cercana a cero**. Esto evidencia una distribución **fuertemente sesgada a la derecha**: la gran mayoría de los flujos de red tienen tiempos de llegada entre paquetes nulos o instantáneos (flujos de un solo paquete o comunicaciones muy rápidas), mientras un subconjunto reducido de flujos con pausas extremadamente largas jala la media hacia valores de cientos de miles o millones de microsegundos. 

> Las variables de **payload backward** (`bwd_pkts_payload.*`) siguen el mismo patrón por una razón diferente: la mayoría de flujos capturados son **unidireccionales** (sin respuesta), por lo que su mediana es 0; solo los flujos con tráfico de retorno contribuyen a elevar la media.
 
> En contraste, las variables de **payload forward** (`fwd_pkts_payload.*`) muestran un comportamiento más simétrico, con mediana alrededor de 120 bytes, lo que sugiere que la mayoría de los paquetes de solicitud sí tienen contenido consistente.
 
*Implicaciones para modelado*
 
- Las variables IAT requieren **transformación logarítmica o winsorización** antes de usarse en modelos de machine learning, ya que su escala actual haría que dominen cualquier métrica de distancia.
- La asimetría extrema en variables backward es una **señal diagnóstica útil** en sí misma: puede usarse directamente como feature para detectar flujos anómalos o ataques de red (escaneos, SYN flood, conexiones semiduplex).
- Las variables con media ≈ mediana (como `flow_pkts_payload.avg`) son las más estables y pueden usarse directamente sin transformación.

---

### 1.2 Variables con muchos ceros

**Umbral de descarte: > 50% de ceros**

| Variable | % ceros | ¿Descartar? |
|---|---|---|
| `fwd_iat.min` | 85.69% | ✅ Sí |
| `fwd_iat.max` | 85.69% | ✅ Sí |
| `fwd_iat.tot` | 85.69% | ✅ Sí |
| `fwd_iat.avg` | 85.69% | ✅ Sí |
| `fwd_iat.std` | 91.00% | ✅ Sí |
| `bwd_iat.min` | 86.09% | ✅ Sí |
| `bwd_iat.max` | 86.08% | ✅ Sí |
| `bwd_iat.tot` | 86.08% | ✅ Sí |
| `bwd_iat.avg` | 86.08% | ✅ Sí |
| `bwd_iat.std` | 91.36% | ✅ Sí |
| `flow_iat.min` | 13.01% | ❌ No |
| `flow_iat.max` | 12.98% | ❌ No |
| `flow_iat.tot` | 12.98% | ❌ No |
| `flow_iat.avg` | 12.98% | ❌ No |
| `flow_iat.std` | 85.71% | ✅ Sí |
| `fwd_pkts_payload.min` | 14.05% | ❌ No |
| `fwd_pkts_payload.max` | 5.53% | ❌ No |
| `fwd_pkts_payload.tot` | 5.53% | ❌ No |
| `fwd_pkts_payload.avg` | 5.53% | ❌ No |
| `fwd_pkts_payload.std` | 91.17% | ✅ Sí |
| `bwd_pkts_payload.min` | 93.86% | ✅ Sí |
| `bwd_pkts_payload.max` | 85.77% | ✅ Sí |
| `bwd_pkts_payload.tot` | 85.77% | ✅ Sí |
| `bwd_pkts_payload.avg` | 85.77% | ✅ Sí |
| `bwd_pkts_payload.std` | 86.40% | ✅ Sí |
| `flow_pkts_payload.min` | 83.05% | ✅ Sí |
| `flow_pkts_payload.max` | 5.50% | ❌ No |
| `flow_pkts_payload.tot` | 5.50% | ❌ No |
| `flow_pkts_payload.avg` | 5.50% | ❌ No |
| `flow_pkts_payload.std` | 16.43% | ❌ No |

**¿Hay más ceros en variables IAT o payload?**
> Las variables IAT (fwd/bwd) y payload backward concentran los porcentajes más altos de ceros, todos por encima del 85–91%. Las variables `fwd_pkts_payload.*` y `flow_pkts_payload.max/tot/avg` son las más limpias, con apenas 5–16% de ceros. En síntesis: **IAT forward/backward y payload backward** son los grupos con mayor densidad de ceros.

**¿Los ceros tienen una explicación lógica?** (ej. el servidor no respondió → `bwd` en cero)
> Sí. Los ceros en variables `bwd_*` se explican porque gran parte de los flujos son **unidireccionales**: el servidor nunca respondió (escaneos de puertos, SYN sin ACK, ataques DoS), por lo que todos los IAT y payload backward son cero por definición. Los ceros en `fwd_iat.*` y `flow_iat.std` corresponden a flujos de **un solo paquete**: sin un segundo paquete no existe intervalo entre llegadas, y la desviación estándar es cero o indefinida. Ambos patrones son estructurales al protocolo de captura, no errores de datos.
---

### 1.3 Mediana Normal vs Ataque

Registrar las variables con ratio más extremo (mayor o menor diferencia entre grupos).

**Variables donde los ataques tienen valores MÁS ALTOS (ratio > 3):**

| Variable | Mediana Normal | Mediana Ataque | Ratio |
|---|---|---|---|
| `fwd_pkts_payload.min` | 0.00 | 120.00 | NaN (÷0) |
| `fwd_pkts_payload.avg` | 30.71 | 120.00 | 3.91 |
| `fwd_pkts_payload.max` | 36.00 | 120.00 | 3.33 |
| `flow_pkts_payload.std` | 16.00 | 84.85 | 5.30 |

**Variables donde los ataques tienen valores MÁS BAJOS (ratio < 0.3):**

| Variable | Mediana Normal | Mediana Ataque | Ratio |
|---|---|---|---|
| `fwd_iat.min` | 240.80 | 0.00 | 0.00 |
| `fwd_iat.max` | 294,732.09 | 0.00 | 0.00 |
| `fwd_iat.tot` | 881,945.85 | 0.00 | 0.00 |
| `fwd_iat.avg` | 124,491.69 | 0.00 | 0.00 |
| `fwd_iat.std` | 138,889.87 | 0.00 | 0.00 |
| `bwd_iat.min` | 198.13 | 0.00 | 0.00 |
| `bwd_iat.max` | 283,788.92 | 0.00 | 0.00 |
| `bwd_iat.tot` | 597,270.97 | 0.00 | 0.00 |
| `bwd_iat.avg` | 124,996.01 | 0.00 | 0.00 |
| `bwd_iat.std` | 144,852.56 | 0.00 | 0.00 |
| `flow_iat.min` | 183.11 | 3.81 | 0.02 |
| `flow_iat.max` | 268,882.99 | 3.81 | 0.00 |
| `flow_iat.tot` | 882,166.86 | 3.81 | 0.00 |
| `flow_iat.avg` | 68,176.29 | 3.81 | 0.00 |
| `flow_iat.std` | 111,037.43 | 0.00 | 0.00 |
| `bwd_pkts_payload.max` | 68.00 | 0.00 | 0.00 |
| `bwd_pkts_payload.tot` | 104.00 | 0.00 | 0.00 |
| `bwd_pkts_payload.avg` | 52.00 | 0.00 | 0.00 |
| `bwd_pkts_payload.std` | 22.63 | 0.00 | 0.00 |
| `fwd_pkts_payload.std` | 12.93 | 0.00 | 0.00 |
| `flow_pkts_payload.tot` | 176.00 | 120.00 | 0.68 |

**Observación general:**
> El patrón es muy claro y consistente: los flujos de **ataque tienen medianas de IAT iguales a 0** en todas las variables forward, backward y de flujo, mientras que el tráfico normal muestra valores de decenas de miles a cientos de miles de microsegundos. Esto indica que los ataques generan paquetes de forma **prácticamente instantánea y continua**, sin pausas entre ellos — comportamiento típico de floods, escaneos rápidos o tráfico automatizado. En sentido contrario, los ataques muestran **payload forward consistente de 120 bytes** (mediana fija), lo que sugiere paquetes de tamaño fijo y predecible, mientras que el tráfico normal tiene mayor variabilidad y payloads menores en la mediana. Las variables IAT son por tanto las más discriminativas para separar ataque de tráfico legítimo, y las variables de payload forward aportan señal complementaria útil.

---

## Sección 2 — Distribuciones (small multiples)

> Para cada variable ejecutada, registrar la observación en la tabla correspondiente.
> Describir la forma de la curva: estrecha/ancha, pegada al cero/desplazada, simétrica/sesgada.

### 2.1 y 2.2 — Variables IAT

| Variable | Forma curva Normal | Forma curva Ataque | ¿Se separan los grupos? | Nota adicional |
|---|---|---|---|---|
| `fwd_iat.min` | Bimodal: picos en log1p≈5.0 y ≈6.5; dispersa con cola en valores altos (rango 0–17.5) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; prácticamente sin dispersión | Sí | Separación muy clara: Normal tiene tiempos mínimos altos (flujos lentos); Ataque en 0 indica paquetes casi simultáneos |
| `fwd_iat.max` | Multimodal: picos en log1p≈5.5, ≈6.5, ≈12.5 y ≈17.5; muy dispersa (rango 0–20) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; cola mínima | Sí | Separación muy marcada: Normal tiene tiempos máximos muy altos; Ataque concentrado en 0, flujos sin espera entre paquetes |
| `fwd_iat.tot` | Multimodal: picos en log1p≈5.5, ≈14 y ≈18; muy dispersa (rango 0–22) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; rango efectivo 0–2 | Sí | Separación clara: Normal acumula tiempos totales muy altos; Ataque indica flujos con tiempo total de llegada casi nulo |
| `fwd_iat.avg` | Multimodal: picos en log1p≈5.5, ≈11.5 y ≈15.5; muy dispersa (rango 0–18) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; cola mínima hasta log1p≈10 | Sí | Separación muy marcada: Normal presenta tiempos promedio entre paquetes altos; Ataque casi siempre en 0 |
| `fwd_iat.std` | Bimodal: pico dominante en log1p≈0 y pico secundario en log1p≈11.5 y ≈16.5; dispersa (rango 0–18) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; prácticamente sin dispersión | Parcial | Solapamiento en log1p≈0; Normal tiene subpoblación con std muy alta (flujos irregulares) que sí discrimina |
| `bwd_iat.min` | Multimodal: picos en log1p≈4, ≈7.5 y ≈10; dispersa (rango 0–17.5) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; cola mínima | Sí | Separación clara: Normal tiene tiempos mínimos backward significativos; Ataque concentrado en 0 |
| `bwd_iat.max` | Multimodal: picos en log1p≈7.5, ≈10, ≈12.5 y ≈14; muy dispersa (rango 0–19) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; cola mínima hasta log1p≈10 | Sí | Separación muy marcada: Normal tiene tiempos máximos backward muy altos; Ataque prácticamente en 0 |
| `bwd_iat.tot` | Multimodal: picos en log1p≈8, ≈10.5 y ≈14; dispersa (rango 0–22) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; rango efectivo 0–2 | Sí | Separación marcada: Normal acumula tiempos backward totales altos; Ataque indica ausencia de intervalos backward |
| `bwd_iat.avg` | Multimodal: picos en log1p≈7.5, ≈11 y ≈12.5; dispersa (rango 0–17.5) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; cola mínima | Sí | Separación muy clara: Normal con promedios backward altos; Ataque en 0 indica flujos unidireccionales o sin espera |
| `bwd_iat.std` | Bimodal: pico dominante en log1p≈0 y pico secundario en log1p≈11.5 y ≈13.5; muy dispersa (rango 0–18) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; prácticamente sin dispersión | Parcial | Solapamiento en log1p≈0; subpoblación de Normal con std alta sí discrimina, pero la masa en 0 reduce utilidad global |
| `flow_iat.min` | Bimodal: picos en log1p≈4 y ≈5.5; concentrada en rango 3–8 | Multimodal concentrada: picos en log1p≈0, ≈1.5 y ≈2.5; rango 0–5 | Sí | Separación clara aunque ambos son multimodales; Normal tiene tiempos mínimos claramente más altos que Ataque |
| `flow_iat.max` | Bimodal: picos en log1p≈12 y ≈17.5; dispersa (rango 0–20) | Multimodal concentrada: picos en log1p≈0.5, ≈1.5 y ≈2.5; rango 0–6 | Sí | Separación muy marcada: Normal tiene tiempos máximos de flujo altísimos; Ataque completamente concentrado en valores bajos |
| `flow_iat.tot` | Trimodal: picos en log1p≈10, ≈14 y ≈18; muy dispersa (rango 0–22) | Bimodal: picos en log1p≈1.5 y ≈2.5; concentrada (rango 0–6) | Sí | Separación muy marcada: Normal tiene tiempos totales de flujo muy altos; Ataque en valores bajos indica flujos muy cortos |
| `flow_iat.avg` | Trimodal: picos en log1p≈9, ≈11 y ≈15; dispersa (rango 0–17.5) | Multimodal concentrada: picos en log1p≈1, ≈2 y ≈2.5; rango 0–5 | Sí | Separación clara: Normal con promedios de flujo altos; Ataque con promedios muy bajos, consistente con ataques de alta frecuencia |
| `flow_iat.std` | Multimodal: picos en log1p≈10, ≈11.5 y ≈16.5; dispersa con cola larga (rango 0–18) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; rango efectivo 0–2 | Sí | Separación muy marcada: Normal con alta variabilidad en tiempos de flujo; Ataque con std≈0 indica flujos perfectamente regulares |

**¿Qué tipos de ataque tienen distribución más diferente al tráfico normal en IAT?**
> El tráfico de ataque en general muestra una distribución radicalmente diferente al tráfico
> normal en casi todas las variables IAT. El patrón más distintivo es la concentración
> extrema en log1p≈0 en las variables `fwd_iat` y `bwd_iat`, lo que indica flujos con
> intervalos entre paquetes casi nulos — comportamiento consistente con ataques de alta
> frecuencia como DoS/DDoS o flooding. Esta diferencia es especialmente marcada en
> `fwd_iat.max`, `fwd_iat.tot`, `bwd_iat.max`, `bwd_iat.tot` y `flow_iat.max`,
> donde Normal presenta distribuciones dispersas con valores muy altos (log1p entre 10
> y 18) mientras Ataque se agrupa fuertemente en valores cercanos a 0. Las variables
> `flow_iat.avg` y `flow_iat.tot` también muestran separación muy marcada, con Normal
> en rangos altos (log1p≈9–18) y Ataque comprimido en log1p≈1–3.

**¿Algún tipo de ataque tiene curva similar al tráfico normal? (difícil de separar)**
> Las variables `fwd_iat.std` y `bwd_iat.std` son las más difíciles de usar como
> discriminadores, ya que ambas clases comparten una masa importante en log1p≈0:
> el tráfico Normal también presenta una subpoblación con desviación estándar nula
> (paquetes equiespaciados), lo que se solapa directamente con el comportamiento
> del Ataque. Fuera de estas dos variables, no se observa ningún tipo de ataque
> con curva globalmente similar al tráfico normal en las IAT; la separación es
> consistente y clara en las 13 variables restantes.

---

### 2.3 y 2.4 — Variables Payload

| Variable | Forma curva Normal | Forma curva Ataque | ¿Se separan los grupos? | Nota adicional |
|---|---|---|---|---|
| `fwd_pkts_payload.min` | Bimodal: pico dominante en log1p≈0 y segundo pico en log1p≈3.5; distribución dispersa (rango 0–7) | Unimodal muy concentrada: pico agudo en log1p≈4.8; casi sin dispersión | Sí | Normal se concentra en valores bajos y medios; Ataque se agrupa fuertemente en ~4.8. Alta utilidad discriminativa |
| `fwd_pkts_payload.max` | Trimodal: picos en log1p≈3.5, ≈5.1 y ≈6.2; distribución dispersa (rango 0–7.5) | Unimodal concentrada: pico dominante en log1p≈4.8 con cola derecha mínima | Parcial | Los rangos se solapan parcialmente alrededor de log1p≈4–5; Normal tiene mayor dispersión y valores más altos |
| `fwd_pkts_payload.tot` | Multimodal: tres picos claros en log1p≈4, ≈5.2 y ≈6.7; rango 0–14 | Unimodal muy concentrada: pico agudo en log1p≈4.9; rango efectivo 0–6 | Parcial | Ataque se concentra donde Normal tiene su primer pico; Normal tiene subpoblaciones con totales mucho mayores |
| `fwd_pkts_payload.avg` | Bimodal: picos en log1p≈2.2 y ≈3.5; moderadamente dispersa (rango 0–7) | Unimodal muy concentrada: pico agudo en log1p≈4.8; casi sin dispersión | Sí | Los picos de Normal se ubican claramente por debajo del pico de Ataque; buena separación global |
| `fwd_pkts_payload.std` | Multimodal: picos en log1p≈0, ≈2.7 y ≈4.1; muy dispersa (rango 0–6.5) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; casi sin varianza | No | Ambos grupos comparten la zona de log1p≈0; la curva de Ataque indica varianza nula (paquetes de tamaño fijo) |
| `bwd_pkts_payload.min` | Bimodal: pico dominante en log1p≈0 y segundo pico en log1p≈3.5; rango 0–6 | Unimodal extremadamente concentrada: pico agudo en log1p≈0; prácticamente sin dispersión | No | Ambos grupos se concentran en log1p≈0; Ataque con mucho mayor masa en ese punto; baja utilidad discriminativa |
| `bwd_pkts_payload.max` | Multimodal: picos en log1p≈3.2, ≈4.2, ≈6.3 y ≈7.1; muy dispersa (rango 0–7.5) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; cola mínima | Sí | Separación clara: Normal tiene valores típicos entre 3 y 7; Ataque casi todo en 0. Útil como discriminador |
| `bwd_pkts_payload.tot` | Multimodal: picos en log1p≈3.5, ≈4.8 y ≈5.5; dispersa (rango 0–16) | Unimodal extremadamente concentrada: pico agudo en log1p≈0.3; rango efectivo 0–2 | Sí | Separación marcada: Normal tiene totales mucho más altos; Ataque indica flujos backward casi vacíos |
| `bwd_pkts_payload.avg` | Multimodal: picos en log1p≈2, ≈4 y ≈4.6; rango 0–7 | Unimodal extremadamente concentrada: pico agudo en log1p≈0; prácticamente sin dispersión | Sí | Normal presenta promedios intermedios y altos; Ataque concentrado en 0, sugiriendo ausencia de payload en sentido backward |
| `bwd_pkts_payload.std` | Multimodal: picos en log1p≈0, ≈2.3, ≈3.2 y ≈5.5; muy dispersa (rango 0–6.5) | Unimodal extremadamente concentrada: pico agudo en log1p≈0; casi sin dispersión | Parcial | Ataque se solapa con el primer pico de Normal en 0; los picos secundarios de Normal sí discriminan |
| `flow_pkts_payload.min` | Bimodal: pico dominante en log1p≈0 y segundo pico en log1p≈3.5; rango 0–7 | Bimodal asimétrica: pico dominante en log1p≈0 y pico secundario en log1p≈4.8; cola derecha visible | No | Ambos grupos comparten la concentración en log1p≈0; aunque Ataque tiene subpico en 4.8, la superposición en 0 domina |
| `flow_pkts_payload.max` | Trimodal: picos en log1p≈3.5, ≈4.2 y ≈6.5; dispersa (rango 0–7.5) | Unimodal muy concentrada: pico agudo en log1p≈4.5; rango efectivo 0–6 | Parcial | Solapamiento en la zona log1p≈4–5; Normal tiene mayor dispersión y picos en valores más altos |
| `flow_pkts_payload.tot` | Trimodal: picos en log1p≈5, ≈5.3 y ≈6.8; rango 3–10 | Unimodal muy concentrada: pico agudo en log1p≈4.8; rango efectivo 3–7 | Parcial | Solapamiento en zona log1p≈4.8–5; Normal tiene subpoblaciones con totales más altos que Ataque |
| `flow_pkts_payload.avg` | Bimodal: picos en log1p≈2.1 y ≈4.0; moderadamente dispersa (rango 0–7) | Bimodal: pico dominante en log1p≈4.0 y pico secundario en log1p≈4.7; concentrada | Parcial | Pico secundario de Ataque se solapa con segundo pico de Normal; el pico en log1p≈2 de Normal es distintivo |
| `flow_pkts_payload.std` | Bimodal: picos en log1p≈2.6 y ≈5.0; dispersa con cola larga (rango 0–6.5) | Bimodal: pico dominante en log1p≈0 y pico agudo en log1p≈4.5; distribución discontinua | Sí | Separación clara: Ataque tiene masa importante en 0 (sin varianza) y en 4.5; Normal en zonas intermedias |

**¿Los ataques tienden a tener payload más pequeño, más grande, o depende del tipo?**
> Depende de la dirección del flujo. En las variables `fwd_*` (dirección forward),
> el tráfico de ataque tiende a tener payload más grande que el normal: las curvas
> de Ataque se concentran en log1p≈4.8–5, mientras que Normal presenta picos
> en valores más bajos (log1p≈0–3.5). Esto sugiere que el ataque envía paquetes
> forward de tamaño fijo y relativamente grande, consistente con patrones de flood
> con payload constante. En cambio, en las variables `bwd_*` (dirección backward),
> ocurre lo opuesto: el tráfico de ataque se concentra en log1p≈0, indicando
> payload backward casi nulo o inexistente, mientras que Normal presenta valores
> intermedios y altos. Las variables `flow_*` reflejan una combinación de ambos
> comportamientos, con Ataque concentrado alrededor de log1p≈4.5–5 en la mayoría
> de estadísticos. En resumen, el ataque es asimétrico: payload forward grande
> y payload backward nulo, patrón típico de ataques unidireccionales como
> UDP flood o HTTP flood sin respuesta del servidor.

**¿Alguna variable de payload separa los grupos mejor que las de IAT, o viceversa?**
> Las variables IAT separan los grupos de forma más consistente y con menor
> solapamiento que las variables de payload. En IAT, 13 de 15 variables logran
> separación clara (Sí), con los grupos ubicados en zonas completamente distintas
> del eje X. En payload, solo alrededor de 7 de 15 variables alcanzan separación
> clara, mientras que varias muestran solapamiento parcial o nulo (por ejemplo,
> `fwd_pkts_payload.std`, `bwd_pkts_payload.min` y `flow_pkts_payload.min`).
> Sin embargo, dentro del grupo de payload, las variables `bwd_pkts_payload.max`,
> `bwd_pkts_payload.tot` y `bwd_pkts_payload.avg` ofrecen separación comparable
> a las mejores IAT, dado que Ataque colapsa en 0 mientras Normal se distribuye
> en valores altos. La ventaja de las IAT es que la separación es estructural
> (log1p≈0 vs log1p≈5–15) y se mantiene en todas las estadísticas (min, max,
> avg, tot), mientras que en payload la separación depende más de la dirección
> del flujo analizada.

---

## Sección 3 — Boxplots

**¿Las medianas de Ataque son claramente más altas o más bajas que Normal en alguna variable?**
> En todas las variables mostradas, la mediana de Ataque es drásticamente más baja
> que la de Normal. En `fwd_iat.avg`, `fwd_iat.std`, `bwd_iat.std` y `flow_iat.std`
> la mediana de Ataque cae prácticamente en 0, mientras que Normal tiene medianas
> entre log1p≈12 y ≈12.5. La única excepción parcial es `flow_iat.avg`, donde la
> caja de Ataque es visible (mediana ≈1.2) aunque sigue siendo muy inferior a la
> de Normal (≈11.5). Esta diferencia sistemática confirma que el tráfico de ataque
> tiene intervalos entre paquetes consistentemente menores que el tráfico normal.

**¿Qué variable IAT muestra la mayor separación entre los dos grupos?**
> `fwd_iat.avg` y `bwd_iat.avg` muestran la mayor separación entre grupos: Normal
> tiene una caja amplia entre log1p≈7.5 y ≈15, con mediana ≈12, mientras que la
> caja de Ataque es prácticamente invisible al nivel de 0, con el bigote superior
> apenas alcanzando log1p≈0.5. `flow_iat.avg` es la variable donde la separación
> es más visualmente evidente porque la caja de Ataque sí es visible (en naranja),
> lo que permite apreciar con claridad que los dos grupos no se solapan en absoluto
> a nivel de cajas intercuartílicas.

**¿Hay muchos outliers en algún grupo? ¿En cuál variable?**
> El grupo Ataque concentra la mayor cantidad de outliers en todas las variables,
> dado que su distribución está tan comprimida en 0 que cualquier valor moderado
> se aleja del IQR y es marcado como outlier. Las variables con nube de outliers
> más densa son `fwd_iat.avg` y `fwd_iat.std`, donde se observan columnas
> compactas de puntos que alcanzan hasta log1p≈19. En `bwd_iat.avg` y
> `bwd_iat.std` también hay outliers abundantes en Ataque, llegando a log1p≈17–18.
> En el grupo Normal los outliers son escasos y solo se aprecian algunos puntos
> aislados en la parte baja (cerca de 0) en variables como `bwd_iat.avg` y
> `flow_iat.avg`, representando flujos normales con tiempos de llegada
> excepcionalmente cortos.
---

### 3.2 Boxplots Payload — Normal vs Ataque

**¿Las medianas de Ataque son claramente más altas o más bajas que Normal?**
> El comportamiento es opuesto según la variable. En `fwd_pkts_payload.avg` y
> `flow_pkts_payload.avg`, la mediana de Ataque (≈4.8) es notablemente más alta
> que la de Normal (≈3.4 y ≈3.8 respectivamente), confirmando que el ataque
> envía paquetes forward de mayor tamaño promedio. En `bwd_pkts_payload.avg`,
> la mediana de Ataque cae prácticamente en 0, muy por debajo de Normal (≈4.0),
> evidenciando la ausencia de payload en dirección backward durante el ataque.
> En las variables `.tot`, el patrón se invierte también: `fwd_pkts_payload.tot`
> de Ataque tiene mediana ≈4.8 (similar o ligeramente superior a Normal ≈4.9),
> mientras que `bwd_pkts_payload.tot` y `flow_pkts_payload.tot` de Ataque
> tienen mediana en 0 y ≈4.9 respectivamente, con Normal en ≈5.0 en ambos casos.
> En resumen, Ataque tiene medianas más altas en forward y más bajas en backward.

**¿Qué variable de payload muestra la mayor separación entre grupos?**
> `bwd_pkts_payload.avg` es la variable con mayor separación entre grupos: la
> caja de Normal ocupa el rango log1p≈2–4.5 con mediana ≈4.0, mientras que la
> caja de Ataque colapsa completamente en 0, sin solapamiento alguno entre los
> IQR de ambos grupos. `bwd_pkts_payload.tot` muestra una separación similar,
> con Normal entre log1p≈4.5 y ≈10.5 y Ataque con mediana en 0. En contraste,
> las variables `fwd_*` y `flow_*` presentan mayor solapamiento entre cajas,
> lo que reduce su poder discriminativo individual.

**¿El tráfico normal tiene más variabilidad (caja más grande) que el tráfico de ataque?**
> Depende de la variable. En `bwd_pkts_payload.avg`, `bwd_pkts_payload.tot` y
> `flow_pkts_payload.tot`, Normal tiene cajas claramente más grandes que Ataque,
> reflejando mayor variabilidad en el payload backward de flujos legítimos. Sin
> embargo, en `fwd_pkts_payload.avg`, `fwd_pkts_payload.tot` y
> `flow_pkts_payload.avg`, la caja de Ataque es igual o más amplia que la de
> Normal, con bigotes que se extienden desde 0 hasta log1p≈7, lo que indica
> que aunque la mayoría del tráfico de ataque es homogéneo, existe una cola de
> flujos atípicos con payload forward muy variable. En general, Normal es más
> variable en dirección backward y Ataque en dirección forward.

---

### 3.3 Boxplots por tipo de ataque — vista granular

**¿Qué tipo de ataque se diferencia más del resto en variables IAT?**
> El tipo de ataque con caja naranja oscuro/marrón (visible en posiciones
> intermedias del eje X) se diferencia más del resto en variables IAT,
> especialmente en `flow_iat.min`, `flow_iat.max`, `flow_iat.avg` y
> `flow_iat.tot`, donde su caja se ubica en valores log1p≈5–10, claramente
> por encima de los demás tipos de ataque que colapsan en 0. Esto sugiere
> que este tipo de ataque tiene intervalos entre paquetes no nulos, a
> diferencia del comportamiento flood del resto. El tipo rojo (visible en
> `fwd_iat.avg` y `bwd_iat.avg`) también se distingue, con mediana alrededor
> de log1p≈10–11, comportándose más parecido al tráfico normal que al resto
> de ataques.

**¿Qué tipo de ataque se diferencia más del resto en variables payload?**
> En las variables payload, el tipo verde oscuro destaca claramente en
> `bwd_pkts_payload.max`, `bwd_pkts_payload.tot` y `flow_pkts_payload.max`,
> con cajas ubicadas en log1p≈5–6, muy por encima del resto de tipos de
> ataque que se concentran en 0. Igualmente, en `fwd_pkts_payload.std` y
> `flow_pkts_payload.std`, el tipo rojo presenta valores elevados (mediana
> ≈4–5) mientras los demás tipos permanecen en 0. Estos dos tipos (verde y
> rojo) son los que más se alejan del patrón dominante de payload nulo
> observado en la mayoría de ataques.

**¿Hay tipos de ataque con comportamiento similar entre sí? ¿Cuáles?**
> La mayoría de tipos de ataque comparten un comportamiento muy similar: cajas
> completamente colapsadas en log1p≈0 tanto en variables IAT como en variables
> payload backward. Los tipos azul claro, beige/arena y azul oscuro son
> prácticamente indistinguibles entre sí en casi todas las variables, con
> medianas en 0, IQR mínimo y solo outliers dispersos hacia arriba. Este
> comportamiento homogéneo sugiere que corresponden a variantes del mismo
> patrón de ataque de alta frecuencia (flood), diferenciándose únicamente
> en la densidad de sus outliers superiores.

**¿El tráfico normal (`MQTT_Publish`, `Thing_Speak`, `Wipro_bulb`) forma un grupo
visualmente separado de los ataques?**
> Sí, de forma muy clara en las variables IAT. Los tipos de tráfico normal
> (cajas de mayor altura, generalmente en los primeros o últimos lugares del
> eje X) presentan cajas amplias ubicadas consistentemente en log1p≈5–15 en
> todas las variables `fwd_iat`, `bwd_iat` y `flow_iat`, sin solapamiento
> con la mayoría de tipos de ataque que están en 0. En variables payload la
> separación es menos absoluta: los tipos normales tienen cajas en log1p≈3–7
> que se solapan parcialmente con los tipos de ataque verde y rojo, que
> también muestran payload no nulo. En resumen, IAT es el grupo de variables
> donde el tráfico normal queda más claramente aislado del conjunto de ataques.

---

## Sección 4 — Outliers

### 4.1 Tabla de outliers

**Variables con más del 10% de outliers globales:**
> Son 8 variables las que superan el 10% de outliers globales, ordenadas de mayor
> a menor: `flow_pkts_payload.avg` (30.96%), `fwd_pkts_payload.avg` (23.11%),
> `fwd_pkts_payload.max` (23.11%), `fwd_pkts_payload.tot` (23.11%),
> `fwd_pkts_payload.min` (23.11%), `flow_pkts_payload.tot` (23.11%),
> `flow_pkts_payload.max` (22.92%) y `flow_pkts_payload.min` (16.95%).
> Todas pertenecen al grupo payload, lo que indica que estas variables tienen
> distribuciones especialmente irregulares con subpoblaciones alejadas del
> comportamiento central.

**¿Los outliers se concentran más en tráfico Normal o en Ataque?**
> Depende del grupo de variables. En las variables IAT, los outliers se
> concentran masivamente en Normal: `fwd_iat.min` alcanza 19.44% en Normal
> vs 5.18% en Ataque, y `bwd_iat.min` llega a 21.92% en Normal vs 4.83%
> en Ataque. Esto es coherente con lo observado en los boxplots: como el
> tráfico de ataque IAT colapsa en 0, cualquier valor moderado se convierte
> en outlier, pero en términos porcentuales la cola alta de Normal genera
> más outliers. En las variables payload, en cambio, los outliers se
> concentran más en Ataque: `flow_pkts_payload.avg` tiene 23.15% en Ataque
> vs 9.75% en Normal, y las variables `fwd_pkts_payload.*` presentan
> 14.42% en Ataque vs apenas 0.99%–10.45% en Normal.

**¿Hay alguna variable donde el % de outliers en Ataque sea mucho mayor que en Normal?**
> Sí, destacan dos casos extremos. `flow_pkts_payload.avg` es la más
> llamativa: 23.15% en Ataque vs 9.75% en Normal, una diferencia de
> más de 13 puntos porcentuales. `flow_pkts_payload.std` también es
> notable: 23.18% en Ataque vs 8.89% en Normal, siendo además la
> variable con menor % global (4.79%) lo que hace aún más sorprendente
> su concentración en Ataque. Las variables `fwd_pkts_payload.*` (min,
> avg, max, tot) muestran todas 14.42% en Ataque frente a valores entre
> 0.99% y 10.45% en Normal, confirmando que el tráfico de ataque genera
> sistemáticamente más outliers en las variables de payload forward
> que el tráfico normal.

---

### 4.2 Scatterplot ritmo vs tamaño

**¿Los grupos se separan visualmente en el scatter?**
> Sí, de forma considerable aunque no perfecta. La separación más clara
> es en el eje X (ritmo): los ataques automatizados como `DOS_SYN_Hping`,
> todos los tipos NMAP y `ARP_poisioning` se agrupan fuertemente en
> log1p(flow_iat.avg)≈0–1, formando columnas verticales en el extremo
> izquierdo. El tráfico normal (`MQTT_Publish`, `Thing_Speak`, `Wipro_bulb`)
> se distribuye en valores de ritmo más altos (log1p≈10–18). Sin embargo,
> en el eje Y (volumen) los grupos se solapan más: varios tipos de ataque
> y tráfico normal comparten rangos similares de payload total.

**¿En qué cuadrante se concentra el tráfico normal?**
- [ ] Rápido + con datos (arriba-izquierda)
- [x] Lento + con datos (arriba-derecha)
- [ ] Rápido + vacío (abajo-izquierda)
- [ ] Lento + vacío (abajo-derecha)
> `MQTT_Publish` (azul oscuro) se concentra en log1p(iat)≈14–18 con
> payload log1p≈4.7 (constante). `Thing_Speak` (azul claro) aparece
> disperso entre log1p(iat)≈8–16 con payload variable. `Wipro_bulb`
> (naranja) se distribuye en log1p(iat)≈10–18 con payload entre 5 y 14.

**¿En qué cuadrante se concentra `DOS_SYN_Hping`?**
> Rápido + poco volumen (abajo-izquierda): sus puntos forman una línea
> horizontal densa en log1p(iat)≈0 con payload constante en log1p≈4.8,
> reflejando el envío masivo de paquetes SYN idénticos a máxima velocidad.

**¿En qué cuadrante se concentra `DDOS_Slowloris`?**
> Lento + con datos (arriba-derecha, pero con dispersión): a diferencia
> del resto de ataques, `DDOS_Slowloris` (verde oscuro) aparece distribuido
> entre log1p(iat)≈13–16 con payload entre log1p≈5 y ≈8, lo que lo ubica
> sorprendentemente cerca del cuadrante del tráfico normal. Su ritmo lento
> es parte de su estrategia: mantiene conexiones abiertas enviando datos
> esporádicamente para agotar recursos del servidor.

**¿Hay tipos de ataque que se solapan con el tráfico normal en el scatter?**
> Sí, dos tipos presentan solapamiento notable. `DDOS_Slowloris` es el
> más problemático: sus puntos se mezclan visualmente con `Wipro_bulb`
> y `Thing_Speak` en la zona log1p(iat)≈13–16, lo que lo haría difícil
> de separar usando solo estas dos variables. `Metasploit_Brute_Force_SSH`
> (rojo) también aparece en la zona central-derecha del scatter
> (log1p(iat)≈7–10, payload≈7–8), solapándose parcialmente con tráfico
> normal de volumen similar. El resto de ataques NMAP y DOS quedan
> claramente separados en el extremo izquierdo sin solapamiento.

---

## Sección 5 — Correlación entre variables

### 5.1 Correlación IAT

**¿Hay pares de variables IAT con correlación > 0.9?**
> Sí, se identifican los siguientes pares con correlación ≥ 0.90:
> - `fwd_iat.max` ↔ `fwd_iat.std`: 0.97
> - `fwd_iat.max` ↔ `flow_iat.max`: 1.00
> - `fwd_iat.max` ↔ `flow_iat.std`: 0.98
> - `fwd_iat.std` ↔ `flow_iat.max`: 0.97
> - `fwd_iat.std` ↔ `flow_iat.std`: 0.97
> - `fwd_iat.avg` ↔ `flow_iat.std`: 0.83 (no supera 0.9, pero se acerca)
> - `bwd_iat.max` ↔ `bwd_iat.std`: 0.94
> - `bwd_iat.avg` ↔ `bwd_iat.std`: 0.92
> - `flow_iat.max` ↔ `flow_iat.std`: 0.98
> - `flow_iat.avg` ↔ `flow_iat.std`: 0.90
> - `flow_iat.tot` ↔ `flow_iat.tot` (con `bwd_iat.tot`): 0.70 (no supera)
> Los grupos más redundantes son: {`fwd_iat.max`, `fwd_iat.std`,
> `flow_iat.max`, `flow_iat.std`} con correlaciones cruzadas todas ≥ 0.97,
> y {`bwd_iat.avg`, `bwd_iat.std`} con 0.92.

**¿Las variables `.tot` están muy correlacionadas con `.avg`?**
- [ ] Sí — son redundantes
- [x] No — aportan información diferente
> `fwd_iat.tot` ↔ `fwd_iat.avg` = 0.15; `bwd_iat.tot` ↔ `bwd_iat.avg` = 0.35;
> `flow_iat.tot` ↔ `flow_iat.avg` = 0.20. Todas las correlaciones son bajas,
> lo que indica que `.tot` captura la acumulación total del tiempo entre paquetes
> mientras `.avg` refleja el ritmo típico, siendo ambas métricas complementarias.

**¿Las variables `fwd_iat.*` están muy correlacionadas con `flow_iat.*`?**
- [x] Sí — son redundantes
- [ ] No — aportan información diferente
> `fwd_iat.max` ↔ `flow_iat.max` = 1.00; `fwd_iat.std` ↔ `flow_iat.std` = 0.97;
> `fwd_iat.avg` ↔ `flow_iat.avg` = 0.86; `fwd_iat.std` ↔ `flow_iat.max` = 0.97.
> La correlación perfecta entre `fwd_iat.max` y `flow_iat.max` sugiere que son
> la misma medida en la práctica. En general, las variables `flow_iat.*` parecen
> estar dominadas por el comportamiento forward, lo que las hace altamente
> redundantes con sus contrapartes `fwd_iat.*` y candidatas a eliminación
> en un proceso de selección de features.

---

### 5.2 Correlación Payload

**¿Hay pares de variables payload con correlación > 0.9?**
> Sí, se identifican los siguientes pares con correlación ≥ 0.90:
> - `fwd_pkts_payload.min` ↔ `fwd_pkts_payload.avg`: 0.80 (no supera, pero cercano)
> - `fwd_pkts_payload.max` ↔ `fwd_pkts_payload.std`: 0.91
> - `bwd_pkts_payload.max` ↔ `bwd_pkts_payload.avg`: 0.90
> - `bwd_pkts_payload.max` ↔ `bwd_pkts_payload.std`: 0.97
> - `bwd_pkts_payload.avg` ↔ `bwd_pkts_payload.std`: 0.89 (muy cercano)
> - `bwd_pkts_payload.tot` ↔ `flow_pkts_payload.tot`: 0.99
> - `flow_pkts_payload.max` ↔ `bwd_pkts_payload.max`: 0.97
> - `flow_pkts_payload.max` ↔ `bwd_pkts_payload.std`: 0.95
> - `flow_pkts_payload.max` ↔ `flow_pkts_payload.std`: 0.90
> - `flow_pkts_payload.avg` ↔ `bwd_pkts_payload.avg`: 0.79 (no supera)
> - `flow_pkts_payload.std` ↔ `bwd_pkts_payload.std`: 0.84 (no supera)
> El grupo más redundante es {`bwd_pkts_payload.max`, `bwd_pkts_payload.std`,
> `flow_pkts_payload.max`} con correlaciones cruzadas todas ≥ 0.95, y el par
> `bwd_pkts_payload.tot` ↔ `flow_pkts_payload.tot` con correlación casi
> perfecta de 0.99, siendo claramente candidatos a eliminación conjunta.

**¿Las variables `.tot` están muy correlacionadas con `.avg` en payload?**
- [ ] Sí — son redundantes
- [x] No — aportan información diferente
> `fwd_pkts_payload.tot` ↔ `fwd_pkts_payload.avg` = 0.16;
> `bwd_pkts_payload.tot` ↔ `bwd_pkts_payload.avg` = 0.14;
> `flow_pkts_payload.tot` ↔ `flow_pkts_payload.avg` = 0.16.
> Las tres correlaciones son muy bajas (≤ 0.16), lo que confirma que `.tot`
> captura el volumen acumulado total de bytes transmitidos mientras `.avg`
> refleja el tamaño típico por paquete. Ambas métricas son complementarias
> y no redundantes, por lo que conservar ambas en un modelo aportaría
> información independiente.

---

### 5.3 Pares redundantes identificados

Registrar los pares con r > 0.9 y cuál conservar de cada par:

| Par | Correlación | ¿Cuál conservar? |
|---|---|---|
| `fwd_iat.tot` vs `flow_iat.tot` | 0.9997 | `fwd_iat.tot` — más específico en dirección |
| `fwd_iat.max` vs `flow_iat.max` | 0.9966 | `fwd_iat.max` — flow está dominado por fwd |
| `bwd_pkts_payload.tot` vs `flow_pkts_payload.tot` | 0.9937 | `flow_pkts_payload.tot` — captura el flujo completo |
| `flow_iat.max` vs `flow_iat.std` | 0.9815 | `flow_iat.max` — más interpretable |
| `fwd_iat.max` vs `flow_iat.std` | 0.9804 | `fwd_iat.max` — ya retenido arriba; eliminar `flow_iat.std` |
| `bwd_pkts_payload.max` vs `bwd_pkts_payload.std` | 0.9740 | `bwd_pkts_payload.max` — más interpretable |
| `fwd_iat.max` vs `fwd_iat.std` | 0.9722 | `fwd_iat.max` — ya retenido; eliminar `fwd_iat.std` |
| `bwd_pkts_payload.max` vs `flow_pkts_payload.max` | 0.9719 | `flow_pkts_payload.max` — ya retenido arriba |
| `fwd_iat.std` vs `flow_iat.std` | 0.9718 | Ambas candidatas a eliminar — retenidas indirectamente |
| `fwd_iat.std` vs `flow_iat.max` | 0.9689 | `flow_iat.max` — ya retenido; eliminar `fwd_iat.std` |
| `bwd_pkts_payload.std` vs `flow_pkts_payload.max` | 0.9453 | `flow_pkts_payload.max` — ya retenido arriba |
| `bwd_iat.max` vs `bwd_iat.std` | 0.9388 | `bwd_iat.max` — más interpretable |
| `bwd_iat.avg` vs `bwd_iat.std` | 0.9228 | `bwd_iat.avg` — más interpretable |
| `fwd_pkts_payload.max` vs `fwd_pkts_payload.std` | 0.9076 | `fwd_pkts_payload.max` — más interpretable |
| `flow_iat.avg` vs `flow_iat.std` | 0.9032 | `flow_iat.avg` — más interpretable |

---

## Sección 6 — Tráfico automatizado

### 6.1 Tabla resumen std ≈ 0

**¿El porcentaje de flujos con std ≈ 0 es mayor en Ataque o en Normal?**
> En Ataque de forma abrumadora en todas las variables. Las variables IAT
> muestran los valores más extremos: `fwd_iat.std` tiene 97.37% en Ataque
> vs 34.72% en Normal, `bwd_iat.std` 97.70% vs 35.33%, y `flow_iat.std`
> 94.82% vs apenas 5.11%. En payload el patrón se repite: `fwd_pkts_payload.std`
> alcanza 97.49% en Ataque vs 35.20% en Normal, y `bwd_pkts_payload.std`
> 95.49% vs 6.02%. La única excepción relativa es `flow_pkts_payload.std`,
> donde Ataque tiene 17.93% vs 3.22% en Normal — sigue siendo mayor en
> Ataque pero con una brecha mucho menor que el resto.

**¿Qué es más constante en los ataques: el ritmo (IAT) o el tamaño (payload)?**
> Ambos grupos son igualmente constantes en Ataque, con porcentajes de
> std ≈ 0 superiores al 94% en casi todas las variables. Sin embargo,
> el ritmo (IAT) es ligeramente más constante: `bwd_iat.std` alcanza
> 97.70% y `fwd_iat.std` 97.37%, mientras que en payload `fwd_pkts_payload.std`
> llega a 97.49% y `bwd_pkts_payload.std` a 95.49%. La diferencia más
> notable está en `flow_pkts_payload.std` (17.93%), que es la única variable
> con std ≈ 0 significativamente por debajo del 95% en Ataque, sugiriendo
> que el tamaño de payload a nivel de flujo completo tiene algo más de
> variabilidad que el ritmo de llegada. En síntesis, los ataques envían
> paquetes a intervalos perfectamente regulares y de tamaño fijo,
> comportamiento consistente con herramientas automatizadas de flood.

---

### 6.2 Detalle por tipo de ataque

**¿Qué tipos de ataque tienen casi todos sus flujos con IAT std ≈ 0? (ritmo de máquina)**

| Tipo de ataque | % IAT std ≈ 0 | Interpretación |
|---|---|---|
| `DOS_SYN_Hping` | 100.00% | Flood puro: paquetes SYN enviados a ritmo perfectamente constante por herramienta automatizada |
| `NMAP_OS_DETECTION` | 100.00% | Escaneo de SO con timing fijo; NMAP envía probes a intervalos deterministas |
| `NMAP_XMAS_TREE_SCAN` | 99.90–99.95% | Escaneo de puertos con flags especiales; ritmo de máquina casi perfecto |
| `NMAP_TCP_scan` | 99.50% | Escaneo TCP sistemático con intervalos constantes entre paquetes |
| `NMAP_FIN_SCAN` | 92.86–96.43% | Variante de escaneo NMAP; levemente menos regular que los anteriores |
| `NMAP_UDP_SCAN` | 94.13–94.32% | Escaneo UDP; timing regular pero con algo más de variabilidad por naturaleza del protocolo |

**¿Qué tipos de ataque tienen casi todos sus flujos con payload std ≈ 0? (paquetes idénticos)**

| Tipo de ataque | % payload std ≈ 0 | Interpretación |
|---|---|---|
| `DOS_SYN_Hping` | 100.00% (fwd/bwd) | Paquetes SYN de tamaño fijo; payload completamente homogéneo por diseño del ataque |
| `NMAP_OS_DETECTION` | 100.00% (fwd/bwd) | Probes de tamaño estándar definido por el protocolo de detección de SO |
| `NMAP_XMAS_TREE_SCAN` | 99.80–99.95% | Paquetes de escaneo con estructura fija; casi sin variación en tamaño |
| `NMAP_TCP_scan` | 99.80% | Paquetes TCP de escaneo uniformes en tamaño |
| `NMAP_UDP_SCAN` | 94.09–99.73% | Paquetes UDP de tamaño estándar; ligera variabilidad en dirección flow |
| `NMAP_FIN_SCAN` | 85.71–96.43% | Paquetes FIN uniformes aunque con algo más de dispersión en flow |

**¿El tráfico normal tiene más variabilidad que los ataques en estas variables?**
> Sí, de forma muy clara. Los tres tipos de tráfico normal muestran porcentajes
> de std ≈ 0 significativamente menores que los ataques de alta frecuencia.
> `MQTT_Publish` es el más variable de todos con valores entre 0.22% y 0.36%,
> prácticamente ningún flujo tiene ritmo o tamaño constante. `DDOS_Slowloris`
> y `Metasploit_Brute_Force_SSH` también presentan baja constancia (0.56–19.66%
> y 10.81–24.32% respectivamente), comportándose más parecido al tráfico normal
> que al resto de ataques. Esto confirma que la variabilidad en IAT y payload
> es una característica propia del tráfico legítimo, mientras que la uniformidad
> extrema es la firma de los ataques automatizados tipo NMAP y DOS/SYN.

---

### 6.3 Violinplot de variabilidad

**¿Los violines del grupo Ataque son más delgados y pegados al cero que los de Normal?**
> Sí, de forma muy marcada en 5 de las 6 variables. En `fwd_iat.std`,
> `bwd_iat.std` y `flow_iat.std`, el violín de Ataque es prácticamente
> una línea vertical pegada a log1p≈0, con una pequeñísima protuberancia
> en la base que refleja la masa concentrada en cero; en contraste, los
> violines de Normal son amplios y multimodales, con expansiones visibles
> en log1p≈11–12, ≈15–16 y ≈17–18, ocupando casi todo el rango disponible.
> En `fwd_pkts_payload.std` y `bwd_pkts_payload.std` el patrón se repite:
> Ataque colapsa en 0 mientras Normal muestra violines con múltiples
> ensanchamientos entre log1p≈2 y ≈6. La única excepción notable es
> `flow_pkts_payload.std`, donde el violín de Ataque es sorprendentemente
> ancho y centrado alrededor de log1p≈4–4.5, siendo la única variable
> donde Ataque no colapsa en cero.

**¿Hay algún tipo de ataque con violín sorprendentemente ancho (alta variabilidad)?**
> Sí, `flow_pkts_payload.std` es el caso más llamativo: el violín de
> Ataque tiene una anchura comparable al de Normal en el rango log1p≈3.5–5,
> con la mediana alrededor de log1p≈4.4, lo que indica que una fracción
> significativa del tráfico de ataque presenta alta variabilidad en el
> tamaño de payload a nivel de flujo completo. Esto es consistente con
> lo observado en la tabla de std ≈ 0, donde `flow_pkts_payload.std`
> solo alcanzaba 17.93% en Ataque — muy por debajo del 94–100% del
> resto de variables. Este comportamiento podría corresponder a los
> tipos de ataque como `DDOS_Slowloris` o `Metasploit_Brute_Force_SSH`,
> que según los datos anteriores son los menos uniformes en payload
> dentro del grupo Ataque.

---

## Sección 7 — Resumen y conclusiones

### 7.1 Variables candidatas identificadas

Variables con separación clara entre grupos (`diferencia > 1` en escala log1p) y `pct_ceros < 50%`:

| Variable | Ratio Ataque/Normal (mediana) | % ceros | ¿Incluir en modelo? |
|---|---|---|---|
| `flow_iat.avg` | ≈ 0.00 (68,176 vs 3.81) | 12.98% | Sí |
| `flow_iat.min` | ≈ 0.02 (183 vs 3.81) | 13.01% | Sí |
| `flow_iat.max` | ≈ 0.00 (268,883 vs 3.81) | 12.98% | Sí |
| `flow_iat.tot` | ≈ 0.00 (882,167 vs 3.81) | 12.98% | Sí |
| `fwd_pkts_payload.avg` | 3.91 (30.71 vs 120.00) | 5.53% | Sí |
| `fwd_pkts_payload.min` | NaN — normal ≈ 0, ataque ≈ 120 | 14.05% | Sí |
| `fwd_pkts_payload.max` | 3.33 (36.00 vs 120.00) | 5.53% | Sí |
| `flow_pkts_payload.std` | 5.30 (16.00 vs 84.85) | 16.43% | Sí |

**¿Las variables más útiles son principalmente IAT, payload, o ambas por igual?**
> Ambas aportan, pero con roles distintos. Las variables **IAT** (`flow_iat.*`) son las más
> potentes como discriminador primario: tienen separación estructural y consistente, con
> la mediana de ataque colapsada en ≈3.81 µs frente a decenas o cientos de miles en tráfico
> normal, y porcentajes de ceros manejables (≈13%). Las variables de **payload forward**
> (`fwd_pkts_payload.avg`, `.min`, `.max`) aportan señal complementaria y ortogonal — el
> ataque tiene payload fijo de 120 bytes mientras el normal varía — lo que aumenta la
> capacidad discriminativa del modelo especialmente para casos como `DDOS_Slowloris` que
> tiene IAT alto similar al normal pero sigue enviando paquetes de tamaño fijo. En conjunto,
> un modelo que combine ambas familias superará a uno que use solo una de ellas.

---

### 7.2 Respuestas a las preguntas guía

**1. ¿Los ataques tienen paquetes más pequeños o más grandes que el tráfico normal?**
> Depende de la dirección del flujo. En la dirección **forward**, los ataques tienen paquetes
> más **grandes**: el payload forward se concentra en log1p≈4.8 (≈120 bytes de mediana)
> mientras el tráfico normal presenta medianas de 30–36 bytes y mayor dispersión. En la
> dirección **backward**, ocurre lo contrario: los ataques tienen payload prácticamente **nulo**
> (mediana ≈0), mientras el tráfico normal presenta valores intermedios (52–68 bytes). Este
> patrón asimétrico — payload forward grande y backward nulo — es la firma de ataques
> unidireccionales como UDP flood o SYN flood, donde el atacante envía sin esperar respuesta.
>
> Variables que respaldan esta respuesta: `fwd_pkts_payload.avg`, `fwd_pkts_payload.min`,
> `fwd_pkts_payload.max`, `bwd_pkts_payload.avg`, `bwd_pkts_payload.tot`

---

**2. ¿Los tiempos entre paquetes son más bajos en ataques?**
> Sí, de forma drástica y consistente en prácticamente todas las variables IAT. La mediana
> de ataque es ≈0 en todas las variables `fwd_iat.*` y `bwd_iat.*`, y ≈3.81 µs en las
> `flow_iat.*`, frente a medianas de entre 183 µs y 882,167 µs en tráfico normal. Los ataques
> de alta frecuencia (`DOS_SYN_Hping`, todas las variantes NMAP) envían paquetes a intervalos
> prácticamente nulos, con 97–100% de flujos con `std ≈ 0`, lo que confirma un ritmo de máquina
> perfectamente regular.
>
> Excepción identificada: `DDOS_Slowloris` presenta IAT alto (log1p≈13–16), deliberadamente
> similar al tráfico normal. Su estrategia consiste en mantener conexiones abiertas enviando
> datos esporádicamente para agotar recursos del servidor, por lo que su firma en IAT es
> opuesta al resto de ataques. `Metasploit_Brute_Force_SSH` también tiene IAT moderado
> (log1p≈7–10), diferenciándose del comportamiento flood dominante.
>
> Variables que respaldan esta respuesta: `flow_iat.avg`, `flow_iat.max`, `fwd_iat.avg`,
> `bwd_iat.avg`, `flow_iat.tot`

---

**3. ¿Hay tráfico automatizado o repetitivo? ¿En qué tipos?**
> Sí, de forma inequívoca. El 94–100% de los flujos de los principales tipos de ataque tienen
> `std ≈ 0` tanto en IAT como en payload, lo que significa intervalos entre paquetes y tamaños
> perfectamente constantes — comportamiento imposible en tráfico humano legítimo. Esto indica
> herramientas automatizadas con timing determinista (hping3 para SYN flood, NMAP para escaneos).
> El tráfico normal muestra menos de 0.36% de flujos con std ≈ 0 en `MQTT_Publish`, lo que
> contrasta radicalmente con los ataques.
>
> Tipos confirmados con std ≈ 0 (ritmo y tamaño de máquina):
> - `DOS_SYN_Hping`: 100% en todas las variables IAT y payload
> - `NMAP_OS_DETECTION`: 100% en IAT y payload
> - `NMAP_XMAS_TREE_SCAN`: 99.80–99.95%
> - `NMAP_TCP_scan`: 99.50–99.80%
> - `NMAP_UDP_SCAN`: 94.09–99.73%
> - `NMAP_FIN_SCAN`: 85.71–96.43%
>
> Tipos con comportamiento menos uniforme (más parecido al tráfico normal):
> - `DDOS_Slowloris`: 0.56–19.66% (variabilidad alta, intencionalmente errático)
> - `Metasploit_Brute_Force_SSH`: 10.81–24.32%

---

**4. ¿Qué variables temporales o de payload parecen más útiles para un modelo de detección?**

| # | Variable | Razón |
|---|---|---|
| 1 | `flow_iat.avg` | Mayor separación en medianas (68,176 vs 3.81 µs); bajo % de ceros (13%); captura el ritmo típico del flujo completo |
| 2 | `fwd_pkts_payload.avg` | Ratio 3.91 entre grupos; 120 bytes fijos en ataque vs variabilidad en normal; complementa IAT para capturar Slowloris |
| 3 | `flow_iat.max` | Correlación 1.00 con `fwd_iat.max`; captura el intervalo más largo del flujo — muy alto en normal, ≈0 en ataque |
| 4 | `bwd_pkts_payload.avg` | Mayor separación en payload backward: ataque en 0, normal en log1p≈2–4.5; IQR sin solapamiento |
| 5 | `flow_pkts_payload.std` | Ratio 5.30; único estadístico de variabilidad con separación clara; discrimina ataques con payload heterogéneo como Metasploit |

---

### Hallazgos no esperados o llamativos

> **`DDOS_Slowloris` como caso atípico crítico:** es el único tipo de ataque que se ubica
> en el cuadrante del tráfico normal en el scatter (IAT log1p≈13–16, payload log1p≈5–8),
> solapándose visualmente con `Wipro_bulb` y `Thing_Speak`. Un modelo entrenado únicamente
> con variables IAT fallaría en detectarlo; requiere features adicionales de payload o de
> duración de conexión.
>
> **Correlación perfecta `fwd_iat.max` ↔ `flow_iat.max` = 1.00:** indica que en este dataset
> el flujo completo está dominado por el comportamiento forward — las variables `flow_iat.*`
> no aportan información independiente respecto a `fwd_iat.*` y son candidatas directas a
> eliminación, reduciendo dimensionalidad sin pérdida de información.
>
> **`flow_pkts_payload.std` con violín ancho en Ataque:** es la única variable donde el grupo
> Ataque no colapsa en 0, con masa centrada en log1p≈4–4.5. Esto es inesperado dado el patrón
> dominante de uniformidad en ataques y puede reflejar la mezcla de tipos con diferente tamaño
> de payload (NMAP vs Metasploit vs Slowloris) en el cálculo a nivel de flujo completo.
>
> **Flujos de un solo paquete como mayoría:** el 85–91% de flujos en variables `fwd_iat.*`
> tienen valor 0, indicando que la mayor parte del tráfico capturado corresponde a flujos de
> un único paquete — muchos de ellos ataques de escaneo que envían un paquete por puerto.
> Esto implica que las variables IAT forward/backward, aunque altamente discriminativas en
> los flujos donde sí hay intervalo, tienen utilidad limitada por su densidad de ceros.

---

### Variables descartadas y razón

| Variable | Razón de descarte |
|---|---|
| `fwd_iat.min`, `fwd_iat.max`, `fwd_iat.tot`, `fwd_iat.avg`, `fwd_iat.std` | > 85% de ceros — flujos de un solo paquete dominan |
| `bwd_iat.min`, `bwd_iat.max`, `bwd_iat.tot`, `bwd_iat.avg`, `bwd_iat.std` | > 86% de ceros — flujos unidireccionales sin tráfico backward |
| `flow_iat.std` | 85.71% de ceros y redundante con `fwd_iat.std` (r = 0.97) |
| `fwd_pkts_payload.std` | 91.17% de ceros; Ataque concentrado en 0 — sin poder discriminativo |
| `bwd_pkts_payload.min` | 93.86% de ceros; ambos grupos en 0 — sin diferencia entre grupos |
| `bwd_pkts_payload.max`, `bwd_pkts_payload.tot`, `bwd_pkts_payload.avg`, `bwd_pkts_payload.std` | > 85% de ceros — flujos sin respuesta backward dominan |
| `flow_pkts_payload.min` | 83.05% de ceros; Normal y Ataque ambos concentrados en 0 |
| `flow_iat.max`, `flow_iat.tot` | Redundantes con `fwd_iat.max` (r ≈ 1.00) y `fwd_iat.tot` (r = 0.9997) |
| `fwd_iat.std`, `flow_iat.std` | Redundantes con `fwd_iat.max` / `flow_iat.max` (r ≥ 0.97) |
| `bwd_iat.std` | Redundante con `bwd_iat.max` (r = 0.94) y `bwd_iat.avg` (r = 0.92) |
| `bwd_pkts_payload.std`, `flow_pkts_payload.max` | Redundantes con `bwd_pkts_payload.max` (r = 0.97 y 0.97 respectivamente) |
| `bwd_pkts_payload.tot` | Redundante con `flow_pkts_payload.tot` (r = 0.9937) |
