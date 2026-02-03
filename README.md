# UD6-Servicios-de-Streamingg

# Servicios de streaming

## Qué es?

### Descarga directa vs Streaming

- Descarga Directa: el usuario demanda un fichero con un peso de
100MB y 10 minutos de duración. Comienza la descarga. Se almacena
en buffer y comienza la reproducción. El usuario termina la
reproducción a los 2 minutos. El servidor ha entregado las 100MB.


- Streaming: Datos enviados en flujo constante. No hay almacenamiento
local permanente. Solo se consume el ancho de banda que el cliente ha
utilizado (2 minutos según el ejemplo anterior). 



## Topología de red

### Unicast: conexión 1 a 1 (estándar de internet)

- Mecánica: Si hay 100 oyentes, el servidor abre 100 sockets TCP y envía
la información 100 veces.

- Cálculo de Ancho de Banda: BW(tot) = BW(stream) x N(usuarios)

- Desventaja: Poco escalable.

## Multicast:

- Mecánica: El servidor envía la información a una dirección multicast
(224.0.0.0 - 239.255.255.255). Routers replican el paquete solo si
tienen suscriptores.

- Desventaja: Routers bloquean paquetes multicast. Solo viable en redes
internas. 

## QoS: Jitter, Buffer

### Jitter (Fluctuación)

Es la variación en el tiempo de llegada de los paquetes.

- Ejemplo: El paquete 1 tarda 20ms, el paquete 2 tarda 150ms, el paquete 3 tarda
20ms.

- Si el Jitter es superior al tamaño del buffer, el audio se corta (Buffer Underrun).

### Buffer (Amortiguador)

Es una memoria temporal en el cliente (y en el servidor).

- Función: Acumular suficientes segundos de audio para absorber el Jitter de la red.

- Efecto: A mayor buffer → Mayor estabilidad → Mayor latencia (retraso).


### Burst-on-Connect (Ráfaga de conexión)

Una característica específica de servidores como Icecast.

- Problema: Al conectarse, el oyente tardaría varios segundos en llenar
su buffer a velocidad normal (1x).

- Solución (Burst): El servidor envía los datos iniciales (ej. 64KB) a la
máxima velocidad posible que permita la red (ej. 10x), llenando el
buffer del cliente casi instantáneamente para que el audio empiece a
sonar de inmediato (Time-to-first-byte reducido).


## Protocolos de Streaming

### 1. Capa de transporte: TCP vs UDP
En TCP si un paquete de audio/vídeo se pierde, el cliente no lo
reproduce y por tanto el servidor lo reenvía (ACK/NACK).
La ventaja es evidente: calidad y pasa sin problemas por firewalls, NAT
y proxy al usar puertos estándar.
La desventaja también es evidente: alta latencia. La retransmisión de
paquetes introduce retraso.
En UDP sacrificamos calidad pero latencia mínima. 

TODO AUDIO/VIDEO NO ES UDP


### 2. Capa de aplicación (tres modelos):

1. HTTP Legacy (como usa Icecast2 - lo veremos más adelante)
2. HTTP Adaptativo
3. Real-time

## Protocolos de Streaming

### HTTP Legacy (como usa Icecast2 - lo veremos más adelante)

● Protocolo: ICY

● Mecánica: se abre conexión TCP y el servidor envía flujo de datos sin parar hasta
que el cliente cierra la conexión.

● Puertos: 80. 443, 8000 (Icecast2)

● Formato: flujo continuo de bytes (MP3, Ogg, ACC).

### HTTP Adapatitivo

● Protocolos: HLS (HTTP Live Streaming de Apple) - MPEG-DASH.

● Mecánica: no es flujo continuo. El servidor trocea el fichero en pequeños trozos
(chunks) de 2 a 10 segundos.

● Formato: .ts, .m4s.

● Pro: calidad adaptativa. El servidor envía un Manifest con el que da la opción a
descargar un chunk de mejor o peor calidad. 

### Real-Time

