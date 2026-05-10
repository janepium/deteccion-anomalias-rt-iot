# Conceptos clave: IoT, RT-IoT, tiempo real, tráfico de red y flujo de red

## IoT
IoT, o Internet de las Cosas, se refiere a una red de dispositivos físicos conectados a Internet que pueden recopilar, enviar y recibir datos. Estos dispositivos, como sensores, cámaras, electrodomésticos, vehículos o equipos industriales, integran capacidades de computación y comunicación para interactuar con otros dispositivos, aplicaciones o servicios en la nube.

Un sistema IoT generalmente está compuesto por:

- *Dispositivos inteligentes* que capturan datos del entorno
- Una *aplicación IoT* encargada de procesar y analizar la información
- Una *interfaz de usuario* como una aplicación móvil o página web, que permite monitorear y controlar los dispositivos

En muchos casos, los datos se procesan en tiempo real para tomar decisiones automáticas o asistir al usuario.

En el ámbito empresarial, IoT permite monitorear variables como consumo energético, estado de equipos, temperatura, humedad, tráfico de red o rendimiento de máquinas. Esto facilita la detección de patrones, tendencias y anomalías, ayudando a mejorar la eficiencia operativa, reducir costos, prevenir fallos y tomar decisiones basadas en datos.

Las tecnologías que hacen posible IoT incluyen:

- sensores y actuadores que detectan y provocan cambios físicos en el entorno respectivamente
- redes de conectividad como Wi-Fi, Bluetooth, redes celulares, Zigbee o LoRaWAN
- computación en la nube y computación en el borde
- aprendizaje automático (machine learning)
- mecanismos de seguridad como cifrado, control de acceso y detección de intrusiones

## RT-IoT

RT-IoT significa Real-Time Internet of Things o Internet de las Cosas en Tiempo Real. Se refiere a sistemas IoT donde los dispositivos no solo recopilan y envían datos, sino que además deben procesarlos y responder en un tiempo muy corto, casi inmediato. Se usa en sistemas críticos como vehículos, redes eléctricas, manufactura, control industrial y dispositivos médicos; donde los retrasos o fallas pueden afectar la seguridad del sistema, el entorno o a las personas.

Por ejemplo, en un sistema IoT común, un sensor puede enviar datos de temperatura cada cierto tiempo para ser analizados después. En cambio, en RT-IoT, si un sensor detecta una temperatura peligrosa en una máquina industrial, el sistema debe reaccionar de inmediato, por ejemplo apagando el equipo, activando una alarma o ajustando automáticamente el funcionamiento.

Para detección de anomalías en dispositivos IoT, el concepto de RT-IoT es relevante ya que permite identificar los comportamientos extraños o ataques en el momento en que ocurren, no después. Esto ayuda a responder rápidamente ante fallos, congestión en la red, tráfico sospechoso o posibles amenazas de seguridad.

## Tiempo real

Se refiere a la capacidad de un sistema para procesar información y producir resultados dentro de un plazo específico y predecible. En un sistema de tiempo real , la predictibilidad es tan importante como la velocidad, ya que una respuesta tardía puede afectar el funcionamiento, la seguridad o la calidad del servicio.

En el contexto de IoT y detección de anomalías, el tiempo real permite analizar datos de sensores o tráfico de red de forma oportuna para identificar fallos, ataques o comportamientos inusuales antes de que afecten el funcionamiento del sistema.

## Tráfico de red

Es el conjunto de datos que circulan entre dispositivos conectados a una red. Estos datos viajan en forma de paquetes y permiten la comunicación entre equipos, aplicaciones, servidores, sensores y otros dispositivos. En un entorno IoT, el tráfico de red incluye la información enviada y recibida por sensores, actuadores, gateways, servicios en la nube y aplicaciones de monitoreo. Su análisis permite recopilar, procesar y estudiar información como direcciones IP, puertos, protocolos, paquetes y flujos de comunicación. 

Para esto se plantean diferentes tipos de análisis:

- **Análisis de comportamiento:** que compara patrones actuales con modelos de referencia para detectar actividad sospechosa
- **Análisis estadístico:** que revisa volumen, tamaño de paquetes y distribución de protocolos para identificar anomalías
- **Análisis de flujo:** que examina la comunicación entre dispositivos para encontrar problemas de rendimiento o posibles amenazas.

Estos análisis nos ayudan a mantener la seguridad y el rendimiento de la red, detectar comportamientos anómalos, identificar amenazas y solucionar problemas antes de que afecten el sistema. Por esto, en proyectos de detección de anomalías IoT, el tráfico de red es una fuente clave para reconocer patrones normales y detectar actividades sospechosas.

## Flujo de red
Es una secuencia de paquetes que comparten características comunes dentro de una comunicación en red, como dirección IP de origen y de destino, puertos de origen y destino, protocolo utilizado y, en algunos casos, marcas de tiempo, número de paquetes o cantidad de bytes transmitidos. En otras palabras, es el resumen de una comunicación entre dispositivos dentro de una red.

En el contexto de detección de anomalías en dispositivos IoT, el flujo de red es importante porque permite analizar cómo se comunican los dispositivos sin revisar paquete por paquete. Por ejemplo, se pueden identificar patrones como un sensor enviando más datos de lo normal, conexiones desde orígenes desconocidos, uso inusual de puertos o aumento repentino en la cantidad de paquetes. Estos cambios pueden indicar fallos, comportamientos anómalos o posibles ataques.

## Fuentes
- https://aws.amazon.com/es/what-is/iot/
- https://www.ibm.com/think/topics/internet-of-things
- https://www.mdpi.com/1424-8220/18/12/4356
- https://www.intel.com/content/www/us/en/learn/what-is-a-real-time-system.html
- https://es.wikipedia.org/wiki/Tr%C3%A1fico_de_red
- https://www.ibm.com/mx-es/think/topics/network-traffic-analysis
- https://es.wikipedia.org/wiki/Flujo_de_tr%C3%A1fico_(redes_inform%C3%A1ticas)
