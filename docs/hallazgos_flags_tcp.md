# Hallazgos del análisis de flags TCP - RT-IoT2022

## 1. Objetivo

Se revisaron variables relacionadas con el estado de las conexiones TCP para entender su distribución, la cantidad de ceros, su comportamiento por tipo de ataque (`Attack_type`) y su posible utilidad para modelos de detección de ataques.

## 2. Variables revisadas

| Variable | Significado sencillo |
|---|---|
| `flow_FIN_flag_count` | FIN: indica cierre de una conexión TCP. Puede aparecer cuando una comunicación termina o en escaneos tipo FIN. |
| `flow_SYN_flag_count` | SYN: indica inicio de una conexión TCP. Es importante en intentos de conexión, escaneos y ataques tipo SYN flood. |
| `flow_RST_flag_count` | RST: indica reinicio o corte abrupto de una conexión. Puede aparecer cuando una conexión falla, es rechazada o se interrumpe. |
| `flow_ACK_flag_count` | ACK: confirma recepción de paquetes. Aparece en conexiones establecidas o respuestas dentro de una comunicación TCP. |
| `fwd_PSH_flag_count` | PSH forward: indica envío inmediato de datos desde origen hacia destino. |
| `bwd_PSH_flag_count` | PSH backward: indica envío inmediato de datos desde destino hacia origen. |
| `fwd_URG_flag_count` | URG forward: indica datos urgentes desde origen hacia destino. |
| `bwd_URG_flag_count` | URG backward: indica datos urgentes desde destino hacia origen. |
| `flow_CWR_flag_count` | CWR: bandera usada para control de congestión TCP. |
| `flow_ECE_flag_count` | ECE: bandera usada para notificación de congestión TCP. |


## 3. Distribución de clases

El dataset cargado tiene **123.117 registros** y **84 columnas**. En este archivo no aparece una clase `Normal`; todos los registros pertenecen a valores de `Attack_type`. Por eso no se pudo hacer una comparación real Normal vs Ataque. y el análisis se enfocó en comparar los flags entre tipos de ataque.

| Attack_type                |   registros |
|:---------------------------|------------:|
| DOS_SYN_Hping              |       94659 |
| Thing_Speak                |        8108 |
| ARP_poisioning             |        7750 |
| MQTT_Publish               |        4146 |
| NMAP_UDP_SCAN              |        2590 |
| NMAP_XMAS_TREE_SCAN        |        2010 |
| NMAP_OS_DETECTION          |        2000 |
| NMAP_TCP_scan              |        1002 |
| DDOS_Slowloris             |         534 |
| Wipro_bulb                 |         253 |
| Metasploit_Brute_Force_SSH |          37 |
| NMAP_FIN_SCAN              |          28 |


## 4. Distribución de ceros

La siguiente tabla muestra cuántos ceros tiene cada flag. Un porcentaje alto de ceros significa que esa bandera casi no aparece en los flujos.

| variable            |   ceros |   no_ceros |   porcentaje_ceros |   porcentaje_no_ceros |
|:--------------------|--------:|-----------:|-------------------:|----------------------:|
| bwd_URG_flag_count  |  123117 |          0 |             100    |                  0    |
| flow_CWR_flag_count |  123046 |         71 |              99.94 |                  0.06 |
| flow_ECE_flag_count |  123045 |         72 |              99.94 |                  0.06 |
| fwd_URG_flag_count  |  121111 |       2006 |              98.37 |                  1.63 |
| flow_FIN_flag_count |  115053 |       8064 |              93.45 |                  6.55 |
| bwd_PSH_flag_count  |  113159 |       9958 |              91.91 |                  8.09 |
| fwd_PSH_flag_count  |  110612 |      12505 |              89.84 |                 10.16 |
| flow_RST_flag_count |   26553 |      96564 |              21.57 |                 78.43 |
| flow_ACK_flag_count |   22543 |     100574 |              18.31 |                 81.69 |
| flow_SYN_flag_count |   16791 |     106326 |              13.64 |                 86.36 |


## 5. Distribución general de cada flag

Resumen estadístico de los conteos por flag:

