# PRACTICA 3 : WIFI  y BLUETOOTH  

El objetivo de la practica es comprender el funcionamiento de WIFI Y BT.

Para lo cual realizaremos una practica  donde  generaremos un web server desde utilizando 
nuestra ESP32  y tambien  una comunicacion  serie con una aplicacion de un movil con BT .



## Introducción teórica  

Si bien el objetivo de esta asignatura es el manejo de los microcontroladores  y microprocesadores .
Es muy complicado que podamos explicar el uso de periféricos  especializados WIFI o Bluetooth sin tener idea de 
cual es  el fundamento en que se basan . Por lo que daremos algunas referencias básicas  que les servirán para
 entender  los conceptos de la practica  ; si bien su comprensión la realizaran en otras asignaturas de redes.


Con respecto a la WIFI

 Por una parte recomendamos la lectura  de  protocolos TCP/IP  UDP   

https://www.tlm.unavarra.es/~daniel/docencia/lir/lir05_06/slides/1-Conceptosbasicos.pdf

Por otra parte   wifi

http://www.radiocomunicaciones.net/pdf/curso-iniciacion-wifi.pdf


Por otro  la API REST 

https://aprendiendoarduino.wordpress.com/2019/10/27/api-rest/


Y  por ultimo MQTT

https://ricveal.com/blog/primeros-pasos-mqtt


Con respecto a Bluetooth 

https://randomnerdtutorials.com/esp32-bluetooth-classic-arduino-ide/






## Practica A generación de una pagina web  

No se requiere montaje  

Ejemplo de código :


 ```

/*
  ESP32 Web Server - STA Mode
  modified on 25 MAy 2019
  by Mohammadreza Akbari @ Electropeak
  https://electropeak.com/learn
*/

#include <WiFi.h>
#include <WebServer.h>

// SSID & Password
const char* ssid = "*****";  // Enter your SSID here
const char* password = "*****";  //Enter your Password here

WebServer server(80);  // Object of WebServer(HTTP port, 80 is defult)

void setup() {
  Serial.begin(115200);
  Serial.println("Try Connecting to ");
  Serial.println(ssid);

  // Connect to your wi-fi modem
  WiFi.begin(ssid, password);

  // Check wi-fi is connected to wi-fi network
  while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected successfully");
  Serial.print("Got IP: ");
  Serial.println(WiFi.localIP());  //Show ESP32 IP on serial

  server.on("/", handle_root);

  server.begin();
  Serial.println("HTTP server started");
  delay(100); 
}

void loop() {
  server.handleClient();
}

// HTML & CSS contents which display on web server
String HTML = "<!DOCTYPE html>\
<html>\
<body>\
<h1>My Primera Pagina con ESP32 - Station Mode &#128522;</h1>\
</body>\
</html>";

// Handle root url (/)
void handle_root() {
  server.send(200, "text/html", HTML);
}
```


### informe
  
  1. Realizar el informe de funcionamiento  asi como la salida por el terminal  y la  visualización de la conexión a la pagina web con un navegador
  2. Modificar la pagina web generando un fichero HTML con  el editor Mark down e incluirlo en el codigo . Incluir un fichero adicional que solo incluya la pagina html 
  3. Generar informe  y codigo  en Github
   
   
   


## Practica B  comunicación bluetooth con el movil 

El código de la practica es el siguiente


```
//This example code is in the Public Domain (or CC0 licensed, at your option.)
//By Evandro Copercini - 2018
//
//This example creates a bridge between Serial and Classical Bluetooth (SPP)
//and also demonstrate that SerialBT have the same functionalities of a normal Serial

#include "BluetoothSerial.h"

#if !defined(CONFIG_BT_ENABLED) || !defined(CONFIG_BLUEDROID_ENABLED)
#error Bluetooth is not enabled! Please run `make menuconfig` to and enable it
#endif

BluetoothSerial SerialBT;

void setup() {
  Serial.begin(115200);
  SerialBT.begin("ESP32test"); //Bluetooth device name
  Serial.println("The device started, now you can pair it with bluetooth!");
}

void loop() {
  if (Serial.available()) {
    SerialBT.write(Serial.read());
  }
  if (SerialBT.available()) {
    Serial.write(SerialBT.read());
  }
  delay(20);
}


```

Utilizar  la siguiente aplicación para realizar la comunicación serie 

![](https://i2.wp.com/randomnerdtutorials.com/wp-content/uploads/2019/05/Bluetooth_Serial_app.png?w=500&quality=100&strip=all&ssl=1)



### informe
  realizar el informe de funcionamiento  asi como las salidas que se obtienen a través de la impresión serie 





## Ejercicios de mejora de nota 

1. Realizar el mismo ejercicio Wifi pero en lugar de realizar la conexión STA hacer una conexión AP
2. BLE ( bluetooth low energy) revisar e implementar el código del siguiente tutorial 
     https://randomnerdtutorials.com/esp32-bluetooth-low-energy-ble-arduino-ide/