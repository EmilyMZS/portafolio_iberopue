# Actuadores Introducción

Los actuadores son dispositivos electrónicos y electromecánicos que convierten energía en movimiento o fuerza. Estos son muy útiles en términos de automatización de proyectos.

En el caso de lo que se aprendio a usar, se utilizaron mayormente motores DC y al final servomotores.

Algo muy importante de este tipo de motores es que funcionan con un cierto voltaje (12v el DC, y 5v los servo), y que tienen una caracteristica conocida como PWM.

El PWM (Modulación por Ancho de Pulsos,) es un control de la energía que un motor usa, o en general una gran cantidad de dispositivos de corriente continua. En el caso de los motores, la regulación de la energía permite un mejor control sobre el funcionamiento y velocidad con la que gira o se mueve el motor.

Tambien es importante saber que es la resolución, la cual se define como la precisión con la que se puede controlar la velocidad o posición del motor. Este funciona con bits por segundo, y por ejemplo, si al motor se le programa con 12 bits, significa que el motor tendra de 0-4096. Esto representa que el motor puede tener 4096 diferentes niveles en los que se mueva.

La diferencia principal entre PWM y Resolución es que PWM es un control enfocado a la energía, y la resolución es un control al mecanismo. El PWM controla cuanta energia recibe el motor, y la resolución que tan fino es el control del motor.
