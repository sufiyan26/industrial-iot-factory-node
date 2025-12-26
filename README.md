# Industrial IoT Factory Node – ESP32

A complete **ESP32-based industrial monitoring node** prototyped in Wokwi simulator. Continuously monitors temperature, humidity, motion, and light levels on factory floor, triggers local alarms for unsafe conditions, displays status on OLED, and streams structured telemetry to cloud via MQTT. Features robust safety interlocks and remote control capabilities.

## Core Concept & Objective

- **Goal:** Deploy edge nodes that provide real-time factory environmental monitoring with local alarming and cloud telemetry, while preventing unsafe remote operations during alarm conditions.
- **Key Requirements:**
  - Multi-sensor fusion (temp/humidity/motion/light)
  - Local visual/audible alarms
  - Structured JSON telemetry over MQTT
  - Safety interlocks for remote commands

## Technical Design

### Hardware (Wokwi Simulation)
ESP32 (WiFi + dual-core)
├── DHT22 (temperature + humidity)
├── PIR HC-SR501 (motion detection)
├── Potentiometer/LDR (analog light/setpoint)
├── Push-button (local reset/override)
├── Buzzer (1000-2000 Hz alarm)
└── SSD1306 OLED (I²C display)
Main Loop (100ms):
├── Poll sensors (DHT22, PIR, analog)
├── Alarm logic: temp>35°C OR PIR in guard window
├── OLED update: current readings + alarm status
├── MQTT JSON publish every 3s: factory/node1/data
└── Command callback: factory/node1/cmd (safety-checked)

MQTT Topics:
Publish: factory/node1/data → {"temp":39.4,"hum":40.2,"motion":1,"light":0.65}
Subscribe: factory/node1/cmd → {"remote_led":1,"override":0}


### Safety & Reliability Features
- **Alarm Interlocks:** Remote commands blocked during active alarms
- **Debouncing:** Button inputs with edge detection
- **Reconnects:** Automatic MQTT/WiFi reconnection logic
- **JSON Validation:** Structured payloads with snprintf formatting
- **Watchdog:** ESP32 hardware watchdog for crash recovery

## Project Structure
├── src/
│ └── factory_node.ino # Complete ESP32 firmware
├── lib/
│ ├── WiFi.h # WiFi connection + reconnects
│ ├── PubSubClient.h # MQTT client + JSON formatting
│ ├── DHT.h # Temperature/humidity sensor
│ └── Adafruit_SSD1306.h # OLED display driver
├── wokwi/
│ └── diagram.json # Complete node + sensor simulation
└── docs/
├── mqtt_schema.md # Topic structure and JSON format
└── alarm_scenarios.md # Tested failure conditions

## Alarm Logic & Triggers
ALARM CONDITIONS:
├── Temperature > 35°C (immediate)
├── PIR motion detected in 10s guard window (20s cooldown)
└── BOTH → High-priority alarm (continuous buzzer)

ALARM ACTIONS:
├── Local: Buzzer (1000-2000Hz sweep) + OLED "ALARM"
├── MQTT: JSON flag + increased publish rate (1s)
└── Remote: All commands blocked until local button reset

## Example MQTT Telemetry
**Normal Operation:**
{"node":"factory1","temp":28.4,"hum":52.1,"motion":0,"light":0.72,"alarm":0,"ts":"2025-10-15T09:32:15Z"}

**Alarm Active:**
{"node":"factory1","temp":39.4,"hum":40.2,"motion":1,"light":0.45,"alarm":1,"ts":"2025-10-15T09:33:22Z"}

## Test Results

| Scenario | Duration | Result |
|----------|----------|--------|
| **Temp Alarm** (>35°C) | Continuous | 100% detection, proper MQTT alert |
| **PIR Motion** (guard window) | 10s trigger | Correct cooldown, no false positives |
| **Remote Block** (during alarm) | All commands | Safety interlock working |
| **MQTT Uptime** | 24h simulation | 100% message delivery |

## Broker Configuration
HiveMQ Cloud (port 1883)
Topic: factory/node1/data (publish)
Topic: factory/node1/cmd (subscribe)
QoS: 1 (at-least-once delivery)

## Skills Demonstrated
ESP32 firmware development • Multi-sensor integration (I²C/GPIO/analog) • MQTT v3.1.1 (PubSubClient) • JSON telemetry formatting • Safety interlocks and state machines • Wokwi hardware simulation • OLED UI development • Industrial IoT edge node design
