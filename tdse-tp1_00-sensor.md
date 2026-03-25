El accionamiento de un pulsador mecánico o de un interruptor genera fluctuaciones rápidas e indeseadas en los contactos al cambiar de posición, un fenómeno transitorio conocido como rebote, además de posibles picos de tensión o "glitches". Para abstraer la captura de esta señal física inestable, el modelo define dos eventos fundamentales que actuán de forma continua como disparadores (triggers) en la tabla de transiciones de estados. El evento EV_BTN_XX_UP ocurre cuando el microcontrolador lee un nivel lógico alto en su pin de entrada representando que el dispositivo físico se encuentra en reposo sin presionar. Inversamente, el evento EV_BTN_XX_DOWN se genera en el instante en que la lectura de la entrada digital detecta un nivel bajo de tensión, indicando un cierre de los contactos físicos por la acción del usuario.

Dado que la mera aparición de estos eventos físicos no garantiza un cambio de estado válido debido a la zona de rebote, el modelo Sensor implementa una validación temporal rigurosa recurriendo a una variable de control (timer) identificada como tick. Cuando el sistema se encuentra en un estado estable de reposo y sorpresivamente detecta el evento EV_BTN_XX_DOWN, se desencadena una primera acción preparatoria: la inicialización del temporizador mediante la instrucción tick = DEL_BTN_XX_MAX. Está acción impone un retardo lógico intencional, bloqueando cualquier notificación prematura al resto de la aplicación mientras dura la vibración mecánica.

Una vez que el flujo del programa ingresa en las etapas de transición para evaluar la caída de la señal, cada nuevo ciclo de 1 milisegundo evalúa el estado de las variables mediante condiciones de guarda. Si la guarda determina que aún queda tiempo de bloqueo (tick > 0), el sistema ejecuta de forma continua la acción de decremento del temporizador, expresada en el código como tick--. El objetivo se cumple cuando la condición de guarda verifica finalmente que el tiempo ha expirado (tick==0). En ese milisegundo exacto, si el evento de presión se mantiene constante, el modelo ejecuta su acción de salida definitiva mediante la instrucción raise EV_SYS_XX_DOWN- Esta última acción se encarga de emitir o despachar un mensaje asincrónico hacia el modulo System, confirmando que el actuador cambió de posición de forma limpia y consolidada. Todo este proceso garantiza un comportamiento no bloqueante, manteniendo la aplicación general ágil mientras se resuelven las deficiencias del hardware en segundo plano.

A continuación, se detalla la tabla de transición de estados que modela el comportamiento completo de la etapa de captura y filtrado (debouncing) para un pulsador. La misma contempla los cuatro estados fundamentales por los que atraviesa la señal de acuerdo a las gráficas temporales de tensión: el reposo sin presionar, la transición inestable de bajada, el estado estable de presionado y la transición inestable de subida al liberar los contactos. Cada celda define cómo reacciona el firmware ante la lectura del periférico, evaluando condiciones lógicas sobre el contador de tics para decidir la próxima transición o la ejecución de una acción:

| Current State | Event | [Guard] | Next State | Actions |
| --- | --- | --- | --- | --- |
| ST_BTN_XX_UP  | EV_BTN_XX_UP | | ST_BTN_XX_UP | |
| ST_BTN_XX_UP  | EV_BTN_XX_DOWN | | ST_BTN_XX_FALLING | tick = DEL_BTN_XX_MAX |
| ST_BTN_XX_FALLING | EV_BTN_XX_UP | [tick > 0] | ST_BTN_XX_FALLING | tick-- |
| ST_BTN_XX_FALLING | EV_BTN_XX_UP | [tick == 0] | ST_BTN_XX_UP | |
| ST_BTN_XX_FALLING | EV_BTN_XX_DOWN | [tick > 0] | ST_BTN_XX_FALLING | tick-- |
| ST_BTN_XX_FALLING | EV_BTN_XX_DOWN | [tick = 0] | ST_BTN_XX_DOWN | raise EV_SYS_XX_DOWN |
| ST_BTN_XX_DOWN | EV_BTN_XX_DOWN | | ST_BTN_XX_DOWN | |
| ST_BTN_XX_DOWN | EV_BTN_XX_UP | | ST_BTN_XX_RISING | tick = DEL_BTN_XX_MAX |
| ST_BTN_XX_RISING | EV_BTN_XX_DOWN | [tick > 0] | ST_BTN_XX_RISING | tick-- |
| ST_BTN_XX_RISING | EV_BTN_XX_DOWN | [tick = 0] | ST_BTN_XX_DOWN | |
| ST_BTN_XX_RISING | EV_BTN_XX_UP | [tick > 0] | ST_BTN_XX_RISING | tick-- |
| ST_BTN_XX_RISING | EV_BTN_XX_UP | [tick = 0] | ST_BTN_XX_UP | raise EV_SYS_XX_UP |