|                     |   count |   mean |     std |   min |   25% |   50% |   75% |   max |
|:--------------------|--------:|-------:|--------:|------:|------:|------:|------:|------:|
| flow_FIN_flag_count |  123117 | 0.1156 |  0.475  |     0 |     0 |     0 |     0 |    10 |
| flow_SYN_flag_count |  123117 | 0.9509 |  0.4743 |     0 |     1 |     1 |     1 |     8 |
| flow_RST_flag_count |  123117 | 0.7965 |  0.437  |     0 |     1 |     1 |     1 |    10 |
| flow_ACK_flag_count |  123117 | 2.6777 | 41.6549 |     0 |     1 |     1 |     1 | 11772 |
| fwd_PSH_flag_count  |  123117 | 0.3513 |  3.9516 |     0 |     0 |     0 |     0 |   864 |
| bwd_PSH_flag_count  |  123117 | 0.3936 |  6.0074 |     0 |     0 |     0 |     0 |  1446 |
| fwd_URG_flag_count  |  123117 | 0.0163 |  0.1266 |     0 |     0 |     0 |     0 |     1 |
| bwd_URG_flag_count  |  123117 | 0      |  0      |     0 |     0 |     0 |     0 |     0 |
| flow_CWR_flag_count |  123117 | 0.001  |  0.0447 |     0 |     0 |     0 |     0 |     4 |
| flow_ECE_flag_count |  123117 | 0.0007 |  0.0315 |     0 |     0 |     0 |     0 |     4 |


## 6. Comparación con `Attack_type`

Se calculó la presencia de cada flag por tipo de ataque. Presencia significa el porcentaje de registros de esa clase donde el flag es diferente de cero.

| Attack_type                |   flow_FIN_flag_count |   flow_SYN_flag_count |   flow_RST_flag_count |   flow_ACK_flag_count |   fwd_PSH_flag_count |   bwd_PSH_flag_count |   fwd_URG_flag_count |   bwd_URG_flag_count |   flow_CWR_flag_count |   flow_ECE_flag_count |
|:---------------------------|----------------------:|----------------------:|----------------------:|----------------------:|---------------------:|---------------------:|---------------------:|---------------------:|----------------------:|----------------------:|
| ARP_poisioning             |                 22.04 |                 24.4  |                 11.95 |                 23.45 |                22.45 |                22.12 |                  0   |                    0 |                  0    |                  0    |
| DDOS_Slowloris             |                 45.51 |                 99.06 |                 45.51 |                 97.94 |                97.94 |                28.09 |                  0   |                    0 |                  0    |                  0    |
| DOS_SYN_Hping              |                  0    |                100    |                 89.76 |                 89.76 |                 0    |                 0    |                  0   |                    0 |                  0    |                  0    |
| MQTT_Publish               |                  0.87 |                 99.81 |                 99.59 |                 99.98 |                99.73 |                99.64 |                  0   |                    0 |                  0    |                  0    |
| Metasploit_Brute_Force_SSH |                 75.68 |                 78.38 |                  2.7  |                 78.38 |                78.38 |                75.68 |                  0   |                    0 |                  0    |                  0    |
| NMAP_FIN_SCAN              |                 89.29 |                  3.57 |                  0    |                  3.57 |                 3.57 |                 3.57 |                  0   |                    0 |                  0    |                  0    |
| NMAP_OS_DETECTION          |                  0    |                  0    |                100    |                100    |                 0    |                 0    |                  0   |                    0 |                  0    |                  0    |
| NMAP_TCP_scan              |                  0    |                 99.8  |                 99.8  |                100    |                 0.2  |                 0    |                  0   |                    0 |                  0    |                  0    |
| NMAP_UDP_SCAN              |                  5.37 |                  5.48 |                  5.29 |                  5.37 |                 5.37 |                 0.08 |                  0   |                    0 |                  0    |                  0    |
| NMAP_XMAS_TREE_SCAN        |                 99.85 |                  0.05 |                 99.2  |                 99.25 |                99.85 |                 0.05 |                 99.8 |                    0 |                  0    |                  0    |
| Thing_Speak                |                 45.93 |                 46.57 |                 13    |                 46.6  |                46.5  |                46.5  |                  0   |                    0 |                  0.62 |                  0.63 |
| Wipro_bulb                 |                 60.87 |                 63.24 |                 43.87 |                 69.57 |                62.85 |                63.64 |                  0   |                    0 |                  8.3  |                  8.3  |


También se calcularon los promedios por tipo de ataque:

