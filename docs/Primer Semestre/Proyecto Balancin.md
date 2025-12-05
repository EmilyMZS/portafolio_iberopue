## 1) Resumen

- **Nombre del proyecto:** _Balancin con Reconocimiento de Camara_  
- **Equipo / Autor(es):** _Emily Mendez, Aldo Fernandez, Alexandra Groot, Valeria Piña_  
- **Curso / Asignatura:** _Introducción a la Mecatrónica_  
- **Fecha:** _24/11/2025_  
- **Descripción breve:** _Desarrollo de una plataforma con cámara que balancee una pelota, utilizando servomotores y el reconocimento de colores desde Python._
  

## 2) Objetivos

**General:**  
_Diseñar y construir un sistema que mantenga una pelota centrada sobre una plataforma utilizando visión por computadora y control PID._

**Específicos:**
- _Procesar la imagen de la cámara en Python para detectar la pelota._
  
- _Enviar los datos obtenidos al Arduino mediante Bluetooth._
  
- _Implementar control PID que ajuste los servomotores para estabilizar la plataforma._


## 3) Alcance y Exclusiones

- **Incluye:** _El proyecto contempla un mes de trabajo partiendo del código base proporcionado por el profesor, realizando modificaciones necesarias para adaptar el reconocimiento por cámara, la comunicación con Arduino y el control del mecanismo. Incluye también el diseño y fabricación del balancín y los soportes de servomotores._


## 4) Planeación
La planeación del proyecto se estructuró en tres etapas principales:

1. Ajustes al código base:
Modificación del código de Arduino para interpretar correctamente los comandos enviados desde Python.
2. Diseño mecánico en SolidWorks:
Creación de los soportes para servomotores y del punto de apoyo del centro, siendo estos impresas en 3D y cortadas en MDF.
3. Integración y calibración:
Ensamble completo y ajustes de los parámetros del PID para lograr un movimiento estable del balancín.


## 5) Desarrollo

### Diseño
El diseño del balancín se basó en un mecanismo previamente visto en un video, adaptándolo a las necesidades del proyecto. Se incorporaron dos soportes laterales para servomotores y un soporte central con esfera para permitir el movimiento en los ejes X y Y.
Las piezas que sostienen a los servomotores y la esfera de la mitad fueron impresas en 3D, y los demas materiales fueron cortados en MDF.

<img width="985" height="410" alt="piezas_3d" src="https://github.com/user-attachments/assets/5489b7a9-9b3e-4970-bf42-e44b323f2a7a" />


### Programación
El desarrollo del código incluye:

- Detección de la pelota mediante filtros HSV en Python.
- Cálculo del error respecto al centro de la imagen.
- Envío de instrucciones por Bluetooth hacia el Arduino.
- Implementación de un controlador PID en Arduino para mover los servos de acuerdo con el error.
- El sistema funciona de manera continua, permitiendo un ajuste constante de la plataforma para mantener la pelota dentro de la zona deseada.

### Código Arduino

```cpp
#include <BluetoothSerial.h>
#include <ESP32Servo.h>
 
BluetoothSerial SerialBT;
 
Servo servoX;
Servo servoY;
 
// Configuración de servos
// LÍMITES CORREGIDOS: X toma los límites que tenía Y y viceversa
const int MIN_ANG_X = 120;   // Servo X mínimo
const int MAX_ANG_X = 30;    // Servo X máximo
const int CENTER_X = 100;    // Centro aproximado
 
const int MIN_ANG_Y = 100;   // Servo Y mínimo
const int MAX_ANG_Y = 10;    // Servo Y máximo
const int CENTER_Y = 85;     // Centro aproximado
 
// PID más rápido
float Kp = 1.0;  
float Ki = 0.01;
float Kd = 0.2;
 
// Variables X  
int eX = 0;
float integralX = 0;
float prevErrorX = 0;
float posX = CENTER_X;
 
// Variables Y
int eY = 0;
float integralY = 0;
float prevErrorY = 0;
float posY = CENTER_Y;
 
// Suavizado
float maxStep = 3.0;  // ligeramente mayor para movimientos más rápidos
 
unsigned long lastTime = 0;
 
void setup() {
  Serial.begin(115200);
  SerialBT.begin("ESPbalancin");
 
  servoX.attach(27);
  servoY.attach(14);
  servoX.write(posX);
  servoY.write(posY);
 
  lastTime = millis();
  Serial.println("PID rápido con límites X/Y intercambiados listo");
}
 
void loop() {
  unsigned long now = millis();
  float dt = (now - lastTime) / 1000.0;
  if (dt <= 0) dt = 0.001;
 
  // Leer errores de Bluetooth
  while (SerialBT.available()) {
    String cmd = SerialBT.readStringUntil('\n');
    cmd.trim();
    if (cmd.startsWith("EX:")) eX = cmd.substring(3).toInt();
    else if (cmd.startsWith("EY:")) eY = cmd.substring(3).toInt();
  }
 
  // ===== PID X =====
  float errorX_PID = -eX;  // invertir X
  integralX += errorX_PID * dt;
  float derivativeX = (errorX_PID - prevErrorX) / dt;
  float outputX = Kp * errorX_PID + Ki * integralX + Kd * derivativeX;
  float targetX = CENTER_X + outputX;
 
  // Suavizado
  float diffX = targetX - posX;
  if (abs(diffX) > maxStep) posX += (diffX > 0 ? maxStep : -maxStep);
  else posX = targetX;
 
  posX = constrain(posX, MIN_ANG_X, MAX_ANG_X);
  prevErrorX = errorX_PID;
 
  // ===== PID Y =====
  float errorY_PID = -eY;  
  integralY += errorY_PID * dt;
  float derivativeY = (errorY_PID - prevErrorY) / dt;
  float outputY = Kp * errorY_PID + Ki * integralY + Kd * derivativeY;
  float targetY = CENTER_Y + outputY;
 
  float diffY = targetY - posY;
  if (abs(diffY) > maxStep) posY += (diffY > 0 ? maxStep : -maxStep);
  else posY = targetY;
 
  posY = constrain(posY, MIN_ANG_Y, MAX_ANG_Y);
  prevErrorY = errorY_PID;
 
  // Mover servos
  servoX.write(posX);
  servoY.write(posY);
 
  lastTime = now;
 
  Serial.printf("EX=%d EY=%d | posX=%.1f posY=%.1f\n", eX, eY, posX, posY);
  delay(20);
}
```