● RTMP (Real-Time Messaging Protocol): funciona sobre TCP. Está
obsoleto para usuario final pero se usa para enviar vídeo al servidor
(por ejemplo de OBS a YT/Twitch).

● RTSP (Real-Time Streaming Protocol): usado en cámaras de seguridad
(CCTV) y sistemas domóticos. Generalmente usa UDP para datos y
TCP para control.

● WebRTC: para videoconferencia. P2P, cifrado, UDP, funciona en
navegador sin plugins (Google Meet, Discord). 


## Protocolos de Streaming

Como hemos visto, la industria utiliza mucho más TCP que UDP. Netflix, HBO, Disney+, etc utilizan HTTP adaptativo. Al ver una película estás descargando pequeños chunks, trozos del vídeo, secuencialmente vía TCP.
Spotify y Apple Music también usa TCP. 

¿Has escuchado una canción alguna vez en Spotify en el que pierdas un fragmento? Quizás se pare pero no escucharás algo raro como en una videollamada.
Twitch (del lado del receptor): usa TCP. Por eso hay un delay.

La radio online también es TCP (y vamos a ponerlo en práctica con Icecast2).
Cuando se necesita interactuar con la otra parte el delay no es admisible y por
tanto se utiliza UDP. 

## Icecast 2

Icecast2 es un software de servidor de streaming de medios de código
abierto. En términos sencillos, actúa como una "antena de radio virtual"
en internet: recibe el audio de una fuente (un locutor o una lista de
reproducción) y lo distribuye a miles de oyentes simultáneamente.
Icecast2 NO genera el contenido, solo lo distribuye. Por tanto, necesita
de un cliente que le entregue contenidos.

- Formatos: OGG / MP3

- Gestión de oyentes

- Puntos de montaje (ej. /radio-asir y /radio-smr)

apt update
apt install icecast2
Configura contraseñas (usa la misma para todas las opciones)
Comprueba que el puerto 8000 esté abierto

## Mixxx
add-apt-repository ppa:mixxx/mixxx
apt update
apt install mixxx
Configuración de la emisión en vivo:

## Códecs

Son algoritmos que permiten la compresión de los ficheros de
audio/vídeo. También para la descompresión.
¿Para qué? Para reducir el trasiego de información sin perder
calidad.
Por ejemplo: una canción de 3 minutos en calidad CD sin
compresión podría pesar unos 32MB. La misma canción en MP3
(comprimida) pesaría unos 3MB.
Ejemplos de códecs de audio: MP3, AAC, Vorbis, WAV.
Ejemplos de códecs de vídeo: H.264, H.265, AV1.


## Frecuencia de muestreo: 

el audio es una onda analógica. Para
digitalizarla hay que muestrearla, algo así como hacerle fotos cada X
tiempo.
Estándar: 44,1kHz.


## Profundidad de bits:

si la frecuencia de muestreo eran las “fotos” que hacíamos a la onda, la profundidad es la calidad de dicha foto.
Se trata de la cantidad de bits que se transmiten por segundo: a mayor cantidad, más calidad. 
Estándar: 16 bit (calidad CD)

## Canales: 
Número de audios independientes que viajan en el mismo
stream.

## Códecs con pérdida / sin pérdida

### - Códecs con pérdida: 

Reducen peso sacrificando información que puede ser imperceptible por el oído humano.
Al comprimirlo dicha información se pierde y es irrecuperable.
Ejemplo: MP3.

### - Códecs sin pérdida:

Comprime el fichero como lo haría un .zip pero sin eliminar información. Al descomprimir el flujo de bits es idéntico al original.
El factor de compresión es menor a los códecs con pérdida.
Ejemplo: FLAC, WAV. 

### Cálculo de peso

Sin compresión WAV: 3 minutos de canción en estéreo con frecuencia de muestreo de 44,1kHz y profundidad de 16 bits. 
P = 44100 x 16 x 2 x 3 x 60 = 254016000/8 =31752000 (aprox 31,75MB). 


# EJERCICIOS;
Cálculo de peso
###  1. Calcula el peso aproximado de un archivo de audio sin compresión (WAV) de 5 minutos, con una frecuencia de muestreo de 44,1 kHz, 16 bits y en estéreo.

