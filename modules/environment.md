# Environment Module

The environment module monitors vessel and ambient conditions including temperature, humidity, and bilge water detection.

---

## Hardware

| Component | Model |
|-----------|-------|
| MCU | ESP32 |
| CAN transceiver | SN65HVD230 |
| Temp/humidity sensor | DHT22 or BME280 |
| Bilge sensor | Float switch or conductivity probe |

---

## MQTT Topics

| Topic | Unit | Description |
|-------|------|-------------|
| `helmos/environment/temp` | °C | Air temperature |
| `helmos/environment/humidity` | % | Relative humidity |
| `helmos/environment/bilge` | bool | Bilge water present |
| `helmos/environment/pressure` | hPa | Barometric pressure (if BME280) |

Data is published every 60 seconds by default, and immediately on bilge alarm.

---

## Bilge Alarm

If `helmos/environment/bilge` publishes `true`, the core triggers a UI alert and logs the event with timestamp. The alert persists until the bilge sensor returns `false`.

---

## Config Schema

```json
{
  "publish_interval_s": { "type": "int", "default": 60, "min": 10, "max": 3600 },
  "bilge_pin":          { "type": "int", "default": 4 },
  "sensor_type":        { "type": "string", "default": "DHT22", "options": ["DHT22", "BME280"] }
}
```
