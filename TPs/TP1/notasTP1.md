# Paso a paso:

# TP1-01 – 4to Proyecto p/placa NUCLEO-F103RB con FreeRTOS
1. GitHub:
    - no lo creo porque no tenemos numero de grupo, y una vez creado es raro - cambiarlo. Se crea el miercoles, son 2 segundos.
    Analisis de DeepSeek:
        "Cambiar el nombre de un repositorio (especialmente si ya tiene commits, branches o si otros ya lo clonaron) genera problemas. Mejor esperar a tener el nombre definido"

2. Agregar a los profes al repositorio:
    - idem item 1. se hace el miercoles

3. README.md:
    - creo el archivo `README.md` con mis datos, para despues cargarlo en el github
    - se carga el miercoles

4.  soe-tp1_01-application.zip:
    - conecto la plana NUCLEO
    - abro el workspace del TP0 (soe_workspace)
    - descargo e importo el archivo .zip de jcruz al proyecto, sin descomprimir:
        File > Import > General > Existing Projects into Workspace > Select archive file
    - compilo el archivo -> funciona
    - ahora en el workspace estan los 3 proyectos del TP0 + 1er proyecto del TP1

5. asentar detalles:
    - creo el archivo `tdse-tp1_01-application.md`, en mi compu.

6. Gemini (sporsor oficial de la seleccion Argentina de futbol)
    - entro a gemini con mi cuenta de google personal, la de fiuba no funciona(?
    - subo los archivos a gemini y le paso el prompt del enunciado
    - (gemini se toma su buen tiempo en analizar...)

7. depurar:
    - hago el debug del proyecto, presiono el boton y devuelve:
        [info]  
        [info] app_init is running - Tick [mS] =   0
        [info]  RTOS - Event-Triggered Systems (ETS)
        [info]  soe-tp0_03-application: Demo Code
        [info]  
        [info] Task LED is running - Tick [mS] =   0
        [info]  
        [info] Task BTN is running - Tick [mS] =   1
        [info]  Task BTN - BTN PRESSED
        [info]  Task LED - LED BLINK
        [info]  Task BTN - BTN HOVER
        [info]  Task LED - LED OFF
        [info]  Task BTN - BTN PRESSED
        [info]  Task LED - LED BLINK
        [info]  Task BTN - BTN HOVER
        [info]  Task LED - LED OFF
        [info]  Task BTN - BTN PRESSED
        [info]  Task LED - LED BLINK
        [info]  Task BTN - BTN HOVER
        [info]  Task LED - LED OFF
    - al momento de apretar el boton se prende el led LD2
    - # parece estar bien. a chequear el martes.

8. Gemini (sporsor oficial de la seleccion Argentina de futbol)
    - creo un nuevo chat con gemini 
    - subo los archivos a gemini y le paso el prompt del enunciado

9. depurar (de nuevo):
    - no hago el debug porque va a dar lo mismo claramente.
    - # a chequear el martes

10. GitHub:
    - no subo nada porque no esta el repositorio todavia

# Fin
# ================================================================== # 

# TP1 – Actividad 02 – 5to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

1. renombrar:
    - hice exactamente lo que dice el enunciado pero no funciona.. porque
    # NO SE COPIA LA CONFIGURACIÓN DEL DEBUGGER AUTOMATICAMENTE !!
    - copio la configuracion del debbuger de jcruz.
    dentro del proyecto 02: Run > Debug Configurations > 
        en la columna de la izquierda, hay que duplicar el la configuracion `soe-tp1_01-application` y cambiarle los parámetros:
        name, project, C/C++ Application, con el nombre `soe-tp1_02-application`
        y verificar que `Build Configuration` este en `Select Automatically`.
    - ya funciona igual que el 01. 

2. preguntas teoricas:
    - creo el archivo `tdse-tp1_02-application.md`
    - respondo todas las preguntas con la IA
    - # el martes hay que entender las respuestas

3. modificar propridades `task_btn` y `task_led`:
    - *como se hace para modificar las prioridades?*
        #### Prueba A: Prioridad de Botón > Prioridad de LED
        # teoria:
        1.  En `app.c`, cambia la prioridad de `h_task_btn` a `(tskIDLE_PRIORITY + 2ul)`.
        2.  Mantén la de `h_task_led` en `+ 1ul`.
        3.  **Compila y Depura:** Observa qué sucede.
            * **Comportamiento esperado:** Debido a que `task_btn` tiene un bucle infinito `for(;;)` y **no** contiene funciones de bloqueo como `vTaskDelay()`, esta tarea siempre estará en estado "Ready". Al tener mayor prioridad, el programador de FreeRTOS le dará todo el tiempo de CPU a ella, y el LED probablemente deje de parpadear o se comporte de forma errática (Inanición o Starvation).
        # practica:
        - Cambie la prioridad del boton. Debugue el programa, y probe el boton. Y en la terminal me queda:
            [info]  
            [info] app_init is running - Tick [mS] =   0
            [info]  RTOS - Event-Triggered Systems (ETS)
            [info]  soe-tp0_03-application: Demo Code
            [info]  
            [info] Task BTN is running - Tick [mS] =   0
            [info]  Task BTN - BTN PRESSED
            [info]  Task BTN - BTN HOVER
            [info]  Task BTN - BTN PRESSED
            [info]  Task BTN - BTN HOVER
            [info]  Task BTN - BTN PRESSED
            [info]  Task BTN - BTN HOVER
            [info]  Task BTN - BTN PRESSED
            [info]  Task BTN - BTN HOVER
        - parece estar bien... 
        - escribo todo esto en: `tdse-tp1_02-application.md`.


        #### Prueba B: Prioridad de LED > Prioridad de Botón
        # teoria:
        1.  Invierte los valores: `task_led` a `+ 2ul` y `task_btn` a `+ 1ul`.
        2.  **Compila y Depura:**
            * **Comportamiento esperado:** Ahora es la tarea del LED la que tiene el control total. Es muy probable que el sistema ignore por completo tus pulsaciones en el botón, ya que la tarea que lee el pin GPIO nunca recibe tiempo de ejecución.
        # practica:
        - Cambie la prioridad del led. Debugue el programa, y probe el boton. Y en la terminal me queda:
            [info]  
            [info] app_init is running - Tick [mS] =   0
            [info]  RTOS - Event-Triggered Systems (ETS)
            [info]  soe-tp0_03-application: Demo Code
            [info]  
            [info] Task LED is running - Tick [mS] =   0
        - tambien parece estar bien... 

    - vuelvo a dejar todo como estaba: `task_led` en `+ 1ul` y `task_btn` en `+ 1ul`:
        funciona todo igual que antes (TP1-01 - Paso 07)

4. crear 3 instacias de `task_btn`:
    *como se crean las 3 instancias?*

    ### 1. Modificar `app.c`: Crear las tres instancias
    Para poder borrar una tarea específica más adelante, necesitamos guardar sus "identificadores" (Handles). Actualmente solo tienes uno.

    1.  **Declara los nuevos Handles:** En la sección de variables globales de `app.c`, añade dos más:
        ```c
        /* h_task_btn ya existe, la usaremos como la primera instancia */
        TaskHandle_t h_task_btn_2;
        TaskHandle_t h_task_btn_3;
        ```

    2.  **Instancia las tareas en `app_init()`:** Llama a `xTaskCreate` tres veces. Te recomiendo cambiarles el nombre en el segundo parámetro para que en la terminal puedas distinguir quién es quién:
        ```c
        /* Task BTN thread at priority 1 */
        ret = xTaskCreate(task_btn,							/* Pointer to the function thats implement the task. */
                        "Task BTN",						/* Text name for the task. This is to facilitate debugging only. */
                        (2 * configMINIMAL_STACK_SIZE),	/* Stack depth in words. */
                        NULL,								/* We are not using the task parameter. */
                        (tskIDLE_PRIORITY + 1ul),			/* This task will run at priority 1. */
                        &h_task_btn);						/* We are using a variable as task handle. */
        /* Check the thread was created successfully. */
        configASSERT(pdPASS == ret);

        /* Nueva instancia (2) creada */
        ret = xTaskCreate(task_btn,							/* Pointer to the function thats implement the task. */
                        "Task BTN 2",						/* Text name for the task. This is to facilitate debugging only. */
                        (2 * configMINIMAL_STACK_SIZE),	/* Stack depth in words. */
                        NULL,								/* We are not using the task parameter. */
                        (tskIDLE_PRIORITY + 1ul),			/* This task will run at priority 1. */
                        &h_task_btn_2);					/* We are using a variable as task handle. */

        /* Check the thread was created successfully. */
        configASSERT(pdPASS == ret);

        /* Nueva instancia (3) creada */
        ret = xTaskCreate(task_btn,							/* Pointer to the function thats implement the task. */
                        "Task BTN 3",						/* Text name for the task. This is to facilitate debugging only. */
                        (2 * configMINIMAL_STACK_SIZE),	/* Stack depth in words. */
                        NULL,								/* We are not using the task parameter. */
                        (tskIDLE_PRIORITY + 1ul),			/* This task will run at priority 1. */
                        &h_task_btn_3);					/* We are using a variable as task handle. */

        /* Check the thread was created successfully. */
        configASSERT(pdPASS == ret);
        ```

    ---

    ### 2. Modificar `task_led.c`: Eliminar una instancia
    Ahora vamos a hacer que la tarea del LED actúe como "verdugo" de una de las tareas del botón.

    1.  **Dar acceso al Handle:** Al principio de `task_led.c`, indica que vas a usar el handle que vive en `app.c`:
        ```c
        extern TaskHandle_t h_task_btn; // Vamos a borrar la primera instancia
        ```

    2.  **Ejecutar el borrado:** Dentro de la función `task_led`, justo después de la inicialización (donde imprime el mensaje de "Task LED is running") pero **antes** del bucle `for(;;)`, añade la función de eliminación:
        ```c
        void task_led(void *parameters) {
            /* Print out: Task Initialized */
            LOGGER_INFO(" ");
            LOGGER_INFO("%s is running - Tick [mS] = %3d", pcTaskGetName(NULL), (int)xTaskGetTickCount());

            HAL_GPIO_WritePin(task_led_dta.gpio_port, task_led_dta.pin, LED_OFF);

            /* Eliminacion de la instancia BTN 1 (TP1 – Actividad 02 - paso 04) */
            if (h_task_btn != NULL)
            {
                vTaskDelete(h_task_btn);
                LOGGER_INFO(" %s - INSTANCIA BTN 1 ELIMINADA", pcTaskGetName(NULL));

                h_task_btn = NULL; // handle a NULL despues de borrarlo
            }

            /* As per most tasks, this task is implemented in an infinite loop. */
            for (;;)
            {
                /* Print out: Task execution */
                //LOGGER_INFO(" %s - Tick [mS] = %3d", pcTaskGetName(NULL), (int)xTaskGetTickCount());

                /* Run Task Statechart */
                task_led_statechart();
            }
        }
        

    - Hice todos estos cambios pero no funciona. Ni siquiera funciona como en su forma default. Asi que ...
    ## modifico el tamaño del heap_size de FreeRTOS de 3072 B a 8192 B!!!! 
    # esta bien hacer esto? a chequear

    - Ahora si funciona.
    [info]  
    [info] app_init is running - Tick [mS] =   0
    [info]  RTOS - Event-Triggered Systems (ETS)
    [info]  soe-tp0_03-application: Demo Code
    [info]  
    [info] Task LED is running - Tick [mS] =   0
    [info]  Task LED - INSTANCIA BTN 1 ELIMINADA
    [info]  
    [info] Task BTN 2 is running - Tick [mS] =   1
    [info]  
    [info] Task BTN 3 is running - Tick [mS] =   2
    [info]  Task BTN 2 - BTN PRESSED
    [info]  Task LED - LED BLINK
    [info]  Task BTN 2 - BTN HOVER
    [info]  Task LED - LED OFF
    [info]  Task BTN 2 - BTN PRESSED
    [info]  Task BTN 3 - BTN PRESSED
    [info]  Task LED - LED BLINK
    [info]  Task BTN 2 - BTN HOVER
    [info]  Task LED - LED OFF

    - lo asiento en `tdse-tp1_02-application.md`.

# Fin
# ================================================================== # 



El procesamiento periódico significa que la tarea se ejecute cada cierto tiempo fijo (ej: cada 100 ms), y el resto del tiempo se bloquee para que otras tareas usen el CPU.








