# RS485 Modbus to MQTT Gateway Guide

To connect your Power Meter (RS485/Modbus) to the web dashboard, you need a small gateway script running on a computer or computer (like Raspberry Pi) that reads the data and sends it to the MQTT broker.

## Hardware Requirements
1. **Power Meter**: Must support Modbus RTU over RS485 (e.g., PZEM-016, SDM120).
2. **USB-to-RS485 Adapter**: To connect the meter wiring to your computer.

## Software Setup (Python Example)

You can use a simple Python script to bridge the data.

### 1. Install Libraries
```bash
pip install minimalmodbus paho-mqtt
```

### 2. Create Script (`gateway.py`)
```python
import minimalmodbus
import paho.mqtt.client as mqtt
import time
import json

# --- Configuration ---
MODBUS_PORT = 'COM3'  # Windows: COMx, Linux: /dev/ttyUSB0
SLAVE_ID = 1          # Modbus Slave ID of your meter
MQTT_BROKER = 'broker.emqx.io'
MQTT_TOPIC = 'energy/monitor/main'

# --- Modbus Setup ---
instrument = minimalmodbus.Instrument(MODBUS_PORT, SLAVE_ID)
instrument.serial.baudrate = 9600
instrument.serial.timeout = 1

# --- MQTT Setup ---
client = mqtt.Client()
client.connect(MQTT_BROKER, 1883, 60)

print(f"Reading from {MODBUS_PORT} and publishing to {MQTT_TOPIC}...")

while True:
    try:
        # Example Register Addresses (Varies by Meter Model!)
        # Check your meter's manual for correct register addresses.
        
        # Reading Voltage (Address 0, 1 decimal place usually)
        voltage = instrument.read_register(0, 1) 
        
        # Reading Current (Address 1, 3 decimal places) - Example
        current = instrument.read_register(1, 3) 
        
        # Reading Power (Address 3, 1 decimal place) - Example
        power_w = instrument.read_register(3, 1) 
        power_kw = power_w / 1000.0
        
        payload = {
            "voltage": voltage,
            "current": current,
            "power": power_kw
        }
        
        # Publish
        client.publish(MQTT_TOPIC, json.dumps(payload))
        print(f"Sent: {payload}")
        
    except Exception as e:
        print(f"Error: {e}")
        
    time.sleep(2)
```

### 3. Usage
Run this script on the computer connected to the RS485 adapter. The web dashboard will automatically update when it receives the data from the MQTT broker.
