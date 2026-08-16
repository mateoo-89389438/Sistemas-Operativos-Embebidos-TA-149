# Paso a paso:

# TP3-01 – 12vo Proyecto p/placa NUCLEO-F103RB con FreeRTOS
09. Modificar sincronizacion entre task_a y task_b:
    - antes de cambiar nada: 
        las dos tareas a y b hacen lo mismo de formas indeptes. Incrementan su contador y se duermen usando vTaskDelay()
        
    - la idea de la modificacion es:
        - El productor = task_a: genera un elemento y avisa cuando ya está listo
        - El consumidor = task_b: se queda bloqueado esperando a que el productor le avise. Solo cuando recibe la señal se despierta y procesa el elemento.
        - para manejar esta logica utilizamos un semafoto binario

    1. creo el semaforo en app.c (se crea en 0, lo cual es ideal)
    2. 'A' que es el productor, incrementa su propio contador, simulando la creacion del element, y luego pone el elemento a disposicion (entrega el semaforo). Para entregar el semaforo: `xSemaphoreGive`
    3. 'B' que es el consumidor, ya no incrementa el contador de forma autonoma si no que se queda bloqueada hasta que A le avise.

-------------------------------
# TP3-02 – 13er Proyecto p/placa NUCLEO-F103RB con FreeRTOS
09. Modificar sincronizacion entre task_a y task_b:
    Previo:
        Se plantea el concepto de "El cuarto" (room) para manejar un recurso compartido. El cuarto tiene reglas estrictas
            1. Multiples lectores pueden estar en el cuarto al mismo tiempo leyendo 
            2. Solo un escritor puede estar en el cuarto a la vez escribiendo.
            3. Un escritor solo puede entrar cuando el cuarto está vacio.
            4. Nadie (lectores ni escritores) puede entrar mientras un escritor está adentro. 

        - El semaforo binario actua como "llave" del cuarto.
        - El mutex protege la variable compartida.

    - la idea de la modificacion es:
        - El escritor = task_a: genera un elemento y avisa cuando ya está listo
        - El lector = task_b: se queda bloqueado esperando a que el productor le avise. Solo cuando recibe la señal se despierta y procesa el elemento.
        - para manejar esta logica utilizamos un semafoto binario y un mutex
            El semaforo binario: es la llave de acceso al cuarto

    1. se crean 3 recursos en app.c:
        1.1. Un contador (`readers_cnt`) que cuenta la cantidad de lectores en el cuarto. 
        1.2  Un mutex para proteger esto. Para que no pase que mientras una tarea va a cambiar el valor, pierde la prioridad y no llega a actualizar el contador.
        1.3  Un semaforo binario como llave de acceso al cuarto. 
    2. Se escribe el codigo del escritor en `task_a.c`:
        El escritor pide la llave de acceso al cuarto: `xSemaphoreTake`, de forma bloqueante hasta terminar. 
	    Cuando termina, sale del cuarto y devuelve la llave `xSemahpreGive`. 
    3. Se escribe el codigo del lector en `task_b.c`:
        El lector tiene que proteger el contador de lectores con el Mutex. 
        El primer lector que llega cierra el cuarto para los escritores, y el último lector que se va lo abre.


-------------------------------
# TP3-03 – 14to Proyecto p/placa NUCLEO-F103RB con FreeRTOS
09. Modificar sincronizacion entre task_a y task_b: sistema de control de acceso

    1. Se crean 4 semaforos en `app.c`: entry_a, exit_a, entry_b, exit_b
    2. En `task_test.c` se agregan los `xSemaphoreGive()` avisando cada vez que un vehiculo llega al ingreso o egreso A o B
    3. Se modifican las tareas para que dejen de usar delay y simplemente se queden bloquadas esperando su semaforo.
    4. Se aplica la logica de control para cumplir con la capacidad maxima de cruce en la interseccion. Se necesita:
        4.1: Un contador global para saber cuantos autos hay en el cruce
        4.2: Un Mutex para proteger el contador. 
        4.3: Un semaforo binario para manejar la logica del semaforo rojo/verde

    En `task_test.c` están definidos varios casos de entrada y salida de tareas en la interseccion. Para cambiar el caso se modifica `#define E_TASK_TEST_X (3)`.


-------------------------------
# TP3 – Actividad 04 – 15to Proyecto p/placa NUCLEO-F103RB con FreeRTOS
09. Modificar sincronizacion entre task_a y task_b: sistema de control de acceso

Sistema de puertas esclusa:
    En una camara de alta seguridad (como la boveda de un banco) para entrar y salir no se pasa por una sola puerta, si no que se atraviesa un habitáculo intermedio con dos o mas puertas.
    La regla inquebrantable es *nunca puede haber dos puertas abiertas al mismo tiempo*. 
    En este contexto, cada puerta está representada por una tarea (`task_gate_a`, `task_gate_b`).
    Para no gastar procesdor las tareas está siempre dormidas y solo se despiertan cuando `task_test` les avisa que "alguien quiere abrir la puerta".
    Para asegurarnos que solo una puerta se abra, se crea un recurso compartido por todas las puertas: el Mutex (que seria como una unica llave para todas las puertas)

    1. Se usan 4 semaforo binarios, uno para los eventos de cada puerta.
    2. Se usa el mutex como llave de las puertas










