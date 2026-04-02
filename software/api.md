# API Reference

helmos-core exposes a REST API on port 8000. All endpoints return JSON.

Base URL: `http://<rpi-ip>:8000`

---

## System

### GET /
Health check.

```json
{ "status": "ok", "version": "1.0.0" }
```

---

## Config

### GET /config
Returns the full config.json contents.

### POST /config
Replaces the full config. Triggers a config push to all active modules.

### GET /config/{section}
Returns one top-level section of config.

```
GET /config/autopilot
GET /config/vessel
GET /config/rudder
```

### POST /config/{section}
Updates one section. Merges with existing values.

---

## LED (Development)

These endpoints exist for development and testing of the MQTT/WebSocket pipeline.

### GET /led
Returns current LED state.

```json
{ "state": "off" }
```

### POST /led/{state}
Sets LED state. Publishes to `helmos/led/set`.

```
POST /led/on
POST /led/off
```

---

## WebSocket

### WS /ws
Real-time state updates. The server pushes a message whenever MQTT state changes.

```json
{ "state": "on" }
```

Connect from the UI:
```typescript
const ws = new WebSocket("ws://<rpi-ip>:8000/ws")
ws.onmessage = (event) => {
  const data = JSON.parse(event.data)
}
```

---

## Modules

### GET /modules
Returns all known modules and their status.

```json
{
  "autopilot-01": {
    "type": "autopilot",
    "configured": true,
    "status": "online",
    "last_seen": "2026-04-01T18:00:00"
  }
}
```

### POST /modules/{id}/configure
Mark a module as configured after wizard completion.

### POST /modules/{id}/restart
Send restart command to module via MQTT.

---

## Autopilot

### GET /autopilot/status
Returns current autopilot state.

```json
{
  "state": "active",
  "target_heading": 245,
  "current_heading": 243,
  "rudder_angle": 12,
  "pid": { "Kp": 1.0, "Ki": 0.01, "Kd": 0.5 }
}
```

### POST /autopilot/enable
Activates autopilot with current heading as target.

### POST /autopilot/disable
Disables autopilot, returns to manual steering.

### POST /autopilot/target
Sets target heading.

```json
{ "heading": 270 }
```

### POST /autopilot/pid
Updates PID values.

```json
{ "Kp": 1.2, "Ki": 0.01, "Kd": 0.6 }
```
