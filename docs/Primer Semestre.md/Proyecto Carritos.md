## 1) Resumen

- **Nombre del proyecto:** _Carritos con motores DC_  
- **Equipo / Autor(es):** _Emily Mendez, Aldo Fernandez, Alexandra Groot, Valeria Piña, Arturo Martínez, Sebastian Rodríguez, Erik Zepeda_  
- **Curso / Asignatura:** _Introducción a la Mecatrónica_  
- **Fecha:** _15/10/2025_  
- **Descripción breve:** _Desarrollo de un carrito a control remoto con bases de motores DC, puente H, ESP32, Control Bluetooth, etc._
  

## 2) Objetivos

- **General:** _Este proyecto busca la aplicación de los conocimientos adquiridos y practicados durante las clases y poder unirlo todo para crear un coche que compita con los de otros equipos._
- **Específicos:**
  - _Crear un sistema que permita el movimiento rápido y controlado del coche para poder atrapar una pelota y anotar en la portería de otro equipo_


## 3) Alcance y Exclusiones

- **Incluye:** _Crear un coche con materiales y diseños propios, sistema de movimiento dinámico, control remoto a través de conexiones Bluetooth en un plazo de dos semanas aproximadamente._


## 4) Planeación

Para poder desarrollar este proyecto, iniciamos dividiendo nuestro equipo en tareas. Dos se encargarían de elaborar los diseños para crear el coche y los demás hariamos trabajo de electrónica y programación para hacer el trabajo colaborativo, rápido y dinámico.

A continuación, hicimos una lista de los materiales que requeriríamos para desarollar el trabajo:

- Puente H
  
- Dos motores DC
  
- Una protoboard
  
- Un microcontrolador ESP32
- Jumpers
  
- Luces LED
  
- Pila de 9V

Junto con esto, hicimos uso de MDF para poder generar la estructura y diseño del coche, junto con PLA para poder imprimir en 3D una pala para empujar la pelota a la portería del equipo contrario.


## 5) Desarrollo

### Electrónica

En la parte de la electrónica, iniciamos el trabajo conectando los motores a las bases hechas con MDF y conectandolos al puente H, asegurandonos de que la corriente y ground estuvieran conectados apropiadamente. Junto a esto, el IN1, IN2, IN3, e IN4 fueron modificados y conectados para que los motores se movieran apropiadamente hacia delante y atras.

Despues, hicimos uso de las ESP32 para obtener las hojas de datos y hacer las conexiones a esta de manera de que no se creara corto y luego los pines usados fueran escritos en el código de programación.


### Programación

En términos de la programación, teniendo ya los pines conectados en la parte de electrónica, desarrollamos un código que se pudiera conectar via Bluetooth al celular de un compañero y de esta manera controlar el vehículo.

Entonces, creamos diferentes funciones, cada una para cada comando: Derecha, izquierda, adelante, atras, y stop. Se hizo uso del PWM para poder generar los giros del coche, asignando que cuando se gire a la izquierda, la llanta derecha se detendría, y la izquierda seguiría encendida, para que de esta manera el coche gire. Se genero lo inverso para el giro derecho. Para que fuera derecho, las dos llantas van a la misma velocidad, y con stop, ambas llantas estan en 0.

Añadido a esto, dentro del voidLoop se ingreso una función que pudiera controlar la velocidad de los motores de acuerdo al número que se ingresara como mensaje a la ESP.


``` codigo
#include "BluetoothSerial.h"
 
BluetoothSerial SerialBT;


// Pines del puente H
 
const int IN1 = 12; // Motor izquierdo
 
const int IN2 = 11;
 
const int ENA = 13;
 
const int IN3 = 10; // Motor derecho
 
const int IN4 = 9;
 
const int ENB = 7;
 
int valSpeed = 255;


void setup() {
 
  Serial.begin(115200);
 
  SerialBT.begin("CarroESP32"); // Nombre del dispositivo Bluetooth
 
  pinMode(IN1, OUTPUT);
 
  pinMode(IN2, OUTPUT);
 
  pinMode(ENA, OUTPUT);
 
  pinMode(IN3, OUTPUT);
 
  pinMode(IN4, OUTPUT);
 
  pinMode(ENB, OUTPUT);
 
  stopMotors();
 
}


void loop() {
 
  if (SerialBT.available()) {
 
    char command = SerialBT.read();
 
    Serial.println(command);
 
    switch (command) {
 
      case 'F': forward(); break;
 
      case 'B': backward(); break;
 
      case 'L': turnLeft(); break;
 
      case 'R': turnRight(); break;
 
      case 'S': stopMotors(); break;
 
      case '0': setSpeed(0); break;
 
      case '1': setSpeed(25); break;
 
      case '2': setSpeed(50); break;
 
      case '3': setSpeed(75); break;
 
      case '4': setSpeed(100); break;
 
      case '5': setSpeed(125); break;
 
      case '6': setSpeed(150); break;
 
      case '7': setSpeed(175); break;
 
      case '8': setSpeed(200); break;
 
      case '9': setSpeed(255); break;
 
    }
 
  }
 
}


void forward() {
 
  analogWrite(ENA, valSpeed);
 
  analogWrite(ENB, valSpeed);
 
  digitalWrite(IN1, HIGH);
 
  digitalWrite(IN2, LOW);
 
  digitalWrite(IN3, HIGH);
 
  digitalWrite(IN4, LOW);
 
}


void backward() {
 
  analogWrite(ENA, valSpeed);
 
  analogWrite(ENB, valSpeed);
 
  digitalWrite(IN1, LOW);
 
  digitalWrite(IN2, HIGH);
 
  digitalWrite(IN3, LOW);
 
  digitalWrite(IN4, HIGH);
 
}


void turnLeft() {
 
  analogWrite(ENA, valSpeed / 2);
 
  analogWrite(ENB, valSpeed);
 
  digitalWrite(IN1, HIGH);
 
  digitalWrite(IN2, LOW);
 
  digitalWrite(IN3, HIGH);
 
  digitalWrite(IN4, LOW);
 
}


void turnRight() {
 
  analogWrite(ENA, valSpeed);
 
  analogWrite(ENB, valSpeed / 2);
 
  digitalWrite(IN1, HIGH);
 
  digitalWrite(IN2, LOW);
 
  digitalWrite(IN3, HIGH);
 
  digitalWrite(IN4, LOW);
 
}


void stopMotors() {
 
  analogWrite(ENA, 0);
 
  analogWrite(ENB, 0);
 
  digitalWrite(IN1, LOW);
 
  digitalWrite(IN2, LOW);
 
  digitalWrite(IN3, LOW);
 
  digitalWrite(IN4, LOW);
 
}

 
void setSpeed(int val) {
 
  valSpeed = val;
 
}
```
