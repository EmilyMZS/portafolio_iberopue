# Actuadores Introducción

Los actuadores son dispositivos electrónicos y electromecánicos que convierten energía en movimiento o fuerza. Estos son muy útiles en términos de automatización de proyectos.

En el caso de lo que se aprendio a usar, se utilizaron mayormente motores DC y al final servomotores.

Algo muy importante de este tipo de motores es que funcionan con un cierto voltaje (12v el DC, y 5v los servo), y que tienen una caracteristica conocida como PWM.

El PWM (Modulación por Ancho de Pulsos,) es un control de la energía que un motor usa, o en general una gran cantidad de dispositivos de corriente continua. En el caso de los motores, la regulación de la energía permite un mejor control sobre el funcionamiento y velocidad con la que gira o se mueve el motor. Al variar el ciclo de trabajo (el porcentaje de tiempo que la señal está encendida), se puede controlar la velocidad y el comportamiento del motor de forma eficiente.

Tambien es importante saber que es la resolución, la cual se define como la precisión con la que se puede controlar la velocidad o posición del motor. Este funciona con bits, y por ejemplo, si al motor se le programa con 12 bits, significa que el motor tendra de 0-4096. Esto representa que el motor puede tener 4096 diferentes niveles en los que se mueva.

La diferencia principal entre PWM y Resolución es que PWM es un control enfocado a la energía, y la resolución es un control al mecanismo. El PWM controla cuanta energia recibe el motor, y la resolución que tan fino es el control del motor.

Podemos decir que la resolución define cuántos niveles puede tener el PWM, y el PWM determina cuánta energía se entrega en cada nivel.

## Motor DC

Con el motor DC nos enfocamos durante la practica a poder hacerlo girar de un lado a otro, y luego escalarlo para que luego regresara a el 0.

``` codigo

/*Control de 1 solo motor*/

define in1 27

define in2 14

void setup() {

  /*Declarar Pines Como salida*/

  pinMode(in1, OUTPUT);

  pinMode(in2, OUTPUT);

}


void loop() {

  /*ADELANTE*/

  
  digitalWrite(in1, 0);
  
  digitalWrite(in2, 1);
  
  delay(1000);

  
  /*ALTO*/

  
  digitalWrite(in1, 0);
  
  digitalWrite(in2, 0);
  
  delay(1000);

  
  /*ATRAS*/

  
  digitalWrite(in1, 1);
  
  digitalWrite(in2, 0);
  
  delay(1000);

  
  /*ALTO*/

  
  digitalWrite(in1, 0);
  
  digitalWrite(in2, 0);
  
  delay(1000);
  
}```

Este codigo basicamente nos esta mostrando que in1 es una dirección de giro e in2 es otra. Cuando in1 este activo, va a girar para delante, e in2 debe de estar en 0. Para que vaya en reversa, debe ser viceversa. 

Para que el motor pare, se ponen los dos in en 0 para que pare. Y se usan delays de acuerdo a lo necesario.

Como se puede ver, esto solo es de prendido y apagado sin velocidades determinadas.

Para que haya una velocidad en especifico, se define el PWM

- #define PWM 12 // pin de velocidad
- En voidSetup:
ledcAttachChannel(pwm, 1000, 8, 0); // ledcAttachChannel= comando del ESP32, pwm=pin=12, 1000 = frecuencia, 8 = resolucion = (2^8 = 225. Resolución de 0-225), 0= constante. Siempre debe ser 0.
