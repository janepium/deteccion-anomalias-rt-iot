# Patrones de tráfico — RT-IoT2022

---

## Tráfico normal

Estos tres patrones representan dispositivos IoT funcionando con normalidad. Son la línea base del comportamiento esperado en la red.

---

### `MQTT_Publish`

Un dispositivo IoT enviando datos a un servidor central usando el protocolo de mensajería estándar de IoT (MQTT). Es el equivalente a un operario enviando un reporte de turno: el dispositivo publica su información y el servidor la recibe.

---

### `Thing_Speak`

Un dispositivo IoT subiendo datos a la plataforma cloud ThingSpeak para almacenarlos y visualizarlos. Similar a un sensor que registra sus lecturas en una base de datos remota.

---

### `Wipro_bulb`

Bombillas inteligentes Wipro comunicándose con su aplicación de control. Tráfico muy pequeño y esporádico: el dispositivo solo transmite cuando recibe un comando o reporta su estado.

---

## Patrones de ataque

### Agotamiento de recursos

Ataques que buscan colapsar el dispositivo víctima saturando su capacidad de responder conexiones.

---

#### `DOS_SYN_Hping`

El atacante bombardea el dispositivo con miles de solicitudes de conexión falsas por segundo. El dispositivo intenta responder a cada una y se queda sin recursos para atender conexiones legítimas. Es como si alguien llamara al teléfono de soporte miles de veces por minuto sin intención de hablar, bloqueando la línea para los clientes reales.

---

#### `DDOS_Slowloris`

El atacante abre muchas conexiones con el dispositivo y las mantiene deliberadamente vivas enviando información a cuentagotas. El dispositivo espera que cada conexión termine y nunca lo hace, agotando su capacidad de atender nuevas conexiones. A diferencia del anterior, no es rápido ni masivo — es lento y silencioso.

---

### Reconocimiento

Ataques que no causan daño directo. Su objetivo es mapear la red para identificar dispositivos activos, puertos abiertos y sistemas operativos antes de lanzar un ataque real.

---

#### `NMAP_TCP_scan`

Escaneo que verifica qué puertas de entrada (puertos) tiene abiertas el dispositivo, completando y cerrando cada conexión inmediatamente. Es el más detectable pero también el más confiable.

---

#### `NMAP_UDP_SCAN`

Igual que el anterior pero probando un tipo diferente de puertos (UDP), usados por servicios como DNS y SNMP, comunes en IoT.

---

#### `NMAP_FIN_SCAN`

Escaneo más sigiloso que envía señales de cierre de conexión a puertos que nunca se abrieron. Está diseñado para evadir ciertos sistemas de seguridad que solo bloquean intentos de apertura.

---

#### `NMAP_XMAS_TREE_SCAN`

Escaneo que usa una combinación de señales que no existe en tráfico legítimo. La respuesta del dispositivo revela información sobre sus puertos y sistema. Es más difícil de detectar que un escaneo normal.

---

#### `NMAP_OS_DETECTION`

No busca puertos abiertos sino identificar qué sistema operativo corre el dispositivo. Con esa información el atacante puede elegir el exploit más efectivo para ese sistema específico.

---

### Interceptación

---

#### `ARP_poisioning`

En una red local, los dispositivos usan un directorio interno para saber a qué dirección física enviar cada mensaje. El atacante falsifica ese directorio para que todo el tráfico pase por su máquina antes de llegar al destino. Es el equivalente digital a redirigir el correo de alguien a través de tu propia dirección — puedes leerlo, modificarlo o bloquearlo sin que nadie lo note.

---

### Explotación

---

#### `Metasploit_Brute_Force_SSH`

El atacante prueba automáticamente miles de combinaciones de usuario y contraseña para acceder al dispositivo por SSH — el canal que permite controlarlo remotamente desde cualquier lugar. En IoT esto es especialmente efectivo porque la mayoría de dispositivos mantienen las credenciales que vienen de fábrica (admin/admin, root/root). Si lo logra, tiene control total del dispositivo.
