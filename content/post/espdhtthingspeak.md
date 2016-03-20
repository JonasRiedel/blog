+++
date = "2015-08-08T22:13:07+01:00"
tags = ["esp8266","dht22"]
title = "ESP8266-01 MIT DHT22 UND THINGSPEAK ANBINDUNG"

+++

So, nachdem der ESP8266 generell funktioniert hat und sich per AT Befehl ansprechen ließ, wollte ich das Ding jetzt mal "richtig" programmieren.
Dazu die Arduino IDE entsprechend [dieser Anleitung](https://github.com/esp8266/Arduino) heruntergeladen und installiert.

Dann wollte ich die Temperatur und Feutchtigkeit mit einem DHT22 messen und an einen Server im Internet schicken um sie grafisch darzustellen. Temperaturmessung mit einem DHT ist für mich quasi das "Hello World" für neue Boards.

Die finale Schaltung mit dem DHT22 und dem ESP8266-01 sieht dann wie folgt aus:

![](/post/ESP8266_DHT22.png)

<!--more-->
Dann habe ich den Code von dieser Seite genommen und etwas angepasst:

```c
// Original Code from:
// |www.arduinesp.com
// |
// |Plot DTH11 data on thingspeak.com using an ESP8266
// |April 11 2015
// |Author: Jeroen Beemster
// |Website: www.arduinesp.com
// modified by Jonas Riedel 
// for use with DHT22

#include <DHT.h>
#include <ESP8266WiFi.h>
 
// replace with your channel's thingspeak API key,
String apiKey = "xxxxxxxxxxxxxxxx"; 
// enter your own Wlan SSID
const char* ssid = "xxxxx";
//your Wlan Password
const char* password = "xxxxxxxxx";
 
const char* server = "api.thingspeak.com";
#define DHTPIN 2 // On ESP8266 01 this is the GPIO2
 
DHT dht(DHTPIN, DHT22, 15); // Use DHT11 for a DHT11 Sensor :-)
WiFiClient client;
 
void setup() {
  Serial.begin(115200);
  delay(10);
  dht.begin();
 
  WiFi.begin(ssid, password);
 
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
 
  WiFi.begin(ssid, password);
 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print("."); // loop until Wlan connected
  }
  Serial.println("");
  Serial.println("WiFi connected");
 
}
 
void loop() {
 
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    delay(500);
    return;
  }
 
  if (client.connect(server,80)) {  
    String postStr = apiKey;
           postStr +="&field1=";
           postStr += String(t);
           postStr +="&field2=";
           postStr += String(h);
 
     client.print("POST /update HTTP/1.1\n");
     client.print("Host: api.thingspeak.com\n");
     client.print("Connection: close\n");
     client.print("X-THINGSPEAKAPIKEY: "+apiKey+"\n");
     client.print("Content-Type: application/x-www-form-urlencoded\n");
     client.print("Content-Length: ");
     client.print(postStr.length());
     client.print("\n\n");
     client.print(postStr);
 
     Serial.print("Temperature: ");
     Serial.print(t);
     Serial.print(" degrees Celcius Humidity: ");
     Serial.print(h);
     Serial.println("% send to Thingspeak");
  }
  client.stop();
 
  Serial.println("Waiting...");
  // thingspeak needs minimum 15 sec delay between updates but 60s is a good choice here
  delay(60000);
}
```
Entgegen diversen Webseiten, muss in der Arduino IDE als Board zwar "Generic ESP8266 Module" ausgewählt werden, aber was man als "Programmer" einstellt ist egal. Das Program muss dann mit dem Menüpunkt "Sketch->Hochladen" auf den ESP8266 geladen werden, dann wird automatisch der unter "Werkzeug" gewählte COM Port (also der USB Anschluss) genutzt. Es muss also nicht "esptool" als Programmer eingestellt sein !!

Der eigentliche Trick ist, das der Pin GPIO0 des ESP8266-01 während des programmierens auf GND liegen muss.

![](/post/ESP8266Flash.png)


Ich hab dazu einfach die USB Verbindung getrennt, dann den GPIO0 mit GND verbunden und dann wieder die USB Verbindung hergestellt. Dann in der Arduino IDE "Sketch->Hochladen" auswählen und der Code wird kompiliert und hochgeladen. Dabei blinkt der ESP.
Dann USB wieder trennen, GND von GPIO0 trennen und USB wieder ran. Dann sollte man über einen seriellen Monitor (ich nehme CoolTerm) die Ausgaben des ESPs sehen:

![](/post/cooltermdht22.png)


Ich musste dabei immer das DTR Signal abschalten bevor was angezeigt wurde. Dafür in CoolTerm unten einfach auf die DTR LED klicken (wenn die denn leuchtet)

Wenn alles soweit funktioniert, kann man auf [Thingspeak](https://thingspeak.com/) die Daten sehen:

![](/post/thingspeakdht22.png)



### Update vom 01.09.2015
Ich habe die Schaltung mal mit 4 AA (nicht AAA wie in dem Bild) betrieben:



Wenn das ganze dauerhaft mit dem WLan verbunden ist und alle 5min Daten sendet, hat die Batterie (Eneloop) bei mir ca 46h gehalten.