| Attack_type                |   flow_FIN_flag_count |   flow_SYN_flag_count |   flow_RST_flag_count |   flow_ACK_flag_count |   fwd_PSH_flag_count |   bwd_PSH_flag_count |   fwd_URG_flag_count |   bwd_URG_flag_count |   flow_CWR_flag_count |   flow_ECE_flag_count |
|:---------------------------|----------------------:|----------------------:|----------------------:|----------------------:|---------------------:|---------------------:|---------------------:|---------------------:|----------------------:|----------------------:|
| ARP_poisioning             |                0.4317 |                0.4885 |                0.1708 |               12.0378 |               1.9538 |               2.4702 |                0     |                    0 |                0      |                 0     |
| DDOS_Slowloris             |                1.5449 |                2.015  |                0.4551 |                9.5655 |               2.9532 |               1.1292 |                0     |                    0 |                0      |                 0     |
| DOS_SYN_Hping              |                0      |                1      |                0.8976 |                0.8976 |               0      |               0      |                0     |                    0 |                0      |                 0     |
| MQTT_Publish               |                0.0111 |                1.9969 |                0.9959 |               17.4503 |               2.992  |               3.7487 |                0     |                    0 |                0      |                 0     |
| Metasploit_Brute_Force_SSH |                1.4865 |                1.5676 |                0.027  |               21.4595 |               5.027  |               4.2162 |                0     |                    0 |                0      |                 0     |
| NMAP_FIN_SCAN              |                0.9286 |                0.0714 |                0      |                0.3571 |               0.0357 |               0.0357 |                0     |                    0 |                0      |                 0     |
| NMAP_OS_DETECTION          |                0      |                0      |                1      |                1      |               0      |               0      |                0     |                    0 |                0      |                 0     |
| NMAP_TCP_scan              |                0      |                1.001  |                0.998  |                1.01   |               0.004  |               0      |                0     |                    0 |                0      |                 0     |
| NMAP_UDP_SCAN              |                0.0544 |                0.1093 |                0.0529 |                0.3247 |               0.1066 |               0.0008 |                0     |                    0 |                0      |                 0     |
| NMAP_XMAS_TREE_SCAN        |                0.999  |                0.0015 |                0.992  |                0.997  |               0.999  |               0.0005 |                0.998 |                    0 |                0      |                 0     |
| Thing_Speak                |                0.9224 |                0.9338 |                0.2523 |                7.0608 |               1.1073 |               1.2514 |                0     |                    0 |                0.0127 |                 0.008 |
| Wipro_bulb                 |                1.2134 |                1.3755 |                0.8458 |               39.7075 |              10.585  |              11.336  |                0     |                    0 |                0.083  |                 0.083 |


## 7. Respuestas a las preguntas

### ¿Los ataques tienen más SYN?

Como el archivo cargado no contiene clase `Normal`, no se puede afirmar si los ataques tienen más SYN que el tráfico normal. Sin embargo, dentro de los tipos de ataque, `flow_SYN_flag_count` aparece en **86.36%** de todos los registros y en **100%** de `DOS_SYN_Hping`. También aparece casi siempre en `MQTT_Publish`, `NMAP_TCP_scan` y `DDOS_Slowloris`. Esto indica que SYN es una de las variables más importantes del análisis.

### ¿Hay más RST en escaneos o conexiones fallidas?

Sí. `flow_RST_flag_count` aparece con alta presencia en ataques de escaneo o reconocimiento: `NMAP_OS_DETECTION` tiene 100% de presencia, `NMAP_TCP_scan` cerca de 99.80% y `NMAP_XMAS_TREE_SCAN` cerca de 99.20%. Esto sugiere que RST puede estar asociado a conexiones rechazadas, reiniciadas o respuestas anómalas durante escaneos.

### ¿Las variables URG, CWR o ECE aparecen mucho o casi nada?

Las variables URG aparecen muy poco o casi nada: `fwd_URG_flag_count` tiene 98.37% de ceros; `bwd_URG_flag_count` tiene 100.00% de ceros. Además, `flow_CWR_flag_count` y `flow_ECE_flag_count` tienen cerca de **99.94%** de ceros. Por tanto, URG, CWR y ECE tienen baja utilidad general, aunque podrían servir para casos muy específicos si aparecen concentradas en una clase particular.

### ¿Qué flags parecen más útiles para el modelo?

Los flags más útiles parecen ser `flow_SYN_flag_count`, `flow_ACK_flag_count` y `flow_RST_flag_count`, porque aparecen en una proporción alta del dataset y diferencian varios tipos de ataque. `flow_FIN_flag_count` tiene utilidad media o específica, porque aparece poco en general, pero es muy relevante para ataques como `NMAP_FIN_SCAN` y `NMAP_XMAS_TREE_SCAN`. Las variables PSH pueden aportar información adicional sobre transferencia activa de datos. URG, CWR y ECE parecen menos útiles por su baja presencia.

## 8. Flags que más aparecen por ataque

