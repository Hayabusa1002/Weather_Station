#include <Ubidots.h>
#include <Adafruit_BME280.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include "Adafruit_VEML6070.h"

const char* UBIDOTS_TOKEN = "BBFF-raxxqTUOVH79brYDIySzcouU5b4EtT";  // Put here your Ubidots TOKEN
const char* WIFI_SSID = "Redmi Note 8";                             // Put here your Wi-Fi SSID
const char* WIFI_PASS = "hayabusa";                                 // Put here your Wi-Fi password

#define SEALEVELPRESSURE_HPA (1013.25)
#define DHTTYPE DHT22                                               // DHT 22  (AM2302), AM2321

Ubidots ubidots(UBIDOTS_TOKEN, UBI_HTTP);

/* Sensor DHT */
const int DHTPin = 14;                                              //Comunicación de datos en el pin 5 (GPIO 5 -- D5)
// Iniciando sensor
DHT dht(DHTPin, DHTTYPE);

/* sensor BME280 */
Adafruit_BME280 bme;                                                // use I2C interface
float t2,h2,p2;

/* Medición de bateria */
int analogPin = A0;
float raw,voltage;

/* sensor UV */
Adafruit_VEML6070 uv = Adafruit_VEML6070();                         // use I2C interface

void setup(){
   Serial.begin(115200);
   ubidots.wifiConnect(WIFI_SSID, WIFI_PASS);
   unsigned status;
   status = bme.begin(0x76);  
   delay(10);
   dht.begin();
   uv.begin(VEML6070_1_T);   
}

void loop(){
  raw=analogRead(A0);
  voltage=raw/1023*100;
  float h = dht.readHumidity();
  float t = dht.readTemperature();
          
  t2=bme.readTemperature();
  h2=bme.readHumidity();
  p2=bme.readPressure()/100.0F;;
           
  if (isnan(h) || isnan(t)) {
    Serial.println("Fallos al leer el sensor DHT");  }
  
   
   ubidots.add("tem_s", t);
   ubidots.add("hum_s", h);
   ubidots.add("tem_a", t2);
   ubidots.add("hum_a", h2);
   ubidots.add("pre_a", p2);
   ubidots.add("UV",uv.readUV());
   ubidots.add("bateria",voltage); 
   bool bufferSent = false;
   bufferSent = ubidots.send();  // Will send data to a device label that matches the device Id
  
  
  if (bufferSent) {
    // Do something if values were sent properly
    Serial.println("Datos enviados por el dispositivo");
    Serial.println(t);
    Serial.println(t2);
    Serial.println(h);
    Serial.println(h2);
    Serial.println("voltaje");
    Serial.println(voltage);
    
  }
  else{
    Serial.println("Datos no enviados por el dispositivo");
  }

  delay(500);
}
