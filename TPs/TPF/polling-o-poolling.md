Para responder con precisión, primero es importante aclarar una pequeña confusión conceptual de términos: **polling no es un tipo de memoria, sino un modo de transferencia o comunicación de la CPU con un periférico** (en este caso, el bus SPI conectando al microcontrolador con la memoria Flash NOR).

En el marco del Trabajo Final, las 14 actividades dividen el mecanismo de transferencia del driver SPI en tres familias bien definidas:

---

## 1. Actividades que SÍ utilizan Polling (Actividades 01, 02, 07 y 08)

### 📌 ¿En cuáles se usa?

* **Actividades 01 y 02:** Transmisión (Escritura) por Polling.


* **Actividades 07 y 08:** Recepción (Lectura) por Polling.



### ❓ ¿Por qué se usa Polling en estas actividades?

* **Mecanismo:** La CPU ejecuta la función bloqueante `HAL_SPI_Transmit()` o `HAL_SPI_Receive()`. Durante todo el tiempo que tarda la transferencia en el bus SPI, la CPU queda **atrapada en un bucle cerrado (`while`)** consultando los registros del periférico SPI hasta que se envía o recibe el último bit.


* **Objetivo pedagógico:** Serve como punto de partida base para implementar el patrón **Gatekeeper Task**. Permite validar el correcto funcionamiento de las colas (Queues), la exclusión mutua y el cableado físico de los 6 pines sin la complejidad añadida de gestionar interrupciones ni vectores de interrupción.



---

## 2. Actividades que NO utilizan Polling

En el resto de las actividades, el mecanismo de Polling se descarta por completo y se reemplaza por mecanismos asíncronos eficientes:

### A. Transmisiones / Recepciones por Interrupción (IT)

* **Actividades:** 03, 04, 09, 10 y 11.


* **¿Por qué NO se usa Polling?:** La *Gatekeeper Task* inicia la operación con `HAL_SPI_Transmit_IT()` o `HAL_SPI_Receive_IT()` y **cede inmediatamente la CPU**, quedando en estado bloqueado (*Blocked*) mediante un semáforo o una notificación de tarea. Cuando el hardware del SPI termina de transferir los bytes, dispara una interrupción (ISR) que despierta a la *Gatekeeper Task*. La CPU queda libre para ejecutar otras tareas durante el tiempo de transferencia.



### B. Transmisiones / Recepciones por Acceso Directo a Memoria (DMA)

* **Actividades:** 05, 06, 12, 13 y 14.


* **¿Por qué NO se usa Polling?:** La *Gatekeeper Task* le delega la transferencia completa al controlador DMA mediante `HAL_SPI_Transmit_DMA()` o `HAL_SPI_Receive_DMA()`. El hardware de DMA transfiere los datos directamente entre la memoria RAM y el periférico SPI sin que la CPU procese un solo byte. Al finalizar, el DMA genera una interrupción para avisar a la tarea.



---

## 📊 Cuadro Resumen de la Evolución

| Actividades | Operación SPI | Mecanismo de Transferencia | ¿Usa Polling? | Impacto en la CPU |
| --- | --- | --- | --- | --- |
| **01 y 02**<br> | Transmisión

 | **Polling** (`HAL_SPI_Transmit`)

 | **SÍ** | Alto (CPU bloqueada esperando bytes)

 |
| **03 y 04**<br> | Transmisión

 | **Interrupción** (`HAL_SPI_Transmit_IT`)

 | **NO** | Bajo (CPU liberada durante la transferencia)

 |
| **05 y 06**<br> | Transmisión

 | **DMA** (`HAL_SPI_Transmit_DMA`)

 | **NO** | Nulo (Manejo $100\%$ por hardware)

 |
| **07 y 08**<br> | Recepción

 | **Polling** (`HAL_SPI_Receive`)

 | **SÍ** | Alto (CPU bloqueada esperando bytes)

 |
| **09, 10 y 11**<br> | Recepción

 | **Interrupción** (`HAL_SPI_Receive_IT`)

 | **NO** | Bajo (CPU liberada durante la transferencia)

 |
| **12, 13 y 14**<br> | Recepción

 | **DMA** (`HAL_SPI_Receive_DMA`)

 | **NO** | Nulo (Manejo $100\%$ por hardware)

 |