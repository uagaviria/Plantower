# Librería PMS 
Biblioteca Arduino para sensores Plantower PMS.
Admite sensores PMS x003 (1003, 3003, 5003, 6003, 7003).
## Instalación
Simplemente use Arduino Library Manager y busque "PMS Library" en la categoría Sensores.
## Suposiciones principales
- lo más fácil posible,
- consumo mínimo de memoria,
- funciones sin bloqueo,
- admitir una amplia gama de sensores PMS de Plantower,
- admitir modos adicionales, p. ej .: dormir, pasivo, activo (depende del modelo del sensor).

Como fuente de datos puede usar ** cualquier objeto ** que implemente la ** clase Stream **, como Wire, Serial, EthernetClient, e.t.c.
## Ejemplo Basico
Leer en modo activo.
> El modo predeterminado está activo después del encendido. En este modo, el sensor enviará datos seriales al host automáticamente. El modo activo se divide en dos submodos: modo estable y modo rápido. Si el cambio de concentración es pequeño, el sensor se ejecutará en modo estable con el intervalo real de 2,3 s. Y si el cambio es grande, el sensor se cambiará automáticamente a modo rápido con un intervalo de 200 ~ 800 ms, cuanto mayor sea la concentración, menor será el intervalo.
```cpp
#include "PMS.h"

PMS pms(Serial);
PMS::DATA data;

void setup()
{
  Serial.begin(9600);   // GPIO1, GPIO3 (TX/RX pin on ESP-12E Development Board)
  Serial1.begin(9600);  // GPIO2 (D4 pin on ESP-12E Development Board)
}

void loop()
{
  if (pms.read(data))
  {
    Serial1.print("PM 1.0 (ug/m3): ");
    Serial1.println(data.PM_AE_UG_1_0);

    Serial1.print("PM 2.5 (ug/m3): ");
    Serial1.println(data.PM_AE_UG_2_5);

    Serial1.print("PM 10.0 (ug/m3): ");
    Serial1.println(data.PM_AE_UG_10_0);

    Serial1.println();
  }

  // Hacer otras cosas...
}
```
## Output
```
PM 1.0 (ug/m3): 13
PM 2.5 (ug/m3): 18
PM 10.0 (ug/m3): 23

PM 1.0 (ug/m3): 12
PM 2.5 (ug/m3): 19
PM 10.0 (ug/m3): 24

...
```
## Ejemplo:

```cpp
#include "PMS.h"

PMS pms(Serial);
PMS::DATA data;

void setup()
{
  Serial.begin(9600);   // GPIO1, GPIO3 (TX/RX pin on ESP-12E Development Board)
  Serial1.begin(9600);  // GPIO2 (D4 pin on ESP-12E Development Board)
  pms.passiveMode();    // Switch to passive mode
}

void loop()
{
  Serial1.println("Despertarse, esperar 30 segundos para lecturas estables...");
  pms.wakeUp();
  delay(30000);

  Serial1.println("Enviar solicitud de lectura...");
  pms.requestRead();

  Serial1.println("Leyendo datos...");
  if (pms.readUntil(data))
  {
    Serial1.print("PM 1.0 (ug/m3): ");
    Serial1.println(data.PM_AE_UG_1_0);

    Serial1.print("PM 2.5 (ug/m3): ");
    Serial1.println(data.PM_AE_UG_2_5);

    Serial1.print("PM 10.0 (ug/m3): ");
    Serial1.println(data.PM_AE_UG_10_0);
  }
  else
  {
    Serial1.println("No data.");
  }

  Serial1.println("Going to sleep for 60 seconds.");
  pms.sleep();
  delay(60000);
}
```
## Output
```
Despertarse, esperar 30 segundos para lecturas estables ...
Enviar solicitud de lectura ...
Leyendo datos ...
PM 1.0 (ug / m3): 13
PM 2.5 (ug / m3): 18
PM 10.0 (ug / m3): 23
Dormir por 60 segundos.
Despertarse, esperar 30 segundos para lecturas estables ...
Enviar solicitud de lectura ...
Leyendo datos ...
PM 1.0 (ug / m3): 12
PM 2.5 (ug / m3): 19
PM 10.0 (ug / m3): 24
Dormir por 60 segundos.

...
```
## Observaciones
Probado con PMS 7003 y ESP-12E. Todos los sensores de Plantower PMS usan el mismo protocolo.
Tenga en cuenta que la función de retraso () en los ejemplos es una función de bloqueo.
Intente evitar tal solución si su proyecto lo requiere (vea el ejemplo de Expert.ino en el directorio de ejemplos).
Para mediciones más precisas, puede leer varias muestras (en modo pasivo o activo) y calcule el promedio.
Los datos estables deben obtenerse al menos 30 segundos después de que el sensor se active desde el modo de suspensión debido al rendimiento del ventilador.
