# 🚜 The Future Farmer's Guide to IoT: Building a Smart Farm

**Welcome, Engineer.**

This repository contains the code and instructions to build a **Digital Twin** of a Smart Farm. You are about to learn how to make a physical machine (a Smart Farm kit) talk to a powerful computer brain (a Server) over the invisible waves of the internet.

This is the exact same technology used by Tesla factories, Amazon warehouses, and vertical farming startups. By the end of this guide, *you* will know how to build it.

---

## 🧠 What You Will Learn
* **C++ Coding:** Writing the language that controls hardware.
* **Networking & IP:** How computers find each other in a crowded room.
* **MQTT:** The language machines use to whisper to each other across the world.
* **SCADA:** Building a "Mission Control" dashboard to monitor your farm from anywhere.

---

## 🛠 Phase 1: The Setup

Professional engineers spend 80% of their time fixing setup issues. If this part feels hard, it’s not because you’re bad at it—it’s because it *is* hard. Follow these steps to get your tools ready.

### 1. Install the Driver
Your computer needs a "translator" to talk to the cheap computer chip on the farm kit.
* **Download:** Get the **CH340 Driver** (search for `CH34xVCPDriver`).
* **Install:** Run the installer.
* **⚠️ Mac Users:** You must go to **System Settings > Privacy & Security** and click **Allow** after installing. Then restart your computer.

### 2. Prepare Arduino IDE
This is where we write the code for the physical board.
1.  Download **Arduino IDE 2.0**.
2.  **Add the ESP32 Board Manager:**
    * Go to *Settings* (or Preferences).
    * Paste this link into "Additional Boards Manager URLs":
        `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
3.  **Install the Board Package:**
    * Go to *Tools > Board > Boards Manager*.
    * Search for **ESP32** and install the package by *Espressif Systems*.
4.  **Install Libraries:**
    * Go to *Sketch > Include Library > Manage Libraries*.
    * Search for and install **PubSubClient** by Nick O'Leary.
    * (Optional) Install the library folder that came with your specific kit using *Add .ZIP Library*.

### 3. Prepare the Server (Ignition)
Ignition is industrial software used by big factories. We will use the free **Maker Edition**.
1.  Download **Ignition 8.1** (Select the version matching your computer chip).
2.  **Install:** Run the installer.
3.  **Launch:** Open your browser to `http://localhost:8088`.
4.  **Create User:** Set up an Admin account and **write down your password!**
5.  **Install MQTT Modules:**
    * Go to *Config > Modules*.
    * Install **MQTT Distributor** and **MQTT Engine** (downloaded from Cirrus Link).

---

## 🔌 Phase 2: The Hardware Hookup

### 1. Connect the Board
* Plug the ESP32 board into your computer.
* In Arduino IDE, go to *Tools > Port*. Select the USB port (e.g., `/dev/cu.usbserial...` or `COM3`).
* **Troubleshooting:** If you don't see the port, your cable might be "Charge Only." Swap it for a data cable.

### 2. The "Servo Trap" (⚠️ CRITICAL)
* **Before** building the wooden house, plug the Blue Servo Motor into the board.
    * **Brown:** G (Ground)
    * **Red:** V (5V)
    * **Orange:** Pin 26
* Upload the basic servo code to move it to **180 degrees**.
* *Why?* If you don't do this, the motor might snap the plastic door when you turn it on later!

---

## 📡 Phase 3: The Code (The "Brain")

We need to create an MQTT User so the board is allowed to talk to the server.
1.  Go to `http://localhost:8088`.
2.  Navigate to **Config > MQTT Distributor > Settings > Users**.
3.  Create a new user:
    * **Username:** `esp32`
    * **Password:** `farm`
    * **ACLs:** `RW #` (This creates a "Read/Write All" permission).

### The Master Sketch
Copy the code below into a new Arduino file. You must edit the top 3 lines to match your home network.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// ==========================================
//      ⬇️ UPDATE THESE 3 LINES ⬇️
// ==========================================
const char* ssid = "YOUR_WIFI_NAME";        // Keep the quotes!
const char* password = "YOUR_WIFI_PASSWORD";
const char* mqtt_server = "172.16.x.x";   // Your Computer's IP Address

// --- CONFIGURATION ---
const char* mqtt_user = "esp32";
const char* mqtt_pass = "farm";

// --- PINS ---
#define ButtonPin 5
#define LED 27
#define PhotocellPin 34

// --- VARIABLES ---
int ledState = 0; 
unsigned long lastSensorTime = 0;
const long sensorInterval = 2000; // Read sensor every 2 seconds

WiFiClient espClient;
PubSubClient client(espClient);

// --- THE LISTENER (The Board's Ears) ---
void callback(char* topic, byte* payload, unsigned int length) {
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.print("Ignition says: ");
  Serial.println(message);

  if (message == "ON") {
    ledState = 1;
    digitalWrite(LED, HIGH);
    client.publish("farm/light", "ON"); 
  } 
  else if (message == "OFF") {
    ledState = 0;
    digitalWrite(LED, LOW);
    client.publish("farm/light", "OFF"); 
  }
}

void setup() {
  Serial.begin(115200);
  
  // Setup Pins
  pinMode(ButtonPin, INPUT);
  pinMode(LED, OUTPUT);
  pinMode(PhotocellPin, INPUT);
  digitalWrite(LED, LOW); 

  // Network Setup
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // TASK 1: CHECK PHYSICAL BUTTON (Instant)
  int ReadValue = digitalRead(ButtonPin);
  if (ReadValue == 0) {
    delay(10); 
    if (digitalRead(ButtonPin) == 0) {
      ledState = !ledState; 
      
      if(ledState) {
        digitalWrite(LED, HIGH);
        client.publish("farm/light", "ON"); 
      } else {
        digitalWrite(LED, LOW);
        client.publish("farm/light", "OFF");
      }
      while (digitalRead(ButtonPin) == 0); // Wait for release
    }
  }

  // TASK 2: READ SENSOR (Every 2 Seconds)
  unsigned long currentMillis = millis();
  if (currentMillis - lastSensorTime >= sensorInterval) {
    lastSensorTime = currentMillis; 
    
    int lightVal = analogRead(PhotocellPin);
    char msg[10];
    itoa(lightVal, msg, 10);
    
    client.publish("farm/lightLevel", msg);
  }
}

// --- NETWORK HELPERS ---
void setup_wifi() {
  delay(10);
  Serial.print("Connecting to WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected!");
}

void reconnect() {
  while (!client.connected()) {
    if (client.connect("ESP32Client", mqtt_user, mqtt_pass)) {
      Serial.println("Connected to Ignition");
      client.subscribe("farm/light/set"); // Subscribe to remote commands
      
      // Sync Light Status
      if(ledState) client.publish("farm/light", "ON");
      else client.publish("farm/light", "OFF");
      
    } else {
      delay(5000);
    }
  }
}
