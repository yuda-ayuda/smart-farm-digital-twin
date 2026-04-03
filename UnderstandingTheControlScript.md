# The Control Script (Python/Jython)
*Decoding the script used to control physical hardware.*

To flip the physical LED from a web browser, we use a Python script. This acts as a translator between the Toggle Switch (which speaks `True/False`) and the ESP32 (which expects `"ON"/"OFF"`).

### The Script Breakdown

```python
# 1. GET THE SWITCH STATE
is_on = self.props.selected
# Looks at the Toggle Switch and saves its state (True or False).

# 2. TRANSLATE TO TEXT
msg = "ON" if is_on else "OFF"
# If the switch is True, the message is "ON". Otherwise, it's "OFF".

# 3. FIRE THE COMMAND TO THE NETWORK
system.cirruslink.engine.publish("Chariot SCADA", "farm/light/set", str(msg).encode("utf-8"), 0, 0)
# system.cirruslink.engine.publish: The command to send an MQTT message.
# "Chariot SCADA": Your local Gateway connection name.
# "farm/light/set": The topic the ESP32 is listening to.
# str(msg).encode("utf-8"): Packages the text safely for the network.
