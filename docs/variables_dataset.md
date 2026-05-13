# Variables del dataset RT-IoT2022

Este documento presenta una revisión inicial de las variables del dataset RT-IoT2022. Para cada variable se indica su tipo de dato, un posible significado y su utilidad dentro del análisis de tráfico de red para detección de anomalías o ataques.

## Tabla de variables

| Variable Name | Data Type | Posible significado | Utilidad |
|---|---|---|---|
| `id.orig_p` | Integer | Puerto de origen de la conexión de red. Representa el puerto desde el cual se inicia la comunicación. | Puede ayudar a identificar patrones de tráfico, servicios usados por los dispositivos o comportamientos sospechosos relacionados con ciertos puertos. |
| `id.resp_p` | Integer | Puerto de destino o respuesta de la conexión. Representa el puerto del equipo que recibe la comunicación. | Ayuda a identificar servicios atacados o usados, por ejemplo puertos asociados a HTTP, SSH, DNS, etc. |
| `proto` | Categorical | Protocolo de red utilizado en el flujo, como TCP, UDP o ICMP. | Permite diferenciar tipos de tráfico y detectar comportamientos asociados a ciertos protocolos. |
| `service` | Continuous | Servicio de red asociado al flujo, como HTTP, DNS, SSH u otro. Aunque aparece como continua, por su significado puede tratarse como categórica. | Ayuda a identificar qué tipo de servicio se estaba usando y si está relacionado con actividad normal o maliciosa. |
| `flow_duration` | Continuous | Duración total del flujo de red. | Puede ayudar a detectar conexiones inusualmente largas, cortas o repetitivas. |
| `fwd_pkts_tot` | Integer | Total de paquetes enviados en dirección forward, es decir, desde el origen hacia el destino. | Sirve para medir la cantidad de tráfico generado por quien inicia la conexión. |
| `bwd_pkts_tot` | Integer | Total de paquetes enviados en dirección backward, es decir, desde el destino hacia el origen. | Permite analizar la respuesta del destino y comparar si hubo comunicación equilibrada o sospechosa. |
| `fwd_data_pkts_tot` | Integer | Total de paquetes con datos enviados desde el origen hacia el destino. | Ayuda a identificar cuánto tráfico útil se envió en la conexión. |
| `bwd_data_pkts_tot` | Integer | Total de paquetes con datos enviados desde el destino hacia el origen. | Sirve para revisar si el destino respondió con datos o si hubo intentos de conexión sin respuesta significativa. |
| `fwd_pkts_per_sec` | Continuous | Tasa de paquetes por segundo enviados desde el origen hacia el destino. | Es útil para detectar tráfico muy rápido o masivo, como ataques DoS o escaneos. |
| `bwd_pkts_per_sec` | Continuous | Tasa de paquetes por segundo enviados desde el destino hacia el origen. | Permite analizar qué tan rápido responde el destino y detectar respuestas anómalas. |
| `flow_pkts_per_sec` | Continuous | Cantidad total de paquetes por segundo dentro del flujo de red. | Sirve para medir la intensidad del tráfico. Valores muy altos pueden estar relacionados con ataques DoS, DDoS o escaneos rápidos. |
| `down_up_ratio` | Continuous | Relación entre el tráfico de descarga y subida dentro del flujo. | Ayuda a identificar si el tráfico está desbalanceado entre origen y destino, lo cual puede indicar comportamientos sospechosos. |
| `fwd_header_size_tot` | Integer | Tamaño total de los encabezados de paquetes enviados desde el origen hacia el destino. | Puede servir para analizar características técnicas del tráfico y detectar patrones inusuales en los paquetes enviados. |
| `fwd_header_size_min` | Integer | Tamaño mínimo de encabezado observado en los paquetes enviados desde el origen hacia el destino. | Ayuda a identificar variaciones en los paquetes y posibles diferencias entre tráfico normal y malicioso. |
| `fwd_header_size_max` | Integer | Tamaño máximo de encabezado observado en los paquetes enviados desde el origen hacia el destino. | Permite detectar paquetes con encabezados inusualmente grandes o comportamientos atípicos. |
| `bwd_header_size_tot` | Integer | Tamaño total de los encabezados de paquetes enviados desde el destino hacia el origen. | Sirve para analizar las respuestas del destino y comparar su comportamiento con el tráfico enviado por el origen. |
| `bwd_header_size_min` | Integer | Tamaño mínimo de encabezado observado en los paquetes enviados desde el destino hacia el origen. | Puede ayudar a detectar patrones anormales en las respuestas del destino. |
| `bwd_header_size_max` | Integer | Tamaño máximo de encabezado observado en los paquetes enviados desde el destino hacia el origen. | Útil para identificar variaciones o anomalías en los paquetes de respuesta. |
| `flow_FIN_flag_count` | Integer | Cantidad de banderas TCP FIN observadas en el flujo. La bandera FIN indica cierre de conexión. | Ayuda a identificar cómo terminan las conexiones. Valores anómalos pueden estar relacionados con escaneos, conexiones incompletas o tráfico sospechoso. |
| `flow_SYN_flag_count` | Integer | Cantidad de banderas TCP SYN observadas en el flujo. La bandera SYN se usa para iniciar una conexión TCP. | Es útil para detectar intentos de conexión masivos o repetitivos, como escaneos de puertos o ataques SYN flood. |
| `flow_RST_flag_count` | Integer | Cantidad de banderas TCP RST observadas en el flujo. La bandera RST indica reinicio o cierre abrupto de una conexión. | Puede ayudar a identificar conexiones rechazadas, interrumpidas o comportamientos sospechosos durante escaneos y ataques. |
| `fwd_PSH_flag_count` | Integer | Cantidad de banderas TCP PSH en dirección forward, es decir, desde el origen hacia el destino. PSH indica que los datos deben entregarse rápidamente a la aplicación. | Permite analizar el envío activo de datos desde el origen y detectar patrones inusuales de comunicación. |
| `bwd_PSH_flag_count` | Integer | Cantidad de banderas TCP PSH en dirección backward, es decir, desde el destino hacia el origen. | Ayuda a revisar si el destino respondió enviando datos de forma inmediata o con patrones anómalos. |
| `flow_ACK_flag_count` | Integer | Cantidad de banderas TCP ACK observadas en el flujo. ACK se usa para confirmar la recepción de paquetes. | Sirve para analizar el comportamiento de confirmación en una conexión y detectar tráfico irregular o incompleto. |
| `fwd_URG_flag_count` | Integer | Cantidad de banderas TCP URG en dirección forward. URG indica que un paquete contiene datos urgentes. | Puede ayudar a identificar paquetes poco comunes o comportamientos anómalos, ya que esta bandera no suele aparecer con frecuencia en tráfico normal. |
| `bwd_URG_flag_count` | Integer | Cantidad de banderas TCP URG en dirección backward. | Permite detectar respuestas con datos urgentes o comportamientos atípicos desde el destino hacia el origen. |
| `flow_CWR_flag_count` | Integer | Cantidad de banderas TCP CWR observadas en el flujo. CWR se relaciona con control de congestión en TCP. | Puede aportar información sobre condiciones de red o comportamientos técnicos inusuales durante la comunicación. |
| `flow_ECE_flag_count` | Integer | Cantidad de banderas TCP ECE observadas en el flujo. ECE se relaciona con notificaciones de congestión en TCP. | Ayuda a identificar señales de congestión o características específicas del tráfico que podrían diferenciar tráfico normal y malicioso. |
| `fwd_pkts_payload.min` | Continuous | Tamaño mínimo de la carga útil de los paquetes enviados desde el origen hacia el destino. | Sirve para analizar el tamaño mínimo de los datos enviados y detectar patrones de tráfico con paquetes muy pequeños o inusuales. |
| `fwd_pkts_payload.max` | Continuous | Tamaño máximo de la carga útil de los paquetes enviados desde el origen hacia el destino. | Permite detectar paquetes con cargas inusualmente grandes en la dirección de envío. |
| `fwd_pkts_payload.tot` | Continuous | Tamaño total de la carga útil enviada desde el origen hacia el destino. | Sirve para medir el volumen total de datos enviados por el origen. |
| `fwd_pkts_payload.avg` | Continuous | Promedio del tamaño de la carga útil de los paquetes enviados desde el origen hacia el destino. | Ayuda a identificar patrones de tamaño promedio en el tráfico enviado. |
| `fwd_pkts_payload.std` | Continuous | Desviación estándar del tamaño de la carga útil en paquetes enviados desde el origen hacia el destino. | Permite medir qué tan variable es el tamaño de los paquetes enviados. |
| `bwd_pkts_payload.min` | Continuous | Tamaño mínimo de la carga útil de los paquetes enviados desde el destino hacia el origen. | Ayuda a analizar el tamaño mínimo de las respuestas del destino. |
| `bwd_pkts_payload.max` | Continuous | Tamaño máximo de la carga útil de los paquetes enviados desde el destino hacia el origen. | Permite detectar respuestas con cargas inusualmente grandes. |
| `bwd_pkts_payload.tot` | Continuous | Tamaño total de la carga útil enviada desde el destino hacia el origen. | Sirve para medir el volumen total de datos respondidos por el destino. |
| `bwd_pkts_payload.avg` | Continuous | Promedio del tamaño de la carga útil en paquetes enviados desde el destino hacia el origen. | Ayuda a comparar el tamaño promedio de las respuestas frente al tráfico enviado. |
| `bwd_pkts_payload.std` | Continuous | Desviación estándar del tamaño de la carga útil en paquetes enviados desde el destino hacia el origen. | Permite identificar variaciones anormales en el tamaño de las respuestas. |
| `flow_pkts_payload.min` | Continuous | Tamaño mínimo de la carga útil considerando todos los paquetes del flujo. | Ayuda a caracterizar el comportamiento general del flujo de red. |
| `flow_pkts_payload.max` | Continuous | Tamaño máximo de la carga útil considerando todos los paquetes del flujo. | Puede detectar paquetes muy grandes o inusuales dentro del flujo. |
| `flow_pkts_payload.tot` | Continuous | Tamaño total de la carga útil de todos los paquetes del flujo. | Mide el volumen total de datos transmitidos en la conexión. |
| `flow_pkts_payload.avg` | Continuous | Promedio del tamaño de la carga útil en el flujo completo. | Ayuda a diferenciar flujos pequeños, normales o de alto volumen. |
| `flow_pkts_payload.std` | Continuous | Desviación estándar del tamaño de la carga útil en el flujo completo. | Permite detectar si los tamaños de paquetes son constantes o muy variables. |
| `fwd_iat.min` | Continuous | Tiempo mínimo entre paquetes consecutivos enviados desde el origen hacia el destino. | Ayuda a detectar tráfico muy rápido o automatizado. |
| `fwd_iat.max` | Continuous | Tiempo máximo entre paquetes consecutivos enviados desde el origen hacia el destino. | Puede indicar pausas largas o patrones irregulares en el envío. |
| `fwd_iat.tot` | Continuous | Tiempo total entre llegadas de paquetes en dirección origen a destino. | Sirve para analizar la duración y ritmo del tráfico enviado. |
| `fwd_iat.avg` | Continuous | Promedio del tiempo entre paquetes enviados desde el origen hacia el destino. | Ayuda a identificar si el origen envía paquetes de forma rápida, lenta o regular. |
| `fwd_iat.std` | Continuous | Desviación estándar del tiempo entre paquetes en dirección origen a destino. | Permite detectar variaciones anormales en el ritmo de envío. |
| `bwd_iat.min` | Continuous | Tiempo mínimo entre paquetes consecutivos enviados desde el destino hacia el origen. | Ayuda a analizar la rapidez de respuesta del destino. |
| `bwd_iat.max` | Continuous | Tiempo máximo entre paquetes consecutivos enviados desde el destino hacia el origen. | Puede mostrar pausas o respuestas irregulares del destino. |
| `bwd_iat.tot` | Continuous | Tiempo total entre llegadas de paquetes en dirección destino a origen. | Sirve para analizar el comportamiento temporal de las respuestas. |
| `bwd_iat.avg` | Continuous | Promedio del tiempo entre paquetes enviados desde el destino hacia el origen. | Ayuda a comparar el ritmo de respuesta frente al ritmo de envío. |
| `bwd_iat.std` | Continuous | Desviación estándar del tiempo entre paquetes en dirección destino a origen. | Permite detectar respuestas con tiempos muy variables o anómalos. |
| `flow_iat.min` | Continuous | Tiempo mínimo entre paquetes consecutivos dentro del flujo completo. | Útil para detectar ráfagas de paquetes o tráfico automatizado. |
| `flow_iat.max` | Continuous | Tiempo máximo entre paquetes consecutivos dentro del flujo completo. | Puede indicar interrupciones, pausas largas o comportamiento irregular. |
| `flow_iat.tot` | Continuous | Tiempo total entre llegadas de paquetes dentro del flujo. | Ayuda a caracterizar la duración temporal del flujo. |
| `flow_iat.avg` | Continuous | Promedio del tiempo entre paquetes del flujo completo. | Sirve para analizar el ritmo general de comunicación. |
| `flow_iat.std` | Continuous | Desviación estándar del tiempo entre paquetes del flujo completo. | Permite detectar tráfico constante o con variaciones sospechosas. |
| `payload_bytes_per_second` | Continuous | Cantidad de bytes de carga útil transmitidos por segundo. | Mide la velocidad de transferencia de datos y puede ayudar a detectar tráfico masivo o ataques. |
| `fwd_subflow_pkts` | Continuous | Cantidad de paquetes en subflujos enviados desde el origen hacia el destino. | Ayuda a analizar segmentos internos del flujo y patrones de envío. |
| `bwd_subflow_pkts` | Continuous | Cantidad de paquetes en subflujos enviados desde el destino hacia el origen. | Permite revisar cómo responde el destino dentro de partes del flujo. |
| `fwd_subflow_bytes` | Continuous | Cantidad de bytes en subflujos enviados desde el origen hacia el destino. | Sirve para medir el volumen de datos enviado en segmentos del flujo. |
| `bwd_subflow_bytes` | Continuous | Cantidad de bytes en subflujos enviados desde el destino hacia el origen. | Ayuda a medir el volumen de respuesta en segmentos del flujo. |
| `fwd_bulk_bytes` | Continuous | Cantidad de bytes enviados en bloque desde el origen hacia el destino. | Puede identificar transferencias grandes de datos o tráfico repetitivo. |
| `bwd_bulk_bytes` | Continuous | Cantidad de bytes enviados en bloque desde el destino hacia el origen. | Sirve para detectar respuestas grandes o transferencia masiva desde el destino. |
| `fwd_bulk_packets` | Continuous | Cantidad de paquetes enviados en bloque desde el origen hacia el destino. | Ayuda a identificar envíos agrupados o ráfagas de tráfico. |
| `bwd_bulk_packets` | Continuous | Cantidad de paquetes enviados en bloque desde el destino hacia el origen. | Permite detectar respuestas agrupadas o tráfico masivo desde el destino. |
| `fwd_bulk_rate` | Continuous | Tasa de envío de datos en bloque desde el origen hacia el destino. | Útil para identificar velocidades anómalas de transferencia en dirección forward. |
| `bwd_bulk_rate` | Continuous | Tasa de envío de datos en bloque desde el destino hacia el origen. | Ayuda a detectar respuestas de alta velocidad o comportamiento anormal en dirección backward. |
| `active.min` | Continuous | Tiempo mínimo durante el cual el flujo estuvo activo antes de entrar en inactividad. | Ayuda a analizar períodos cortos de actividad en la conexión. |
| `active.max` | Continuous | Tiempo máximo durante el cual el flujo estuvo activo. | Puede identificar conexiones con actividad prolongada o inusual. |
| `active.tot` | Continuous | Tiempo total en que el flujo estuvo activo. | Sirve para medir cuánto tiempo hubo transmisión efectiva de datos. |
| `active.avg` | Continuous | Promedio del tiempo activo del flujo. | Ayuda a comparar patrones de actividad entre tráfico normal y malicioso. |
| `active.std` | Continuous | Desviación estándar del tiempo activo del flujo. | Permite detectar variabilidad en los periodos de actividad. |
| `idle.min` | Continuous | Tiempo mínimo durante el cual el flujo estuvo inactivo. | Ayuda a identificar pausas cortas entre transmisiones. |
| `idle.max` | Continuous | Tiempo máximo durante el cual el flujo estuvo inactivo. | Puede mostrar conexiones con pausas largas o comportamiento irregular. |
| `idle.tot` | Continuous | Tiempo total de inactividad del flujo. | Sirve para analizar cuánto tiempo la conexión permaneció sin transmitir datos. |
| `idle.avg` | Continuous | Promedio del tiempo de inactividad del flujo. | Ayuda a diferenciar conexiones constantes de conexiones intermitentes. |
| `idle.std` | Continuous | Desviación estándar del tiempo de inactividad del flujo. | Permite detectar variaciones anormales en los periodos de pausa. |
| `fwd_init_window_size` | Integer | Tamaño inicial de la ventana TCP en dirección origen a destino. | Ayuda a analizar parámetros de comunicación TCP y posibles diferencias entre tráfico normal y ataques. |
| `bwd_init_window_size` | Integer | Tamaño inicial de la ventana TCP en dirección destino a origen. | Permite estudiar la capacidad inicial de recepción anunciada por el destino. |
| `fwd_last_window_size` | Integer | Último tamaño de ventana TCP observado en dirección origen a destino. | Puede ayudar a identificar cambios en el control de flujo durante la conexión. |
| `Attack_type` | Categorical | Tipo de tráfico registrado, por ejemplo tráfico normal o tipo específico de ataque. | Es la variable objetivo del dataset y se usa para entrenar modelos de clasificación. |
| `id` | Integer | Identificador único o índice del registro dentro del dataset. | Sirve para identificar registros, pero normalmente no se usa como variable predictora en el modelo. |


