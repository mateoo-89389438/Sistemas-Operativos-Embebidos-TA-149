# TP3 – Sincronización de Tareas de FreeRTOS

- Los propositos de este TP3 son 2 principalmente:
    1. Eliminar la sobrecarga reemplazando retardos innecesarios (`vTaskDelay`) por estados de bloqueo eficaces. Las tareas solo tienen que consumir tiempo de procesamiento cuando hay un estímulo real del entorno.
    2. Garantizar la Exclusión Mutua: protegiendo recursos compartidos frente a accesos concurrentes.

--- 
## TP3 – Actividad 01 – 12vo Proyecto p/placa NUCLEO-F103RB con FreeRTOS

En el libro *"The Little Book of Semaphores"* se presenta el modelo productor-consumidor. Usando esto el objetivo es sincronizar dos tareas A y B que antes corrían en paralelo de forma descoordinada, forzando a que uno espere la señalización de la otra.
Se configuró a A como productor y a B como consumidor.

- Se hicieron modificaciones en el codigo:
    1. Se creo el semaforo binario global `xSemaphore_Producer_Consumer` en `app.c` iniciado en 0 con `xSemaphoreCreateBinary()`.
    2. El productor `task_a` simula su procesamiento con un delay de 250ms y luego entrega el semaforo con `xSemaphoreGive()`
    3. Al consumir `task_b` se le eliminó la espera activa, haciendo que solo se active cuando recibe el semaforo con `xSemaphoreTake()` y luego pase a estar bloqueada.

--- 
## TP3 – Actividad 02 – 13er Proyecto p/placa NUCLEO-F103RB con FreeRTOS

Hay 3 archivos:

1. `app.c`: es el principal. Crea las tareas, asigna las prioridades, imprime la informacion por pantalla, etc.
2. `task_a` y `task_b`: es el archivo de cada tarea. Tienen un contador que se incrementa cada vez que se ejecuta la tarea. Y un delay establecido.
3. `app_it.c`: es el archivo que maneja las interrupciones.

- En el libro *"The Little Book of Semaphores"* Se plantea el concepto de "El cuarto" (room) para manejar un recurso compartido. El 4 tiene reglas estrictas:
    1. Multiples lectores pueden estar en el cuarto al mismo tiempo leyendo 
    2. Solo un escritor puede estar en el cuarto a la vez escribiendo.
    3. Un escritor solo puede entrar cuando el cuarto está vacio.
    4. Nadie (lectores ni escritores) puede entrar mientras un escritor está adentro. 
    - El semaforo binario actua como "llave" del cuarto.
    - El mutex protege la variable compartida.

- La idea de la modificacion es:
    - El escritor = `task_a`: genera un elemento y avisa cuando ya está listo
    - El lector = `task_b`: se queda bloqueado esperando a que el productor le avise. Solo cuando recibe la señal se despierta y procesa el elemento.
    - para manejar esta logica utilizamos un semafoto binario y un mutex. El semaforo binario: es la llave de acceso al cuarto

- Modificaciones en el codigo:
    1. se crean 3 recursos en `app.c`:

        1.1. Un contador (`readers_cnt`) que cuenta la cantidad de lectores en el cuarto. 

        1.2  Un mutex (`xSemaphoreCreateMutex()`) para proteger esto. Para que no pase que mientras una tarea va a cambiar el valor, pierde la prioridad y no llega a actualizar el contador.
        
        1.3  Un semaforo binario (`xSemaphoreCreateBinary()`) como llave de acceso al cuarto. 

    2. Se escribe el codigo del escritor en `task_a.c`:
        El escritor pide la llave de acceso al cuarto: `xSemaphoreTake()`, de forma bloqueante hasta terminar. 
	    Cuando termina, sale del cuarto y devuelve la llave `xSemaphoreGive()`. 
    3. Se escribe el codigo del lector en `task_b.c`:
        El lector tiene que proteger el contador de lectores con el Mutex. 
        El primer lector que llega cierra el cuarto para los escritores, y el último lector que se va lo abre.

---
## TP3-03 – 14to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

- Se plantea un sistema de control de acceso que monitoria el ingreso y egreso de vehiculos. Las reglas son:
    1. No puede haber mas de N vehiculos en el puente.
    2. Hay 2 entradas y 2 salidas
        2.1 Las entradas se manejan en `entry_a` y `entry_b`
        2.2 Las salidas se manejan en `exit_a` y `exit_b`

- Modificiones de codigo:
    1. Se crean 4 semaforos en `app.c` para manejar el estado de cada entrada y cada salida: `entry_a, exit_a, entry_b, exit_b`
    **IMPORTANTE** : El semaforo binario es la "flag" avisando que un auto llego o salio. No es el semaforo verde/rojo de transito.
    2. En `task_test.c` se agregan los `xSemaphoreGive()` avisando cada vez que un vehiculo llega al ingreso o egreso A o B
    3. Se modifican las tareas para que dejen de usar delay y simplemente se queden bloquadas esperando su semaforo.
    4. Se aplica la logica de control para cumplir con la capacidad maxima de cruce en la interseccion. Se necesita:

        4.1: Un contador global (`uint32_t g_crossing_cnt`) para saber cuantos autos hay en el cruce

        4.2: Un Mutex para proteger el contador (`SemaphoreHandle_t xMutex_Crossing`). 

        4.3: Un semaforo binario (`SemaphoreHandle_t xSem_SpaceAvailable`) para manejar la logica del semaforo de transito rojo/verde. 

    En `task_test.c` están definidos varios casos de entrada y salida de tareas en la interseccion. Para cambiar el caso se modifica `#define E_TASK_TEST_X (3)`.


---
---
## TP3 – Actividad 04 – 15to Proyecto p/placa NUCLEO-F103RB con FreeRTOS

- Se plantea un sistema de puertas esclusa:
    - En una camara de alta seguridad (como la boveda de un banco) para entrar y salir no se pasa por una sola puerta, si no que se atraviesa un habitáculo intermedio con dos o mas puertas.
    - La regla inquebrantable es *nunca puede haber dos puertas abiertas al mismo tiempo*. 
    - En este contexto, cada puerta está representada por una tarea (`task_gate_a`, `task_gate_b`).
    - Para no gastar procesador las tareas están siempre dormidas y solo se despiertan cuando `task_test` les avisa que "alguien quiere abrir la puerta" con un `xSemaphoreGive()`.
    - Para asegurarnos que solo una puerta se abra, se crea un recurso compartido por todas las puertas: el Mutex (que seria como una unica llave para todas las puertas) `xMutex_Airlock`.

    1. Se implementaron semáforos binarios globales de solicitud y cierre (ej. `xSem_OpenRequest_B` y `xSem_DoorClosed_B`) inicializados en 0 en `app.c`.
    2. Se usan 4 semaforo binarios, uno para los eventos de cada puerta.
    2. Se usa el mutex como llave de las puertas



