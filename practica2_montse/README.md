
# Práctica 2

## Código 1

```c++

     #include <Arduino.h>

     struct Button {
     const uint8_t PIN;
     uint32_t numberKeyPresses;
     bool pressed;
     };
     Button button1 = {18, 0, false};
     void IRAM_ATTR isr() {
     button1.numberKeyPresses += 1;
     button1.pressed = true;
     }

     void setup() {
     Serial.begin(115200);
     pinMode(button1.PIN, INPUT_PULLUP);
     attachInterrupt(button1.PIN, isr, FALLING);
     }

     void loop() {
     if (button1.pressed) {
     Serial.printf("Button 1 has been pressed %u times\n",
     button1.numberKeyPresses);
     button1.pressed = false;
     }
     //Detach Interrupt after 1 Minute
     static uint32_t lastMillis = 0;
     if (millis() - lastMillis > 60000) {
     lastMillis = millis();
     detachInterrupt(button1.PIN);
     Serial.println("Interrupt Detached!");
     }
     }

 ```


Este código representa un botón, el cuál tiene información sobre el pin al que está conectado con resistencia de pull-up interna, este  contiene el número de pin, el número de veces que ha sido presionado y si ha sido presionado o no.

Se define la función **isr()**, que es la función de interrupción adjuntada al pin que se activará cuando se presioné el botón. Dentro de esta función, se incrementa el contador de pulsaciones del botón y se establece el indicador de presionado en verdadero.

En el **setup()**, se configura el botón antes de entrar en el loop. Se incializa la comunicación serial, se establece el pin del botón y se adjunta la función de interrupción isr() al pin del botón para poder manejar las pulsaciones.
    
Mediante el **loop()** comprueba si el botón ha sido presionado o no y si el botón ha sido presionado te dice las veces y después de un minuto de interrupción este se desvincula de la interrupción.

El programa imprimirá por pantalla el mensaje : "Button 1 has been pressed x times", con el número de veces que ha sido presionado y cuando haya pasado un minuto de haberse adjuntado la interrupción se imprimirá el mensaje "Interrupt Detached!".

## Código 2

```c++

    #include <Arduino.h>

    volatile int interruptCounter;
    int totalInterruptCounter;
    hw_timer_t * timer = NULL;
    portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;
    void IRAM_ATTR onTimer() {
    portENTER_CRITICAL_ISR(&timerMux);
    interruptCounter++;
    portEXIT_CRITICAL_ISR(&timerMux);
    }

    void setup() {
    Serial.begin(115200);
    timer = timerBegin(0, 80, true);
    timerAttachInterrupt(timer, &onTimer, true);
    timerAlarmWrite(timer, 1000000, true);
    timerAlarmEnable(timer);
    }

    void loop() {
    if (interruptCounter > 0) {
    portENTER_CRITICAL(&timerMux);
    interruptCounter--;
    portEXIT_CRITICAL(&timerMux);
    totalInterruptCounter++;
    Serial.print("An interrupt as occurred. Total number: ");
    Serial.println(totalInterruptCounter);
    }
    }

  ```


El código muestra un temporizador que cuenta el número de interrupciones y luego muestra el número total de estas.

Para ello declara dos contadores, uno se llama **InterruptCounter**  que se usa para ir contando el número de interrupciones y luego está **totalInterruptCounter** que se usa para mantener el recuento total de interrupciones.

Declara un puntero a una estructura de tipo hw_timer_t llamado timer. Esta estructura se utiliza para representar el temporizador hardware que se utilizará en el programa.

Declara una variable timerMux de tipo portMUX_TYPE que se utiliza para gestionar el acceso a recursos críticos compartidos entre la rutina de interrupción y el resto del programa. Se inicializa con el valor portMUX_INITIALIZER_UNLOCKED para indicar que el recurso compartido está desbloqueado al principio.

Define la función de interrupción **onTimer()** que se ejecutará cada vez que se detecte una interrupción. Incrementa interruptCounter para contar la interrupción y luego se utiliza portENTER_CRITICAL_ISR() Y portEXIT_CRITICAL_ISR() para asegurar que el incremento se ha realizado de manera segura y evitar conflictos entre la función de interrupción y otras funciones del programa.

La función **setup()** se ejecuta al inicio del programa. Este inicializa el temporizador hardware utilizando la función timerBegin(), adjunta la función de interrupción onTimer() al temporizador utilizando timerAttachInterrupt(), configura el temporizador para generar una interrupción cada 1 segundo con timerAlarmWrite(), y finalmente, habilita la interrupción del temporizador utilizando timerAlarmEnable().

La función **loop()**  se ejecuta continuamente después de que la función setup() haya finalizado. En este caso, comprueba si ha ocurrido alguna interrupción. Si es así, decrementa interruptCounter para indicar que se ha procesado una interrupción, incrementa totalInterruptCounter y luego muestra un mensaje en el puerto serie indicando el número total de interrupciones ocurridas hasta el momento.

## Conclusión

Se han configurado dos codigos que usan interrupciones para contar eventos.

El primer código usa **attachInterrupt** que es una interrupción externa para detectar cambios en el estado del botón y contar cuántas veces se ha presionado este; en cambio el segundo código utiliza el **timer** para generar interrupciones periódicas.

En coclusión, hemos usado interrupciones externas (attachInterrupt) para contar eventos externos y usamos interrupciones internas (timer) para contar eventos internos.  

