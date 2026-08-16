# TP1 – Tareas de FreeRTOS

El proposito de este TP1 es romper el paradigma del super-loop y pasar a implementar un paradigma de sistemas operativos en tiempo real (RTOS).

--- 
## TP1 – Actividad 01 – 4to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

- Hacer consultas a Gemini
- Entender el codigo
- Asentar todo en el archivo `markdown`
- No hubo modificaciones en el codigo

Hay 3 tareas definidas:

1. `task_bnt`: es la tarea que maneja el boton (presionado/ no presionado) con antirrebote
2. `task_led`: es la tarea que maneja el LED (encido/apagado)
3. `task_led_interfece`: una tarea intermedia entre el boton y el led para que el botón no pueda comunicarse directamente con el LED.

--- 
---

## TP1 – Actividad 02 – 5to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

- Se hicieron 3 pasos de modificaciones de codigo:
    1. Prioridad de `task_btn` > prioridad de `task_led`
        El boton se inicia y funciona pero el led queda apagado.
    2. Prioridad de `task_led` > prioridad de `task_btn`
        El led se inicia pero como nada lo activa queda apagado.
    3. 
        - Se crearon 3 tareas indeptes `task_btn`(`BTN 1`, `BTN 2` y `BTN 3`) dentro de `app.c`.
        
        - Se modificó `task_led` para hacer que elimine una de las tareas creadas en el paso 1, con `vTaskDelete(h_task_btn)`. 
        - El boton enciende el led con las tareas 2 y 3

---
---
## TP1 – Actividad 03 – 6to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

- Se hicieron 2 pasos de modificaciones de codigo:
    1. Modificacion de `task_led` para probar.
    2. 
        - Se crearon 2 instancias de `task_btn` para manejar 2 botones diferentes: el boton azul de la _NUCLEO y un boton externo.
        - Se creo la estructura `task_btn_dta_t` en `task_btn.h` y se definieron 2 instancias de estructura
        - Se modificaron las funciones `task_btn()` y `task_btn_statechart()` para manejar el funcionamiento de los 2 botones
        - En `app.c` se creo una instancia de tarea para cada boton.

---
---

## TP1 – Actividad 04 – 7mo Proyecto p/placa NUCLEO-F103RB con FreeRTOS

- Se hizo una unica modificacion en el codigo:
    1. Se implemento el procesamiento periodico de las tareas `task_btn` y `task_led` con agregango la funcion `vTaskDelay()` en cada una.