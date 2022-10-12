#include <ESP8266WiFi.h>
#include <SPI.h>
#include <Wire.h>
#include <DallasTemperature.h>
#include <OneWire.h>

#define ONE_WIRE_BUS 4 //D2

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

//#define DHTPIN D4;
//DHT dht(DHTPIN, DHT11);

//Credential for Cloud

String apiKey = "PXQS5GWO0L0YA6P7";
const char *ssid = "Redmi 4";
const char *pass = "123456789";
const char *server = "api.thingspeak.com";

 
//const int AirValue = 790;   //you need to replace this value with Value_1
//const int WaterValue = 390;  //you need to replace this value with Value_2
const int SensorPin = A0;
int soilMoistureValue = 0;
int soilmoisturepercent=0;
int relaypin = 13;
int soilTemp = 0;

WiFiClient client;
 
void setup() {
  Serial.begin(9600); // open serial port, set the baud rate to 9600 bps
  pinMode(relaypin, OUTPUT);
 
//dht.begin();

 WiFi.begin(ssid,pass);
 sensors.begin();

  while(WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
    }
    Serial.println("");
    Serial.println("WiFi connected");
    delay(4000);

}  
 
 
void loop() 
{
//  float h = dht.readHumidity();
//  float t = dht.readTemperature();
 
//  Serial.print("Humidity: ");
//  Serial.print(h,0);
//  Serial.println("%");
//  Serial.print("Temperature: ");
//  Serial.print(t,0);
//  Serial.println("%");
//  delay(1000);

  //Soil Moisture
  soilMoistureValue = analogRead(SensorPin);  //put Sensor insert into soil
  Serial.println(soilMoistureValue);
  
  soilmoisturepercent = ( 100.00 - ( (analogRead(SensorPin) / 1023.00) * 100.00 ) );
 
  Serial.print("soilmoisture : ");
  Serial.print(soilmoisturepercent);
  Serial.println("%");
  delay(1000);

 
  if(soilmoisturepercent < 35)
  {
    digitalWrite(13, HIGH);
    Serial.println("Motor is ON");
   Serial.println("-------------------");
  }
  else
  {
    digitalWrite(13, LOW);
    Serial.println("Motor is OFF");
   Serial.println("-------------------");
  } 
  delay(1000);
  
  //Soil Temperature
   sensors.requestTemperatures();
   Serial.print("Temperature is: ");
   soilTemp = sensors.getTempCByIndex(0);
   Serial.println(soilTemp);
   delay(1000);
  
  
  if (client.connect(server, 80)) // "184.106.153.149" or api.thingspeak.com
  {
    String postStr = apiKey;
      postStr += "&field1=";
      postStr += String(soilmoisturepercent);
      postStr += "&field2=";
      postStr += String(soilTemp);
      postStr += "&field3=";
      postStr += String(soilMoistureValue);
      postStr += "&field4=";
      postStr += String(13);
      postStr += "\r\n\r\n\r\n\r\n";
    
    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(postStr.length());
    client.print("\n\n");
    client.print(postStr);
   
  }
    client.stop();
}
