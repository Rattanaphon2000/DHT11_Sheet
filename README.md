# DHT11_Sheet
/* Record  temerature and relative humidity from DHT11 sensor to google sheet by IFTTT */
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClient.h>
//DHT sensor
#include"DHT.h"
#define D4 2 // DHT Digital pin
#define DHTPIN D4 // DHT Digital pin 
#define DHTTYPE DHT11 // DHT 11
DHT dht(DHTPIN,DHTTYPE);

const char* ssid = "TOT_GOOGG";
const char* password = "GaiSaeb001";
const char* Location = "Innovation building";
int counter =0; //
float temp,humid; //

// Domain Name with full URL Path for HTTP POST Request
// REPLACE WITH YOUR EVENT NAME AND API KEY - open the documentation: https://ifttt.com/maker_webhooks
const char* serverName = "http://maker.ifttt.com/trigger/DHT11GIF/with/key/kH2FrH2fFUB63ImnfUWtEN3WrMtqQrzTZLtrCsWzYpL";
// Example:
//const char* serverName = "http://maker.ifttt.com/trigger/Wichuta/with/key/byw0k1ssHvGGT6YCDsHUsi";

// THE DEFAULT TIMER IS SET TO 10 SECONDS FOR TESTING PURPOSES
// For a final application, check the API call limits per hour/minute to avoid getting blocked/banned
unsigned long lastTime = 0;
// Set timer to 10 minutes (600000)
//unsigned long timerDelay = 600000;
// Timer set to 10 seconds (10000)
unsigned long timerDelay = 10000;

// DHT sensor reading
void readTempHumid(){
  humid=dht.readHumidity(); //
  temp=dht.readTemperature(); //
  if(isnan(humid)||isnan(temp)){
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
}
void setup() {
  dht.begin();  //dht
  Serial.println("DHT begins");
  delay(1000);
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  Serial.println("Connecting");
  while(WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to WiFi network with IP Address: ");
  Serial.println(WiFi.localIP());
 
  Serial.println("Timer set to 10 seconds (timerDelay variable), it will take 10 seconds before publishing the first reading.");

  // Random seed is a number used to initialize a pseudorandom number generator
  randomSeed(analogRead(0));
}

void loop() {
  //Send an HTTP POST request every 10 seconds
  if ((millis() - lastTime) > timerDelay) {
    //Check WiFi connection status
    if(WiFi.status()== WL_CONNECTED){
      HTTPClient http;
      readTempHumid(); 
      // Your Domain name with URL path or IP address with path
      http.begin(serverName);
      
      // Specify content-type header
      http.addHeader("Content-Type", "application/x-www-form-urlencoded");
      // Data to send with HTTP POST
      String httpRequestData = "value1=" + String(Location) + "&value2=" + String(temp)+ "&value3=" + String(humid);           
      // Send HTTP POST request
      int httpResponseCode = http.POST(httpRequestData);
      
      /*
      // If you need an HTTP request with a content type: application/json, use the following:
      http.addHeader("Content-Type", "application/json");
      // JSON data to send with HTTP POST
      String httpRequestData = "{\"value1\"😕"" + String(random(40)) + "\",\"value2\"😕"" + String(random(40)) + "\",\"value3\"😕"" + String(random(40)) + "\"}";
      // Send HTTP POST request
      int httpResponseCode = http.POST(httpRequestData);
      */
     
      Serial.print("HTTP Response code: ");
      Serial.println(httpResponseCode);
        
      // Free resources
      http.end();
    }
    else {
      Serial.println("WiFi Disconnected");
    }
    lastTime = millis();
  }
}
