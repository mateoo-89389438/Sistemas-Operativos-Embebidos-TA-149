# TP2 – Comunicación de Tareas de FreeRTOS

El proposito de este TP2 es aprender a comunicar tareas de forma segura.

--- 
## TP2 – Actividad 01 – 4to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

- Hacer consultas a Gemini
- Entender el codigo
- Asentar todo en el archivo `markdown`
- No hubo modificaciones en el codigo

Hay 3 tareas definidas (mismas del TP1):

1. `task_bnt`: es la tarea que maneja el boton (presionado/ no presionado) con antirrebote
2. `task_led`: es la tarea que maneja el LED (encido/apagado)
3. `task_led_interfece`: una tarea intermedia entre el boton y el led para que el botón no pueda comunicarse directamente con el LED.

--- 
---

## TP2 – Actividad 02 – 5to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

Se buscó reemplazar la comunicación indirecta basada en funciones de interfaz global por un mecanismo de colas.

- Se realizaron las siguientes modificaciones en el código:
    1. Se declaró el manejador global de la cola (`xButtonQueue`) y se la inicializó en `app.c` usando `xQueueCreate()`.
    2. En `task_btn.c`, dentro de la máquina de estados (`ST_BTN_XX_FALLING`), se eliminó la función anterior de interfaz y se implementó el envío de datos de manera no bloqueante mediante `xQueueSend()`.
    3. En `task_led.c`, dentro de `task_led_statechart()`, se incorporó la recepción del evento consultando la cola de forma no bloqueante mediante `xQueueReceive() == pdTRUE` para activar el flag de la máquina de estados sin romper el temporizado de los parpadeos.

--- 
---

## TP2 – Actividad 03 – 6to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

Se busco modificar la comunicación entre las task_btn y task_led , recurrir a una Semáforo binario.

- Se hicieron 2 pasos de modificaciones de codigo:
    1. Se creo el semaforo binario en `app.c` con `xSemaphoreCreateBinary()`
    2. Como la comunicacion entre boton y led se realiza mediante `task_led_interface.c`, en este archivo se agrego `xSemaphoreGive()` para que entregue el semaforo cuando se produce el evento `h_sem_led_event`.
    3. Para que el led maneje la logica tambien del semaforo, dentro de `task_led.c` en la funcion `task_led_statechart()` se agregó `xSemaphoreTake()`.

*Importante: como en este caso el LED tiene que entrar obligatoriamente cada 50ms a hacer el parpadeo, si se bloqueara esperando el botón, el LED dejaría de titilar. Por eso se uso un semaforo binario **no bloqueante**

--- 
---

## TP2 – Actividad 04 – 7mo Proyecto p/placa NUCLEO-F103RB con FreeRTOS

El objetivo principal es eliminar por completo el polling (la consulta periódica) para leer el botón. En vez de que la tarea del botón se despertierte todo el tiempo a "mirar" si se habia presionado el pin, hay que configurar el hardware para que el propio botón le avise al sistema operativo mediante una interrupción.

- Se hicieron 4 pasos de modificaciones de codigo:
    1. Se modificó el modo del pin GPIO del boton azul (PC13) y se lo puso en `External interrupt mode` con flaco de bajada `Falling Edge` para que la señal se dispare justo al presionar el pulsador.
    2. Se creo el semaforo binario en `app.c` con `xSemaphoreCreateBinary()`
    3. Se declaró la funcion `HAL_GPIO_EXTI_Callback()` que detecta mediante funciones de la HAL cuando el botón fue presionado. Cuando pasa esto entrega el semaforo binario con `xSemaphoreGiveFromISR()`.
    4. Para que el boton se maneje efectivamente mediante el semaforo binario se elimino `vTaskDelay()` y se agregó `xSemaphoreTake()` que lo envia la HAL del punto 3.

*Importante: como en este caso no hay polling, el botón no se necesita estar haciendo nada mas que no sea esperar a ver si fue presionado. Entones por eso se uso un semaforo binario **bloqueante** `portMAX_DELAY`. 