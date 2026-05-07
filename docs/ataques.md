# Ataques IoT — Referencia del dataset RT-IoT

> **Convención de variables:** `fwd_` = dirección origen→destino · `bwd_` = destino→origen · `flow_` = flujo completo  
> Cada entrada describe qué es el ataque, por qué es relevante en IoT y qué señal deja en las variables del dataset.

---

## Índice

| Categoría | Ataques |
|---|---|
| [DoS / DDoS](#1-dos--ddos) | `DOS_SYN_Hping`, `DDOS_Slowloris` |
| [Reconocimiento NMAP](#2-reconocimiento-nmap) | `NMAP_TCP_scan`, `NMAP_UDP_SCAN`, `NMAP_FIN_SCAN`, `NMAP_XMAS_TREE_SCAN`, `NMAP_OS_DETECTION` |
| [Red](#3-red) | `ARP_poisioning` |
| [Protocolos IoT](#4-protocolos-iot) | `Thing_Speak`, `MQTT_Publish`, `Wipro_bulb` |
| [Explotación](#5-explotación) | `Metasploit_Brute_Force_SSH` |
| [Referencia cruzada](#6-referencia-cruzada-variables-→-ataques) | Variables → ataques |

---

## 1. DoS / DDoS

**Objetivo:** agotar recursos del dispositivo víctima (memoria, conexiones, CPU). Los dispositivos IoT son especialmente vulnerables por sus recursos limitados.

---

### `DOS_SYN_Hping`
*94,659 muestras*

Inundación de paquetes SYN usando la herramienta Hping. TCP requiere tres pasos para abrir una conexión (SYN → SYN-ACK → ACK). El atacante envía SYN masivos con IPs falsas y nunca completa el tercer paso — el dispositivo acumula conexiones a medias hasta colapsar.

| Variable | Valor esperado | Razón |
|---|---|---|
| `flow_SYN_flag_count` | muy alto | un SYN por cada paquete de ataque |
| `flow_ACK_flag_count` | ≈ 0 | el handshake nunca se completa |
| `flow_RST_flag_count` | alto | el servidor intenta cerrar conexiones huérfanas |
| `fwd_pkts_per_sec` | muy alto | bombardeo sostenido |
| `down_up_ratio` | muy bajo | tráfico casi exclusivamente unidireccional |
| `fwd_pkts_payload.avg` | ≈ 0 | los SYN no llevan datos |
| `bwd_pkts_tot` | muy bajo | el servidor apenas puede responder |
| `flow_duration` | corto | flujos que se abren pero no progresan |

---

### `DDOS_Slowloris`
*534 muestras*

Agotamiento lento de conexiones HTTP. Abre muchas conexiones legítimas y las mantiene enviando headers incompletos a cuentagotas — el servidor espera indefinidamente sin liberar recursos. Especialmente efectivo contra interfaces web de configuración IoT (routers, cámaras).

| Variable | Valor esperado | Razón |
|---|---|---|
| `flow_duration` | muy largo | conexión abierta deliberadamente |
| `fwd_pkts_per_sec` | muy bajo | envío intencionalmente lento |
| `fwd_iat.avg`, `fwd_iat.std` | altos | grandes pausas entre paquetes |
| `idle.avg`, `idle.max` | muy altos | el flujo pasa la mayor parte del tiempo inactivo |
| `payload_bytes_per_second` | muy bajo | pocos bytes transferidos por segundo |
| `flow_SYN_flag_count` | 1 por flujo | a diferencia del SYN Flood, el handshake sí se completa |
| `flow_ACK_flag_count` | presente | conexión TCP establecida normalmente |

---

## 2. Reconocimiento NMAP

**Objetivo:** mapear la red antes de atacar — descubrir dispositivos activos, puertos abiertos y sistema operativo. No causa daño directo.

**Señal común a todos los escaneos NMAP:** `flow_duration` muy corto · `fwd_pkts_payload.tot` ≈ 0 · `fwd_pkts_tot` mínimo (2–4 paquetes) · `id.resp_p` varía sistemáticamente entre flujos.

---

### `NMAP_TCP_scan`
*1,002 muestras*

Completa el handshake TCP con cada puerto objetivo y cierra inmediatamente con RST. El más confiable pero también el más detectable.

| Variable | Valor esperado |
|---|---|
| `flow_SYN_flag_count` | alto |
| `flow_ACK_flag_count` | alto |
| `flow_RST_flag_count` | alto — cierre inmediato post-verificación |
| `fwd_pkts_tot` | 2–3 paquetes |

---

### `NMAP_UDP_SCAN`
*2,590 muestras*

Sondea puertos UDP enviando un paquete vacío. Sin respuesta = puerto posiblemente abierto; ICMP "inalcanzable" = cerrado. Descubre servicios invisibles al TCP (DNS, SNMP, DHCP — comunes en IoT).

| Variable | Valor esperado |
|---|---|
| `proto` | UDP |
| `flow_SYN_flag_count`, `flow_ACK_flag_count` | 0 — UDP no tiene flags TCP |
| `bwd_pkts_tot` | ≈ 0 — puertos abiertos no responden |
| `fwd_pkts_tot` | 1–2 paquetes |

---

### `NMAP_FIN_SCAN`
*28 muestras*

Envía un flag FIN (cierre) a puertos que nunca se abrieron. Puerto cerrado responde RST; puerto abierto ignora el paquete. Evade firewalls que solo filtran SYN.

| Variable | Valor esperado |
|---|---|
| `flow_FIN_flag_count` | alto — firma del ataque |
| `flow_SYN_flag_count`, `flow_ACK_flag_count` | 0 |
| `flow_RST_flag_count` | alto (respuesta de puertos cerrados) |
| `fwd_pkts_payload.tot` | 0 |

---

### `NMAP_XMAS_TREE_SCAN`
*2,010 muestras*

Activa FIN + URG + PSH simultáneamente — combinación imposible en tráfico legítimo. La respuesta del dispositivo revela estado de puertos y sistema operativo. Más sigiloso que FIN scan.

| Variable | Valor esperado |
|---|---|
| `flow_FIN_flag_count` | alto |
| `fwd_URG_flag_count` | alto — diferenciador clave vs. FIN scan |
| `fwd_PSH_flag_count` | alto |
| `flow_SYN_flag_count`, `flow_ACK_flag_count` | 0 |
| `fwd_pkts_payload.tot` | 0 |

---

### `NMAP_OS_DETECTION`
*2,000 muestras*

No escanea puertos — analiza cómo responde el dispositivo a paquetes TCP/UDP/ICMP diseñados para revelar el sistema operativo (tamaño de ventana, TTL, comportamiento de flags). Con esa huella digital el atacante elige exploits específicos.

| Variable | Valor esperado |
|---|---|
| `fwd_init_window_size` | valores inusuales o variados — NMAP experimenta |
| `bwd_init_window_size` | valor característico del SO víctima |
| `fwd_header_size_tot` | variable — NMAP modifica headers |
| `proto` | mezcla TCP/UDP/ICMP |
| `flow_duration` | muy corto por flujo, muchos flujos en ráfaga |

---

## 3. Red

### `ARP_poisioning`
*7,750 muestras*

ARP traduce IPs a direcciones MAC en la red local sin autenticación. El atacante envía respuestas ARP falsas de forma continua para redirigir el tráfico a través de su máquina (Man-in-the-Middle). En redes IoT permite interceptar datos de sensores, inyectar comandos a actuadores o desconectar dispositivos sin que se note.

| Variable | Valor esperado | Razón |
|---|---|---|
| `proto` | ARP | protocolo de capa 2 |
| `flow_SYN_flag_count` | 0 | no hay flags TCP en capa 2 |
| `fwd_pkts_per_sec` | alto y sostenido | requiere envío continuo para mantener el envenenamiento |
| `fwd_pkts_payload.avg` | muy bajo y constante | paquetes ARP tienen tamaño fijo |
| `fwd_pkts_payload.std` | muy bajo | todos los paquetes son casi idénticos |
| `bwd_pkts_tot` | bajo | el ataque es mayormente unidireccional |

---

## 4. Protocolos IoT

**Contexto:** este grupo captura tráfico de plataformas y protocolos propios del ecosistema IoT. Puede representar uso legítimo con comportamiento anómalo, exfiltración disfrazada de tráfico normal, o inyección de comandos a través del canal habitual del dispositivo.

---

### `Thing_Speak`
*8,108 muestras*

ThingSpeak (MathWorks) es una plataforma cloud IoT donde los dispositivos publican datos de sensores vía HTTP/HTTPS. Tráfico anómalo puede indicar exfiltración de datos o comunicación C2 camuflada en un canal de apariencia legítima.

| Variable | Valor esperado |
|---|---|
| `service` | HTTP / HTTPS |
| `id.resp_p` | 80 o 443 |
| `fwd_iat.std` | bajo si es periódico legítimo · alto si es anómalo |
| `down_up_ratio` | bajo — el dispositivo envía más de lo que recibe |
| `fwd_pkts_payload.avg` | pequeño y constante |

---

### `MQTT_Publish`
*4,146 muestras*

MQTT es el protocolo estándar de mensajería IoT (publish/subscribe). Los dispositivos publican en "tópicos" a través de un broker central. Sin autenticación o sin TLS (puertos 1883/8883), un atacante puede inyectar comandos falsos a actuadores o leer todos los mensajes de la red.

| Variable | Valor esperado |
|---|---|
| `service` | MQTT |
| `id.resp_p` | 1883 (sin cifrar) · 8883 (TLS) |
| `proto` | TCP |
| `fwd_pkts_payload.avg` | pequeño — mensajes MQTT son compactos |
| `fwd_pkts_per_sec` | bajo y regular |
| `flow_ACK_flag_count` | alto — MQTT usa ACKs de aplicación además de TCP |

---

### `Wipro_bulb`
*253 muestras*

Tráfico de bombillas inteligentes Wipro conectadas a Wi-Fi. Vectores de riesgo documentados: extracción de credenciales almacenadas en el dispositivo, uso como punto de entrada a la red, incorporación a botnets IoT.

| Variable | Valor esperado |
|---|---|
| `fwd_pkts_payload.avg` | muy pequeño y constante |
| `fwd_pkts_per_sec` | bajo — solo activo al recibir comandos |
| `flow_duration` | corto por interacción |
| `id.resp_p` | puerto propietario del fabricante |

---

## 5. Explotación

### `Metasploit_Brute_Force_SSH`
*37 muestras*

SSH permite acceso remoto completo a la línea de comandos del dispositivo. Metasploit automatiza miles de intentos de usuario/contraseña. En IoT el vector es predecible: la mayoría de dispositivos mantienen credenciales de fábrica (admin/admin, root/root). Un acceso exitoso implica control total del dispositivo, posibilidad de instalar malware y usarlo como pivote en la red.

| Variable | Valor esperado | Razón |
|---|---|---|
| `id.resp_p` | 22 | puerto estándar SSH |
| `service` | SSH | |
| `flow_SYN_flag_count` | muy alto | un handshake por cada intento de credencial |
| `flow_RST_flag_count` | alto | el servidor rechaza intentos fallidos |
| `fwd_pkts_per_sec` | alto y sostenido | bruteforce automatizado y rápido |
| `flow_duration` | corto por flujo | cada intento fallido cierra la conexión |
| `down_up_ratio` | muy bajo | muchos intentos enviados, pocos aceptados |
| `fwd_subflow_pkts` | alto | muchos subflujos en la misma sesión de ataque |

---

## 6. Referencia cruzada: variables → ataques

Guía rápida para análisis exploratorio y feature engineering.

| Variable(s) | Ataques donde es discriminante |
|---|---|
| `flow_SYN_flag_count` alto + `flow_ACK_flag_count` ≈ 0 | `DOS_SYN_Hping` |
| `flow_duration` muy largo + `idle.*` altos | `DDOS_Slowloris` |
| `flow_FIN_flag_count` alto + SYN/ACK = 0 | `NMAP_FIN_SCAN` |
| `fwd_URG_flag_count` + `fwd_PSH_flag_count` altos | `NMAP_XMAS_TREE_SCAN` |
| `proto` = UDP + todos los flags = 0 | `NMAP_UDP_SCAN` |
| `fwd_init_window_size` variable + `proto` mixto | `NMAP_OS_DETECTION` |
| `proto` = ARP + `fwd_pkts_payload.std` ≈ 0 | `ARP_poisioning` |
| `id.resp_p` = 22 + `flow_SYN_flag_count` muy alto | `Metasploit_Brute_Force_SSH` |
| `id.resp_p` = 1883 o 8883 + `proto` = TCP | `MQTT_Publish` |
| `id.resp_p` = 80/443 + `fwd_iat.std` bajo | `Thing_Speak` (legítimo) |
| `fwd_pkts_payload.tot` ≈ 0 + flujo muy corto | Todos los escaneos NMAP |
| `down_up_ratio` muy bajo + `fwd_pkts_per_sec` alto | `DOS_SYN_Hping`, `Metasploit_Brute_Force_SSH` |