| variable            | ataque_mayor_presencia   |   presencia_max_% |   promedio_en_ese_ataque |   presencia_global_% |
|:--------------------|:-------------------------|------------------:|-------------------------:|---------------------:|
| flow_FIN_flag_count | NMAP_XMAS_TREE_SCAN      |             99.85 |                   0.999  |                 6.55 |
| flow_SYN_flag_count | DOS_SYN_Hping            |            100    |                   1      |                86.36 |
| flow_RST_flag_count | NMAP_OS_DETECTION        |            100    |                   1      |                78.43 |
| flow_ACK_flag_count | NMAP_OS_DETECTION        |            100    |                   1      |                81.69 |
| fwd_PSH_flag_count  | NMAP_XMAS_TREE_SCAN      |             99.85 |                   0.999  |                10.16 |
| bwd_PSH_flag_count  | MQTT_Publish             |             99.64 |                   3.7487 |                 8.09 |
| fwd_URG_flag_count  | NMAP_XMAS_TREE_SCAN      |             99.8  |                   0.998  |                 1.63 |
| bwd_URG_flag_count  | ARP_poisioning           |              0    |                   0      |                 0    |
| flow_CWR_flag_count | Wipro_bulb               |              8.3  |                   0.083  |                 0.06 |
| flow_ECE_flag_count | Wipro_bulb               |              8.3  |                   0.083  |                 0.06 |


## 9. Variables útiles

| variable            | utilidad           | justificación                                                                       |
|:--------------------|:-------------------|:------------------------------------------------------------------------------------|
| flow_FIN_flag_count | Media / específica | Aunque tiene muchos ceros, aparece mucho en NMAP_FIN_SCAN y NMAP_XMAS_TREE_SCAN.    |
| flow_SYN_flag_count | Alta               | Aparece en gran parte del dataset y diferencia varios tipos de ataque.              |
| flow_RST_flag_count | Alta               | Aparece en gran parte del dataset y diferencia varios tipos de ataque.              |
| flow_ACK_flag_count | Alta               | Aparece en gran parte del dataset y diferencia varios tipos de ataque.              |
| fwd_PSH_flag_count  | Media / específica | No aparece en todo el dataset, pero es fuerte en NMAP_XMAS_TREE_SCAN.               |
| bwd_PSH_flag_count  | Media / específica | No aparece en todo el dataset, pero es fuerte en MQTT_Publish.                      |
| fwd_URG_flag_count  | Media / específica | No aparece en todo el dataset, pero es fuerte en NMAP_XMAS_TREE_SCAN.               |
| bwd_URG_flag_count  | Baja / específica  | Casi siempre está en cero; útil solo si aparece asociado a ataques muy específicos. |
| flow_CWR_flag_count | Baja               | Casi siempre está en cero; solo aparece de forma muy puntual.                       |
| flow_ECE_flag_count | Baja               | Casi siempre está en cero; solo aparece de forma muy puntual.                       |


## 10. Problemas encontrados

- No se encontró clase `Normal` en el archivo cargado, por lo que no fue posible hacer una comparación real Normal vs Ataque.
- El dataset está desbalanceado: `DOS_SYN_Hping` concentra la mayoría de registros.
- Algunas clases tienen muy pocos registros, como `NMAP_FIN_SCAN` y `Metasploit_Brute_Force_SSH`, lo que puede afectar la estabilidad de los porcentajes.
- Algunas variables están casi siempre en cero, especialmente `flow_CWR_flag_count`, `flow_ECE_flag_count` y las variables URG.
- PSH y URG no aparecen como variables de flujo completo, sino separadas por dirección (`fwd` y `bwd`).

## 11. Conclusión

El análisis muestra que los flags TCP sí aportan información para caracterizar ataques en RT-IoT2022. Los más relevantes son `flow_SYN_flag_count`, `flow_ACK_flag_count` y `flow_RST_flag_count`, ya que tienen alta presencia y muestran patrones diferenciados en ataques como `DOS_SYN_Hping`, `NMAP_TCP_scan`, `NMAP_OS_DETECTION` y `NMAP_XMAS_TREE_SCAN`. `flow_FIN_flag_count` es menos frecuente, pero puede ser útil para ataques específicos como `NMAP_FIN_SCAN`. Las variables `flow_CWR_flag_count`, `flow_ECE_flag_count` y URG casi siempre son cero, por lo que tienen baja utilidad general para el modelo. Para un modelo de detección, conviene conservar SYN, ACK, RST y FIN como candidatas fuertes, revisar PSH como variable complementaria y tratar URG, CWR y ECE como variables de utilidad baja o específica.
