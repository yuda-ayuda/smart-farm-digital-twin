# The Two-Way Data Flow
*How data travels from the physical world to the screen, and back again.*

A true SCADA system does two things: it **monitors** (reads data) and it **controls** (sends commands). Here is the exact path the data takes in the Smart Farm.

### ⬆️ Upstream Flow (Monitoring the Light Sensor)
1. **The Sensor:** The physical photocell on the ESP32 measures the room's brightness.
2. **The Wi-Fi:** The ESP32 board pushes that reading over your home Wi-Fi network.
3. **The Gateway:** The MQTT Distributor module receives the reading at `tcp://localhost:1883`.
4. **The Engine:** The MQTT Engine module translates that raw network message into a live Ignition Tag.
5. **The UI Canvas:** The Ignition Designer displays that Tag on a visual gauge or label. 

### ⬇️ Downstream Flow (Turning on the LED)
1. **The Human:** A user clicks a Toggle Switch on the Perspective Dashboard.
2. **The Python Script:** An Event Script triggers, reading the switch (True/False) and translating it into the text "ON" or "OFF".
3. **The Bypass:** The script uses the `system.cirruslink.engine.publish` command to bypass the Tag Browser and shoot a message directly into the MQTT Distributor.
4. **The Wi-Fi:** The message travels back over the network to the specific topic `farm/light/set`.
5. **The Board:** The ESP32 receives the word "ON", fires up a physical pin, and lights the LED.
