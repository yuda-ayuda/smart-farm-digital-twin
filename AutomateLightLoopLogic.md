```
// TASK 2: READ SENSOR (Every 2 Seconds)
  unsigned long currentMillis = millis();
  if (currentMillis - lastSensorTime >= sensorInterval) {
    lastSensorTime = currentMillis; 
    
    int lightVal = analogRead(PhotocellPin);
    char msg[10];
    itoa(lightVal, msg, 10);
    
    client.publish("farm/lightLevel", msg);

    // ⬇️ NEW AUTOMATION CODE ⬇️
    // This makes the decision locally on the board
    if (lightVal < 500 && ledState == 0) {
        ledState = 1;
        digitalWrite(LED, HIGH);
        client.publish("farm/light", "ON");
        Serial.println("Auto: It's dark! Turning light ON.");
    } 
    else if (lightVal > 800 && ledState == 1) {
        ledState = 0;
        digitalWrite(LED, LOW);
        client.publish("farm/light", "OFF");
        Serial.println("Auto: It's bright! Turning light OFF.");
    }
  }

```
