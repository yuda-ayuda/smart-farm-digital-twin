# ESP32 Smart Farm Home Setup Guide

This code allows you to connect your ESP32 to your home network and communicate with an Ignition MQTT Broker. 

## 📋 Student Instructions
To make this work at home, you **MUST** update the following three lines in the code below:
1. **ssid**: Enter your home WiFi name inside the quotes.
2. **password**: Enter your home WiFi password inside the quotes.
3. **mqtt_server**: Enter the IP address of the computer running Ignition.
4. **Namespace**: This code uses `daisyfarm`. If you changed your namespace in Ignition to your own name, make sure to replace `daisyfarm` in all the `client.publish` and `client.subscribe` lines.

---

## 💻 ESP32 Arduino Code

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// ==========================================
//      ⬇️ UPDATE THESE THREE SETTINGS ⬇️
// ==========================================
const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";
const char* mqtt_server = "YOUR_IP_ADDRESS"; 

// --- CONFIGURATION ---
const char* mqtt_user = "esp32";
const char* mqtt_pass = "pass"; 

// --- PINS (Verify your wiring) ---
#define ButtonPin 5
#define LED 27
#define PhotocellPin 34

int ledState = 0; 
unsigned long lastSensorTime = 0;
const long sensorInterval = 2000;

WiFiClient espClient;
PubSubClient client(espClient);

void callback(char* topic, byte* payload, unsigned int length) {
  String message = "";
  for (int i = 0; i < length; i++) message += (char)payload[i];
  
  Serial.print("Ignition says: ");
  Serial.println(message);

  // Logic to respond to Ignition Commands
  if (message == "ON") {
    ledState = 1;
    digitalWrite(LED, HIGH);
    client.publish("daisyfarm/light", "ON"); 
  } else if (message == "OFF") {
    ledState = 0;
    digitalWrite(LED, LOW);
    client.publish("daisyfarm/light", "OFF"); 
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(ButtonPin, INPUT);
  pinMode(LED, OUTPUT);
  pinMode(PhotocellPin, INPUT);
  digitalWrite(LED, LOW); 

  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect("ESP32Daisy", mqtt_user, mqtt_pass)) {
      Serial.println("connected");
      // Subscribe to the control topic
      client.subscribe("daisyfarm/light/set");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Manual Button Logic (Physical Toggle)
  if (digitalRead(ButtonPin) == 0) {
    delay(10); 
    if (digitalRead(ButtonPin) == 0) {
      ledState = !ledState; 
      digitalWrite(LED, ledState ? HIGH : LOW);
      client.publish("daisyfarm/light", ledState ? "ON" : "OFF");
      while (digitalRead(ButtonPin) == 0); 
    }
  }

  // Sensor Logic (Sends data to Ignition every 2 seconds)
  if (millis() - lastSensorTime >= sensorInterval) {
    lastSensorTime = millis(); 
    int lightVal = analogRead(PhotocellPin);
    char msg[10];
    itoa(lightVal, msg, 10);
    client.publish("daisyfarm/lightLevel", msg);
  }
}
