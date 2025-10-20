SAFE PINS ONE CAN USE (when camera is active): On the ESP32 WroverBoard
These GPIO pins are completely FREE and safe:

GPIO13
GPIO14
GPIO15 (minor caveat: affects boot messages, but safe to use)
GPIO16
GPIO17
GPIO32
GPIO33

❌ PINS YOU CANNOT USE (already taken by camera):

GPIO4, 5, 18, 19, 21, 22, 23, 25, 26, 27, 34, 35, 36, 39

❌ PINS TO AVOID (USB programming):

GPIO1 (TX), GPIO3 (RX)

❌ PINS TO AVOID (internal flash - will crash board):

GPIO6, 7, 8, 9, 10, 11


RECOMMENDATION FOR I2C WITH ARDUINO:
Use GPIO13 and GPIO16:
Wiring:
ESP32 GPIO13  →  Arduino A4 (SDA)
ESP32 GPIO16  →  Arduino A5 (SCL)  
ESP32 GND     →  Arduino GND

ESP32 Code:
cppWire.begin(13, 16); // SDA=13, SCL=16

**Sample code**
Arduino
~~~C
#include <Wire.h>

#define I2C_ADDRESS 8  // Slave address

String receivedData = "";

void setup() {
  Serial.begin(9600);  // For debugging
  Wire.begin(I2C_ADDRESS);  // Join I2C bus as slave
  Wire.onReceive(receiveEvent);  // Register receive callback
  Wire.onRequest(requestEvent);  // Register request callback
  
  Serial.println("Arduino I2C Slave Ready");
}

void loop() {
  // Your main code here
  // The I2C communication happens in the callbacks
  delay(100);
}

// Called when master sends data TO Arduino
void receiveEvent(int numBytes) {
  receivedData = "";  // Clear previous data
  
  while (Wire.available()) {
    char c = Wire.read();
    receivedData += c;
  }
  
  Serial.print("Received: ");
  Serial.println(receivedData);
  
  // Do something with the data
  if (receivedData == "LED_ON") {
    digitalWrite(LED_BUILTIN, HIGH);
  } else if (receivedData == "LED_OFF") {
    digitalWrite(LED_BUILTIN, LOW);
  }
}

// Called when master requests data FROM Arduino
void requestEvent() {
  // Send some data back to master
  String response = "Hello from Arduino!";
  Wire.write(response.c_str());
  
  Serial.println("Sent: " + response);
}
~~~

ESP32
~~~C
#include <Wire.h>

#define I2C_SDA 13
#define I2C_SCL 16
#define SLAVE_ADDRESS 8

void setup() {
  Serial.begin(115200);
  Wire.begin(I2C_SDA, I2C_SCL);  // Initialize I2C with custom pins
  Wire.setClock(100000);  // 100kHz - reliable speed
  
  Serial.println("ESP32 I2C Master Ready");
  delay(1000);
}

void loop() {
  // Example 1: Send data TO Arduino
  sendDataToSlave("LED_ON");
  delay(2000);
  
  // Example 2: Request data FROM Arduino
  String response = requestDataFromSlave();
  Serial.print("Arduino replied: ");
  Serial.println(response);
  delay(2000);
  
  // Turn LED off
  sendDataToSlave("LED_OFF");
  delay(2000);
}

// Function to send data to Arduino
void sendDataToSlave(String data) {
  Wire.beginTransmission(SLAVE_ADDRESS);
  Wire.write(data.c_str());  // Send string
  byte error = Wire.endTransmission();
  
  if (error == 0) {
    Serial.println("Sent to Arduino: " + data);
  } else {
    Serial.println("Error sending data: " + String(error));
  }
}

// Function to request data from Arduino
String requestDataFromSlave() {
  String received = "";
  
  Wire.requestFrom(SLAVE_ADDRESS, 20);  // Request 20 bytes
  
  while (Wire.available()) {
    char c = Wire.read();
    received += c;
  }
  
  return received;
}
~~~
