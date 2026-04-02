# Network Architecture

## Operating Modes

OpenHelmOS operates in two network modes depending on internet availability.

### Offline Mode (normal use)

The RPi acts as a WiFi access point. Phones, tablets, and the display connect directly to it. No internet or router required.

```
┌─────────────┐     WiFi AP      ┌─────────────┐
│   RPi Core  │◄────────────────►│   Phone /   │
│  HelmOS-AP  │  192.168.4.0/24  │   Tablet    │
└─────────────┘                  └─────────────┘
      │
      │ HelmOS UI: http://192.168.4.1:5173
      │ API:       http://192.168.4.1:8000
```

### Online Mode (updates)

The RPi connects to a phone hotspot or marina WiFi. Internet access is used only for pulling updates from GitHub.

```
┌─────────────┐     WiFi client   ┌─────────────┐     Internet
│   RPi Core  │◄─────────────────►│   Phone     │◄────────────►  GitHub
│             │                   │  Hotspot    │
└─────────────┘                   └─────────────┘
```

### Network Priority

```
1. Known WiFi network available → online mode (connect as client)
2. No known network → offline mode (start as access point)
```

This is handled automatically by NetworkManager or hostapd + wpa_supplicant.

---

## Update Flow

Updates can be triggered from the UI settings panel without SSH or physical access.

```
Settings → System → Updates → [Check for updates]
        ↓
helmos-ui      v1.2.0 → v1.3.0   [Update]
helmos-core    v1.1.0 → v1.1.0   ✓ Up to date
helmos-fw      v1.0.1 → v1.1.0   [Update]
```

**helmos-ui update:**
```bash
cd ~/helmos-ui && git pull && npm install
sudo systemctl restart helmos-ui
```

**helmos-core update:**
```bash
cd ~/helmos-core && git pull
pip install -r requirements.txt
sudo systemctl restart helmos-core
```

**helmos-fw update (ESP32 OTA):**
Core downloads new firmware binary and pushes it to each module over WiFi. No USB cable needed.

---

## Ports

| Service | Port | Protocol |
|---------|------|----------|
| helmos-ui | 5173 | HTTP |
| helmos-core | 8000 | HTTP / WebSocket |
| Mosquitto | 1883 | MQTT |
| VNC | 5900 | TCP |

---

## Remote Access (Development)

SSH port forwarding allows the developer to view the RPi UI in a local browser without VNC:

```bash
ssh -L 5173:localhost:5173 -L 8000:localhost:8000 juzo@<rpi-ip>
```

Then open `http://localhost:5173` on the desktop machine.