Datos:
Frecuencia de muestreo: 44.100 Hz
Profundidad: 16 bits
Canales: 2 (estéreo)
Duración: 5 minutos
Formato: WAV (sin compresión)

*Fórmula Peso (bits)* = frecuencia × bits × canales × segundos
*Peso (bits)* = frecuencia×bits×canales×segundos

**Cálculo**:
Pasamos minutos a segundos:

5 × 60 = 300s

**Calculamos el total en bits:**

44.100 × 16 × 2 × 300 = 423.360.000 bits

*Pasamos de bits a bytes (÷8):*

423.360.000/8 = 52.920.000  bytes

Pasamos a MB (aprox):

≈52,9MB

### 2. Si emitimos un streaming en MP3 a un bitrate constante (CBR) de 128 kbps, ¿cuánto ancho de banda total consumirá el servidor si tiene 25 oyentes simultáneos?

*Datos*:
Bitrate del stream: 128 kbps
Oyentes: 25
Tipo de red: Unicast

**Fórmula**
BWtotal = BWstream x Nusuarios

*Cálculo:*

128 kbps × 25 =3.200 kbps

Pasamos a Mbps:

3.200/1.000 = 3,2 Mbps

*Resultado*
El servidor consume 3,2 Mbps de subida


### 3. Calcula el bitrate de un flujo de audio que utiliza una frecuencia de 48 kHz, 24 bits de profundidad y un solo canal (mono).

*Datos* 
Frecuencia de muestreo: 48.000 Hz
Profundidad: 24 bits
Canales: 1 (mono)

Fórmula del bitrate**

Bitrate = frecuencia × bits × canales

*Cálculo*

48.000 × 24 × 1 = 1.152.000 bps
48.000×24×1=1.152.000 bps

Pasamos a kbps:

1.152.000/1.000 = 1.152 kbps

*Resultado*
Bitrate = 1.152 kbps (≈ 1,15 Mbps)

### 4. Tienes un servidor con un límite de subida de 10 Mbps. ¿Cuántos oyentes a 192 kbps puede soportar teóricamente antes de saturar la red?

*Datos*
- Ancho de banda total: **10 Mbps = 10.000 kbps**
- Bitrate por oyente: **192 kbps**
- Topología: **Unicast**

**Fórmula**
N = BW_total / BW_oyente

*Cálculo*
10.000 / 192 = 52,08

Solo se pueden tener oyentes completos (no fracciones).

*Resultado*
Máximo teórico: 52 oyentes



# Vídeo – Cálculo de peso y streaming

## Cálculo de peso (vídeo sin comprimir)

**Fórmula general:**

Peso = (Ancho × Alto) × Profundidad de color × FPS × Tiempo

### Resoluciones habituales
- **1080p** → 1920 × 1080  
- **4K** → 4096 × 2160  
- **8K** → 7680 × 4320  

### Profundidad de color
Bits usados para definir el color de cada píxel.  
Valor habitual: **24 bits (8 + 8 + 8)**

### FPS
Frames per second (fotogramas por segundo).

---

## Cálculo de peso (vídeo comprimido)

Al utilizar un **códec**, el vídeo ya no se envía píxel a píxel.  
El códec decide qué información mantiene y cuál elimina.

En ficheros comprimidos, **el dato importante es el bitrate**.

**Fórmula:**

Peso = Bitrate × Tiempo

El **bitrate** es la cantidad de información enviada por segundo.

---

## Vídeo: conceptos generales

La lógica y los conceptos del vídeo son muy similares a los del audio.  
También existen **protocolos** y **códecs**, pero se introduce el concepto de **contenedor**.

### Contenedor
Formato de fichero que puede incluir:
- Pistas de vídeo
- Pistas de audio
- Subtítulos
- Metadatos

**Ejemplos:* MP4, MKV, MOV, OGG

---

# EJERCICIOS RESUELTOS

