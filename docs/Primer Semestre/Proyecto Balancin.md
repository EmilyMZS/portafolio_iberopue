## 1) Resumen

- **Nombre del proyecto:** _Balancin con Reconocimiento de Camara_  
- **Equipo / Autor(es):** _Emily Mendez, Aldo Fernandez, Alexandra Groot, Valeria Piña_  
- **Curso / Asignatura:** _Introducción a la Mecatrónica_  
- **Fecha:** _24/11/2025_  
- **Descripción breve:** _Desarrollo de una plataforma con cámara que balancee una pelota, utilizando servomotores y el reconocimento de colores desde Python._
  

## 2) Objetivos

**General:**  
_Aplicar los conocimientos vistos en clase para diseñar y construir una plataforma que logre balancear una pelota sin que esta caiga, utilizando reconocimiento de colores y control PID._

**Específicos:**

- _Generar el programa que lea la información de la cámara desde Python al detectar la pelota._
- _Enviar esa información al Arduino para mover los servos según la posición._
- _Evitar que la pelota se caiga mediante controladores PID en Arduino._


## 3) Alcance y Exclusiones

- **Incluye:** _Durante un mes, partiendo de códigos proporcionados por el profesor, modificarlos e integrarlos al modelo de balanceo, junto con la construcción del balancín y el uso de servos._


## 4) Planeación



## 5) Desarrollo

### Electrónica
(Descripción aquí…)

### Programación
(Descripción aquí…)

---

### Código Arduino

```cpp
#include <BluetoothSerial.h>
#include <ESP32Servo.h>

BluetoothSerial SerialBT;

Servo servoX;
Servo servoY;

int posX = 90;
int posY = 90;
int paso = 3;

float Kp = 1.0;
float Ki = 0.5;
float Kd = 0.1;

double errorY = 0;
double errorY_prev = 0;
double integralY = 0;
double derY = 0;

unsigned long t_prev = 0;
double dt = 0;

bool PID_enabled = false;

void setup() {
  Serial.begin(115200);
  SerialBT.begin("ESPbalancin");

  Serial.println("Bluetooth listo. Esperando comandos...");

  servoX.attach(27);
  servoY.attach(14);

  servoX.write(posX);
  servoY.write(posY);

  t_prev = millis();
}

void loop() {

  if (SerialBT.available()) {
    String cmd = SerialBT.readStringUntil('\n');
    cmd.trim();
    Serial.println(cmd);

    if (cmd == "Derecha") posX += paso;
    else if (cmd == "Izquierda") posX -= paso;
    else if (cmd == "X_OK") posX = 90;

    else if (cmd == "Arriba") { posY -= paso; PID_enabled = false; }
    else if (cmd == "Abajo") { posY += paso; PID_enabled = false; }
    else if (cmd == "Y_OK") { posY = 90; PID_enabled = false; }

    posX = constrain(posX, 0, 180);
    posY = constrain(posY, 0, 180);

    servoX.write(posX);
    servoY.write(posY);
  }

  if (PID_enabled) {
    unsigned long t_now = millis();
    dt = (t_now - t_prev) / 1000.0;
    if (dt <= 0) dt = 0.001;

    integralY += errorY * dt;
    derY = (errorY - errorY_prev) / dt;

    float u = Kp * errorY + Ki * integralY + Kd * derY;

    posY = 90 + (int)(u * 0.05);
    posY = constrain(posY, 0, 180);
    servoY.write(posY);

    errorY_prev = errorY;
    t_prev = t_now;
  }

  Serial.print("X=");
  Serial.print(posX);
  Serial.print("  Y=");
  Serial.print(posY);
  Serial.print("  ErrY=");
  Serial.println(errorY);

  delay(10);
}
```

---

### Código Python

```python
import cv2
import numpy as np
import bluetooth
import time

# FUNCTION TO CONNECT TO ESP32
port = 1
sock = bluetooth.BluetoothSocket()
sock.settimeout(20)

print("Attempting to connect to ESP32...")
while True:
    try:
        sock.connect(("6C:C8:40:4D:AE:E6", port))
        print("Connected to ESP32!")
        break
    except Exception as e:
        print("Error in connection... retrying:", e)
    time.sleep(1)

video = cv2.VideoCapture(0)

while True:
    ret, frame_bgr = video.read()
    cv2.flip(frame_bgr, 1, frame_bgr)
    if not ret:
        break

    hsv = cv2.cvtColor(frame_bgr, cv2.COLOR_BGR2HSV)

    low  = np.array([100, 60, 40], dtype=np.uint8)
    high = np.array([130, 255, 255], dtype=np.uint8)
    mask = cv2.inRange(hsv, low, high)

    seg = cv2.bitwise_and(frame_bgr, frame_bgr, mask=mask)

    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    area_mayor = 0
    for actual in contours:
        area = cv2.contourArea(actual)
        if area > area_mayor:
            area_mayor = area
            contorno_mayor = actual

    cv2.drawContours(frame_bgr, contorno_mayor, -1, (0,255,0), 2)

    (x,y), radius = cv2.minEnclosingCircle(contorno_mayor)
    cv2.circle(frame_bgr, (int(x), int(y)), int(radius), (0,255,255), 2)
    cv2.circle(frame_bgr, (int(x), int(y)), 2 , (0,0,255), 2)

    print(x, y)

    centrox = frame_bgr.shape[1] // 2
    centroy = frame_bgr.shape[0] // 2
    ErrorX = int(x) - centrox
    ErrorY = int(y) - centroy

    if ErrorX > 0:
        direccionX ="Derecha\n"
        sock.send(direccionX.encode())
    elif ErrorX < 0:
        direccionX ="Izquierda\n"
        sock.send(direccionX.encode())
    else:
        direccionX= "X_OK\n"
        sock.send(direccionX.encode())

    if ErrorY > 0:
        direccionY ="Abajo\n"
        sock.send(direccionY.encode())
    elif ErrorY < 0:
        direccionY ="Arriba\n"
        sock.send(direccionY.encode())
    else:
        direccionY= "Y_OK\n"
        sock.send(direccionY.encode())

    cv2.imshow("Original", frame_bgr)
    cv2.imshow("Mask", mask)
    cv2.imshow("Segmentado", seg)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

video.release()
cv2.destroyAllWindows()
```

---

## 6) Resultados y Evidencias
(Agregar fotos, videos, gráficas…)

## 7) Conclusiones
(Conclusiones del proyecto)
