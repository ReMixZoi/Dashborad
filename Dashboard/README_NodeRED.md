# Node-RED Integration Guide

This Dashboard is designed to work seamlessly with **Node-RED**. 
Node-RED can act as the "backend" that reads data from your physical sensors (via Modbus, Serial, Zigbee, etc.) and sends it to this web dashboard via MQTT.

## Quick Start with Node-RED

1.  **Install Node-RED**: If you haven't already.
    ```bash
    npm install -g --unsafe-perm node-red
    node-red
    ```
2.  **Open Node-RED**: Go to `http://localhost:1880`.
3.  **Import the Flow**:
    *   Click the **Menu (≡)** (top right) -> **Import**.
    *   Select the `node_red_flow.json` file included in this project.
    *   Click **Import**.
4.  **Deploy**: Click the red **Deploy** button in the top right.

## How it Works

The flow does the following:
1.  **Inject Node**: Triggers every 2 seconds.
2.  **Function Node**: Simulates data (Voltage, Current, Power). *You can replace this with actual Modbus Read nodes.*
3.  **MQTT Out Node**: Publishes the JSON data to the topic `energy/monitor/main` on `broker.emqx.io`.

## Connecting Real Sensors

To use real sensors instead of the simulation:
1.  Install the Modbus nodes in Node-RED:
    *   Menu -> Manage Palette -> Install -> Search `node-red-contrib-modbus`.
2.  Add a **Modbus Read** node to the flow.
3.  Configure it to read your meter's registers (Voltage, Current, Power).
4.  Feed the output into a **Function** node to format it as a JSON object:
    ```javascript
    msg.payload = {
        "voltage": msg.payload[0], // example index
        "current": msg.payload[1],
        "power": msg.payload[2]
    };
    return msg;
    ```
5.  Connect this to the **MQTT Out** node.

## Dashboard Setup

1.  Open `index.html` in your browser.
2.  Click **⚙ SETTINGS**.
3.  Select **Node-RED / MQTT Live**.
4.  Ensure the Topic matches: `energy/monitor/main`.
5.  You should see the data streaming in!
