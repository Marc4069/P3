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


### Codigo
  
 
#include <Arduino.h>
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

// Define UUIDs for the BLE service and characteristic
#define SERVICE_UUID        "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHARACTERISTIC_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"

// Create pointers for BLE objects
BLEServer* pServer = NULL;
BLECharacteristic* pCharacteristic = NULL;
bool deviceConnected = false;
bool oldDeviceConnected = false;

// Variables for sensor data (example)
float temperature = 0;
unsigned long previousMillis = 0;
const long interval = 2000;  // Update interval in milliseconds

// Callback class for handling server events
class MyServerCallbacks: public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) {
      deviceConnected = true;
      Serial.println("Device connected");
    }

    void onDisconnect(BLEServer* pServer) {
      deviceConnected = false;
      Serial.println("Device disconnected");
    }
};

// Callback for handling characteristic events
class CharacteristicCallbacks: public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
      std::string value = pCharacteristic->getValue();
      
      if (value.length() > 0) {
        Serial.println("Received data: ");
        for (int i = 0; i < value.length(); i++) {
          Serial.print(value[i]);
        }
        Serial.println();
      }
    }
};

void setup() {
  Serial.begin(9600);
  Serial.println("Starting BLE application");

  // Initialize BLE device
  BLEDevice::init("ESP32-S3 BLE Device");
  
  // Create the BLE Server
  pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  // Create the BLE Service
  BLEService *pService = pServer->createService(SERVICE_UUID);

  // Create a BLE Characteristic
  pCharacteristic = pService->createCharacteristic(
                      CHARACTERISTIC_UUID,
                      BLECharacteristic::PROPERTY_READ   |
                      BLECharacteristic::PROPERTY_WRITE  |
                      BLECharacteristic::PROPERTY_NOTIFY |
                      BLECharacteristic::PROPERTY_INDICATE
                    );

  // Add the descriptor for notifications
  pCharacteristic->addDescriptor(new BLE2902());
  
  // Set callback to handle write events
  pCharacteristic->setCallbacks(new CharacteristicCallbacks());

  // Start the service
  pService->start();

  // Start advertising
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID(SERVICE_UUID);
  pAdvertising->setScanResponse(true);
  pAdvertising->setMinPreferred(0x06);  // helps with iPhone connections issue
  pAdvertising->setMinPreferred(0x12);
  BLEDevice::startAdvertising();
  
  Serial.println("BLE service started. Waiting for a client connection...");
}

void loop() {
  unsigned long currentMillis = millis();
  
  // Simulate sensor reading and update value periodically
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    
    // Simulate temperature change
    temperature = random(2000, 3000) / 100.0;
    
    // If a device is connected, update the characteristic value
    if (deviceConnected) {
      String tempStr = String(temperature, 2);
      pCharacteristic->setValue(tempStr.c_str());
      pCharacteristic->notify();
      Serial.print("Temperature: ");
      Serial.println(tempStr);
    }
  }

  // Handle connection events
  if (!deviceConnected && oldDeviceConnected) {
    delay(500); // Give the Bluetooth stack time to get ready
    pServer->startAdvertising(); // Restart advertising
    Serial.println("Started advertising again");
    oldDeviceConnected = deviceConnected;
  }
  
  // If we've connected to a device
  if (deviceConnected && !oldDeviceConnected) {
    oldDeviceConnected = deviceConnected;
  }
  
  delay(10); // Small delay for stability
}

   
   


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