---

### Código Python

```python
import cv2
import numpy as np
import bluetooth
import time
 
# Configuración Bluetooth
port = 1
sock = bluetooth.BluetoothSocket()
sock.settimeout(15)
 
print("Conectando a ESP32...")
while True:
    try:
        sock.connect(("6C:C8:40:4D:AE:E6", port))  # Cambia por tu MAC
        print("✔ Conectado a ESP32.")
        break
    except Exception as e:
        print("Reintentando conexión:", e)
    time.sleep(1)
 
video = cv2.VideoCapture(1)
time.sleep(0.5)
 
while True:
    ret, frame = video.read()
    if not ret:
        break
 
    # Rotar 90 grados a la derecha
    frame = cv2.rotate(frame, cv2.ROTATE_90_CLOCKWISE)
 
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
 
    low = np.array([100, 60, 40], dtype=np.uint8)
    high = np.array([130, 255, 255], dtype=np.uint8)
    mask = cv2.inRange(hsv, low, high)
 
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
 
    centrox = frame.shape[1] // 2
    centroy = frame.shape[0] // 2
 
    # Dibuja centro de cámara (verde)
    cv2.circle(frame, (centrox, centroy), 5, (0, 255, 0), 2)
 
    if contours:
        cont = max(contours, key=cv2.contourArea)
        (x, y), radius = cv2.minEnclosingCircle(cont)
        x, y = int(x), int(y)
 
        # Dibuja centro de la pelota (rojo)
        cv2.circle(frame, (x, y), 5, (0, 0, 255), -1)
        cv2.circle(frame, (x, y), int(radius), (0, 255, 255), 2)
 
        ErrorX_raw = x - centrox
        ErrorY_raw = y - centroy
 
        # Ajuste por rotación 90° y control diagonal invertido
        ErrorX = -ErrorY_raw  # invertir eje X
        ErrorY = ErrorX_raw   # invertir eje Y
 
        # Envía errores
        try:
            sock.send(f"EX:{ErrorX}\n".encode())
            sock.send(f"EY:{ErrorY}\n".encode())
        except:
            pass
 
        cv2.putText(frame, f"EX:{ErrorX} EY:{ErrorY}", (20, 40),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
 
    cv2.imshow("Camara", frame)
    cv2.imshow("Mascara", mask)
 
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break  
 
video.release()
cv2.destroyAllWindows()
sock.close()
 
```

---

## 6) Resultados y Evidencias
![proceso_ensamble](https://github.com/user-attachments/assets/ba9a773c-0918-44b5-9a38-01a426a78a98)
![evidencia3](https://github.com/user-attachments/assets/5736d348-d15d-458e-adc0-49a28f037a71)
![evidencia2](https://github.com/user-attachments/assets/54e86df5-84f3-45de-8d70-1219034f979f)
![evidencia1](https://github.com/user-attachments/assets/dd0dc09c-08f2-454c-a2c5-29f90be5dd60)

## 7) Conclusiones
El proyecto permitió poner en practica los conceptos de mecatrónica, combinando diseño, control y visión por computadora. El sistema final demuestra cómo el uso de reconocimiento por cámara y control PID puede aplicarse a un mecanismo basico como balancear una pelota, a sistemas mas complejos que permiten que un proyecto funcione de manera apropiada.

También se destaca la importancia de una buena organización y tiempos de trabajo definidos para lograr un proyecto exitoso.
