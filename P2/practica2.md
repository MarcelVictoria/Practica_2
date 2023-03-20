# PRACTICA 2: Interrupciones
<div align=right> 

Victòria Blanco Bosch 
Marcel Farelo de la Orden
<div>

<div align="justify">

El objetivo de la practica es comprender el funcionamiento de las interrupciones.

<div>

 ## MATERIAL
- Cables
- ESP32

## PRACTICA A
<div align="justify">

La siguiente instrucción establece un pin de interrupción, el nombre de la función que llama cada vez que se dispare la interrupción y cuándo debe dispararse. En este caso *falling* indica que la interrupción sucede cuando se aprieta el botón.
```cpp
attachInterrupt(button1.PIN, isr, FALLING);
```
La función que llamamos pasa a true la variable button1.pressed y cuenta los golpes que ha sido apretado el botón. En el loop* tenemos un condicional que muestra por pantalla el número de veces que ha sido apretado el botón.
```cpp
if (button1.pressed) {
Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
button1.pressed = false;
}
```

En ese mismo código también hay una itnerrupción por Timer (evento programado). Es decir, la interrupción tiene lugar después de un tiempo programado.
```cpp
//Detach Interrupt after 1 Minute
static uint32_t lastMillis = 0;
if (millis() - lastMillis > 60000) {
lastMillis = millis();
detachInterrupt(button1.PIN);
Serial.println("Interrupt Detached!");
}
```
En ese caso después de un minuto se desactivan las interrupciones del pin. Es decir nos aparece por pantalla "Interrupt Detached!".

<div>

## PRACTICA B: Interrupción por Timer

Ahora tenemos un bucle que detecta cada vez quién ha habido una interrupción y va mostrando por pantalla cuántas interrupciones lleva. Las interrupciones ocurren periódicamente sin fin según un timer previamente programado. El código es el siguiente:

```cpp
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
