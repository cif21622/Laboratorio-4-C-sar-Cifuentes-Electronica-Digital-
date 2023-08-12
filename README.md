# Laboratorio-4-C-sar-Cifuentes-Electronica-Digital-
Laboratorio 4 

//César Cifuentes 
//21622
//Prelab Electronica digital 2, laboratorio 4 
//Se adjunta en etrega calculos realizados para colocar el servomotor en determinada posición 


#include "driver/ledc.h"
//Se definen los pines del botón de entrada 
const int ButtonPinIncrement = 18;
const int ButtonPinDecrement = 19;

//Se definen los estados para verificar el estado del bóton y hacer el incremento o decremento del ciclo de trabajo para cambiar las posiciones 
volatile bool incremento = true; 
volatile bool decremento = true;
//Se indica el donde inicia el ciclo de trabajo 
volatile int dutycicle = 75;
//Se definen los vectores de las interrupciones para los botones 
void IRAM_ATTR ISR_1() {
  incremento = false;
}

void IRAM_ATTR ISR_2() {
  decremento = false;
}

//Se define la estructura para almacenar la configuración del PWM 
ledc_channel_config_t channel_config;


void setup() {
  // put your setup code here, to run once:
//Pines para cambiar la posicion del servomotor 
  // Configurar los pines de los botones como entradas con resistencia de pull-up y que se active la interrupción cuando la señal esté en bajada. Se indica tambíen el vector de interrupción 
  pinMode(ButtonPinIncrement, INPUT_PULLUP);
  attachInterrupt(ButtonPinIncrement, ISR_1, FALLING);
  pinMode(ButtonPinDecrement, INPUT_PULLUP);
  attachInterrupt(ButtonPinDecrement, ISR_2, FALLING);


//Configuración del PWM 
  channel_config.gpio_num = 15;
  channel_config.speed_mode = LEDC_HIGH_SPEED_MODE;
  channel_config.channel = LEDC_CHANNEL_0;
  channel_config.intr_type = LEDC_INTR_DISABLE;
  channel_config.timer_sel = LEDC_TIMER_0;
  channel_config.duty = 75;

  // Paso 3: Asociar las configuraciones al canal de salida del PWM 
  ledc_channel_config(&channel_config);

// Paso 5: Declarar una estructura que contendrá las configuraciones del timer
  ledc_timer_config_t timer_config;

//Se usa una frecuencia de 50 HZ para tener un periodo de 20 ms y así ajustar el ciclo de trbajo para colocar en el servomotor los ángulos 
//Paso 6. Ingresar las configuraciones del timer
  timer_config.speed_mode = LEDC_HIGH_SPEED_MODE;
  timer_config.duty_resolution = LEDC_TIMER_10_BIT;
  timer_config.timer_num = LEDC_TIMER_0;
  timer_config.freq_hz = 50;

//Paso 7 Asociar las configuraciones al timer de la señal PWM
ledc_timer_config(&timer_config);


}

void loop() {
  // put your main code here, to run repeatedly:
   //Se establece el máximo valor al que puede llegar el servomotor 
  //Si se habilitó la interrucpicón de incrementar, se cambia el valor del ciclo de trabajo para aumentar 45 grados a la posción anterior
  // Incrementar o decrementar el contador según el botón presionado se detecta si hay cambios generados por la interrupción y se realiza el movimimiento del ciclo se trabajo 
    if (!incremento) {
    dutycicle = dutycicle +25;
    
      if (dutycicle >= 125) {   
        dutycicle = 125; 
        }
      else{   
        dutycicle = dutycicle; 
        }
    delay(200);
    incremento = true;
    }

    //Se establece el minimo valor al que puede llegar el servomotor 
  //Si se habilitó la interrucpicón de decrementar, se cambia el valor del ciclo de trabajo para disminuit 45 grados a la posción anterior
    // Incrementar o decrementar el contador según el botón presionado se detecta si hay cambios generados por la interrupción y se realiza la suma en el contador 
    if (!decremento) {
    dutycicle = dutycicle - 25;
      if (dutycicle <= 25) {   
        dutycicle = 25; 
        }
      else{   
        dutycicle = dutycicle; 
        }
    delay(200);
    decremento = true;
    }
//Se actualiza el ciclo de trabajo para poner el servomotor en la posición indicada 
ledc_set_duty(LEDC_HIGH_SPEED_MODE, LEDC_CHANNEL_0, dutycicle );
ledc_update_duty(LEDC_HIGH_SPEED_MODE, LEDC_CHANNEL_0);
delay(2000);

}
