# Smart Solar Limiter with Node-RED | OpenDTU | MQTT ⚡

![Node-RED](https://img.shields.io/badge/Node--RED-supported-red)
![OpenDTU](https://img.shields.io/badge/OpenDTU-compatible-green)
![MQTT](https://img.shields.io/badge/MQTT-integrated-blue)
![License](https://img.shields.io/badge/license-MIT-yellow)

> ⚠️ **Important:** I take no responsibility for any damages or losses caused by using this project! Use at your own risk.

---

## ℹ️ What is Smart Solar Limiter?

Smart Solar Limiter means that the photovoltaic (PV) system does not feed energy into the public power grid. This is achieved by limiting the power output of the PV system to match the current power consumption of the household. With the help of Node-RED, OpenDTU, and MQTT, such dynamic power limitation can be implemented.

---

## 📋 Requirements

- [Node-RED](https://github.com/node-red/node-red?tab=readme-ov-file)
- [Current power consumption is readable (Tasmota IR Reader)](https://tasmota.github.io/docs/Smart-Meter-Interface/)
- [Your inverter can be limited (via MQTT)](https://tasmota.github.io/docs/Smart-Meter-Interface/)
- [ESP-32 with OpenDTU](https://github.com/tbnobody/OpenDTU)

---

## ⚙️ How it works

This Node-RED flow monitors the household's current power consumption and adjusts the inverter's power limit dynamically based on that information.

### 🔄 Process Steps:

1. The script runs every 15 seconds
2. A time check verifies whether the current time is between sunrise + 2 hours and sunset - 30 minutes
3. "Current Consumption in W" retrieves the household's power consumption from the Tasmota IR Reader
4. The function calculates a new power limit for the inverter based on the current consumption
5. The `rbe` node debounces the power limit to prevent frequent changes
6. The `range` node scales the power limit from 0-1500 watts to 8-100 percent
7. The `mqtt out` node sends the scaled power limit to the inverter via MQTT

---

## 📊 Node-RED Flow

![Screenshot](./Node-RED_Flow.png)

```json
[{"id":"b25cd44df48429b5","type":"tab","label":"Flow 1","disabled":true,"info":"","env":[]},{"id":"176d8c7d11a5c0d1","type":"mqtt out","z":"b25cd44df48429b5","name":"WR MQTT","topic":"solar/SERIENNUMMER_DES_WECHSELRICHTERS/cmd/limit_nonpersistent_relative","qos":"","retain":"","respTopic":"","contentType":"","userProps":"","correl":"","expiry":"","broker":"87b2d138566ff5cc","x":1670,"y":320,"wires":[]},{"id":"a1a3647f577a1900","type":"inject","z":"b25cd44df48429b5","name":"every 15 seconds","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"16","crontab":"","once":false,"onceDelay":0.1,"topic":"","payload":"","payloadType":"date","x":130,"y":320,"wires":[["7bbf98821b7d784c"]]},{"id":"703d1fe697175c20","type":"function","z":"b25cd44df48429b5","name":"Inverter Limit Calculation","func":"// MAX generation of the inverter Watts\nvar maxPower = 1500;\n\n// Get current power limit or default\nvar power = context.get('power') || maxPower;\npower += msg.payload;\n\n// clamp power between 0 and max\nif (power > maxPower) power = maxPower;\nif (power < 0) power = 1;\n\n\n// store current powerlimit and update message\ncontext.set('power', power);\nmsg.payload = power;\n\nreturn msg;","outputs":1,"timeout":"","noerr":0,"initialize":"","finalize":"","libs":[],"x":990,"y":320,"wires":[["c1b6b5ab6f5d47d4","b0c2e49e6d7dd9da"]]},{"id":"905059e3e97e32e9","type":"api-current-state","z":"b25cd44df48429b5","name":"Current Consumption in W","server":"64eac69f.fe1218","version":3,"outputs":1,"halt_if":"","halt_if_type":"str","halt_if_compare":"is","entity_id":"sensor.tasmota_lk13be_power_curr","state_type":"num","blockInputOverrides":false,"outputProperties":[{"property":"payload","propertyType":"msg","value":"","valueType":"entityState"},{"property":"data","propertyType":"msg","value":"","valueType":"entity"}],"for":"0","forType":"num","forUnits":"minutes","override_topic":false,"state_location":"payload","override_payload":"msg","entity_location":"data","override_data":"msg","x":670,"y":320,"wires":[["703d1fe697175c20","dfb1de93b5139f49"]]},{"id":"c1b6b5ab6f5d47d4","type":"rbe","z":"b25cd44df48429b5","name":"debounce","func":"deadband","gap":"30","start":"","inout":"in","septopics":false,"property":"payload","topi":"topic","x":1200,"y":320,"wires":[["46dd59a169196802","7e02cb429a063346"]]},{"id":"b0c2e49e6d7dd9da","type":"debug","z":"b25cd44df48429b5","name":"Calculation","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","targetType":"msg","statusVal":"","statusType":"auto","x":1190,"y":440,"wires":[]},{"id":"dfb1de93b5139f49","type":"debug","z":"b25cd44df48429b5","name":"Current Consumption","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","targetType":"msg","statusVal":"","statusType":"auto","x":920,"y":440,"wires":[]},{"id":"7bbf98821b7d784c","type":"time-range-switch","z":"b25cd44df48429b5","name":"Time Control","lat":"52.199424","lon":"13.4742016","startTime":"sunrise","endTime":"sunset","startOffset":"120","endOffset":"-30","x":380,"y":280,"wires":[["905059e3e97e32e9"],[]]},{"id":"46dd59a169196802","type":"debug","z":"b25cd44df48429b5","name":"Debounce","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","targetType":"msg","statusVal":"","statusType":"auto","x":1420,"y":440,"wires":[]},{"id":"7e02cb429a063346","type":"range","z":"b25cd44df48429b5","minin":"0","maxin":"1500","minout":"8","maxout":"100","action":"scale","round":true,"property":"payload","name":"Convert to Percentage","x":1450,"y":320,"wires":[["f3280877d71191df","176d8c7d11a5c0d1"]]},{"id":"f3280877d71191df","type":"debug","z":"b25cd44df48429b5","name":"To Percentage","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","targetType":"msg","statusVal":"","statusType":"auto","x":1670,"y":440,"wires":[]},{"id":"87b2d138566ff5cc","type":"mqtt-broker","name":"","broker":"192.168.188.88","port":"1883","clientid":"","autoConnect":true,"usetls":false,"protocolVersion":"4","keepalive":"60","cleansession":true,"autoUnsubscribe":true,"birthTopic":"","birthQos":"0","birthPayload":"","birthMsg":{},"closeTopic":"","closeQos":"0","closePayload":"","closeMsg":{},"willTopic":"","willQos":"0","willPayload":"","willMsg":{},"userProps":"","sessionExpiry":""},{"id":"64eac69f.fe1218","type":"server","name":"Home Assistant","version":5,"addon":true,"rejectUnauthorizedCerts":true,"ha_boolean":"y|yes|true|on|home|open","connectionDelay":true,"cacheJson":true,"heartbeat":false,"heartbeatInterval":30,"areaSelector":"friendlyName","deviceSelector":"friendlyName","entitySelector":"friendlyName","statusSeparator":": ","enableGlobalContextStore":false}]
```

---

## 🔧 Configuration

### MQTT Broker Settings
```javascript
Broker: 192.168.188.88
Port: 1883
Topic: solar/SERIAL_NUMBER_OF_INVERTER/cmd/limit_nonpersistent_relative
```

### Power Consumption Sensor
```javascript
Entity: sensor.tasmota_lk13be_power_curr
```

### Inverter Settings
```javascript
Max Power: 1500 watts
Power Limit Range: 0-1500 watts → 8-100%
```

---

## 🛠️ Troubleshooting

**If the flow doesn't work:**
- Verify MQTT broker connection
- Check if the power consumption sensor is providing data
- Ensure your inverter supports MQTT control
- Verify OpenDTU is properly configured

**Time range issues:**
- Adjust latitude and longitude in the time-range-switch node
- Modify sunrise/sunset offsets as needed

---

## 📄 License

MIT License - see LICENSE file for details.
