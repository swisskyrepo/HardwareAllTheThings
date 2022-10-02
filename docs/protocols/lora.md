# LoRa

### LoRa with Arduino on 868.1MHZ

[arduino-LoRa](https://github.com/sandeepmistry/arduino-LoRa)\
use 868.1MHZ with SpreadFactor 10

```c
#include <SPI.h>
#include <LoRa.h>

void setup() {
  Serial.begin(9600);
  while (!Serial);

  Serial.println("LoRa Receiver");

  if (!LoRa.begin(868.1E6)) {
    Serial.println("Starting LoRa failed!");
    while (1);
  }
  LoRa.setSpreadingFactor(10);
}

void onReceive(int packetSize) {

  Serial.print("packet recv\n");
  // read packet
  for (int i = 0; i < packetSize; i++) {
    Serial.print((char)LoRa.read());
  }
}

void loop() {
  LoRa.receive();
  LoRa.onReceive(onReceive); 
}
```

### Bruteforce all the EU frequencies and the SpreadFactor

```c
#include <SPI.h>
#include <LoRa.h>

float freq[5] = { 868.3E6, 868.5E6, 867.1E6, 867.5E6, 867.7E6, 867.9E6 }; 

void setup() {
  Serial.begin(9600);
  while (!Serial);

  Serial.println("LoRa Receiver");

  if (!LoRa.begin(868.1E6)) {
    Serial.println("Starting LoRa failed!");
    while (1);
  }
  LoRa.setSpreadingFactor(10);
}

void onReceive(int packetSize) {

  Serial.print("packet recv\n");
  // read packet
  for (int i = 0; i < packetSize; i++) {
    Serial.print((char)LoRa.read());
  }
}

void loop() {
  
  LoRa.receive();
  LoRa.onReceive(onReceive);
  delay(5000);
  While(1) {
    int i;
    for(i=0; i < 5 ; i++)
    {
      
      LoRa.setFrequency(freq[i]);
      int j;
      for(j=7; j <= 12; j++)
      {
      	
        // loop on spreading factor is finish, set new freq
        LoRa.setSpreadingFactor(i);
        delay(5000);
      }
    }
  }
}
```

### Display RSSI of the packet

> The Received Signal Strength Indication (RSSI) is the received signal power in milliwatts and is measured in dBm.

The RSSI is measured in dBm and is a negative value.\
The closer to 0 the better the signal is.

Typical LoRa RSSI values are:

* RSSI minimum = -120 dBm.
* If RSSI=-30dBm: signal is strong.
* If RSSI=-120dBm: signal is weak.

```c
#include <SPI.h>
#include <LoRa.h>

void setup() {
  Serial.begin(9600);
  while (!Serial);

  Serial.println("LoRa Receiver");

  if (!LoRa.begin(867.1E6)) {
    Serial.println("Starting LoRa failed!");
    while (1);
  }
     LoRa.setSpreadingFactor(8);
}

void onReceive(int packetSize) {
 Serial.print("packet recv\n");
 int rssi = LoRa.packetRssi();
 Serial.print(rssi);
}

void loop() {
  LoRa.receive();
  LoRa.onReceive(onReceive);
  delay(1000);
}
```