### Ejercicio 1: La pesadilla del almacenamiento Un estudio de cine graba en RAW (sin comprimir) con una cámara 4K (3840x2160 píxeles), a 60 fps y una profundidad de color de 30 bits (HDR). 
### ● A. Calcula el bitrate en Gbps (Gigabits por segundo).
### ● B. ¿Cuánto espacio de disco ocupará una toma de solo 10 segundos?
### ● C. Si tienes un disco duro de 1 TB, ¿cuántos minutos de este vídeo podrías guardar?

**Ejercicio 1: La pesadilla del almacenamiento**

Un estudio graba en RAW (sin comprimir) con:
- Resolución: 4K (3840 × 2160)
- FPS: 60
- Profundidad de color: 30 bits (HDR)

---

**A. Calcula el bitrate en Gbps**

*Datos:*
- Ancho = 3840 píxeles  
- Alto = 2160 píxeles  
- Profundidad = 30 bits  
- FPS = 60  

**Fórmula del bitrate:**

Bitrate = Ancho × Alto × Profundidad × FPS

*Cálculo:*

3840 × 2160 = 8.294.400 píxeles  
8.294.400 × 30 = 248.832.000 bits por frame  
248.832.000 × 60 = 14.929.920.000 bps  

*Resultado:*

≈14,93 Gbps

---

**B. Espacio ocupado por 10 segundos de vídeo**

*Datos:*
- Bitrate = 14,93 Gbps
- Tiempo = 10 s

*Cálculo:*

14,93 Gbps × 10 s = 149,3 Gb  

Pasamos a GB:
149,3 / 8 ≈ 18,66 GB

*Resultado:*

≈18,7 GB en solo 10 segundos

---
**C. ¿Cuántos minutos caben en un disco de 1 TB?**

*Datos:*
- 1 TB ≈ 1000 GB
- Consumo por segundo:  
  18,66 GB / 10 s = 1,866 GB/s

*Tiempo total:*

1000 / 1,866 ≈ 536 s  

Pasamos a minutos:
536 / 60 ≈ *8,9 minutos*

*Resultado:*

≈9 minutos de vídeo

---

### Ejercicio 2: Quieres retransmitir la graduación por YouTube. Tienes una conexión de fibra con 20 Mbps de subida (upload). Quieres emitir en 1080p usando el códec H.264. 
### ● A. Si configuras un bitrate de 6 Mbps, ¿qué porcentaje de tu línea de subida estás consumiendo?
### ● B. Si de repente otros 3 alumnos empiezan a emitir sus propias directos a la misma calidad (6 Mbps cada uno), ¿qué pasará con la emisión? Justifica la respuesta técnica (buffering, saturación de red, latencia).
### ● C. ¿Qué solución técnica aplicarías para que los 4 alumnos puedan emitir simultáneamente sin ampliar la línea de fibra?

**Ejercicio 2: Streaming de una graduación en YouTube**

*Datos:*
- Subida disponible: 20 Mbps
- Bitrate por emisión: 6 Mbps
- Códec: H.264
- Resolución: 1080p

---

**A. Porcentaje de la línea consumida**

**Fórmula:**

Porcentaje = (Bitrate / BW total) × 100

*Cálculo:*

(6 / 20) × 100 = 30 %

*Resultado:*

Se consume el 30 % de la subida.

---

**B. 4 alumnos emitiendo a 6 Mbps**

*Consumo total:*

6 × 4 = 24 Mbps

Ancho de banda disponible: 20 Mbps

*Resultado técnico:*
- Saturación de la red
- Aumento de latencia
- Buffering continuo
- Pérdida de paquetes
- Cortes en la emisión

La red no puede soportar la carga total.


**C. Solución técnica**

*Opciones viables:*
- Reducir el bitrate de cada emisión (ej. 4 Mbps)
- Usar **bitrate adaptativo**
- Emitir a un **servidor intermedio** (RTMP → plataforma)
- Programar emisiones en horarios distintos

*Solución recomendada:*
**Reducir bitrate o usar bitrate adaptativo**, manteniendo la calidad sin saturar la red.


