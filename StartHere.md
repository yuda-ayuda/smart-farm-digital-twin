# 🛠️ Step 0: The Simple Hardware Test (Input & Output)

Before diving into Wi-Fi, MQTT, and Ignition, use this simple script to prove your physical wires are connected correctly and to understand how **Inputs** and **Outputs** work.

## 📝 The Code
Copy and paste this into a new Arduino Sketch:

```cpp
#define LED 27        // The Pin connected to the White LED
#define ButtonPin 5   // The Pin connected to the Button

void setup() {
  // OUTPUT: The Board sends electricity OUT to the LED
  pinMode(LED, OUTPUT);
  
  // INPUT: The Board waits for a signal to come IN from the Button
  pinMode(ButtonPin, INPUT);
}

void loop() {
  // If the Button is pressed (Reading 0/LOW in this kit)
  if (digitalRead(ButtonPin) == 0) {
    digitalWrite(LED, HIGH); // Send power to the LED (ON)
  } else {
    digitalWrite(LED, LOW);  // Cut power to the LED (OFF)
  }
}
