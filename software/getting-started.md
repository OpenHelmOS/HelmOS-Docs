# Getting Started

This guide covers installing OpenHelmOS on a Raspberry Pi 5 from scratch.

## Prerequisites

- Raspberry Pi 5 (4GB or 8GB)
- Raspberry Pi OS (64-bit, Desktop)
- MicroSD or NVMe SSD
- Network connection for initial setup
- SSH access or keyboard + monitor

---

## 1. Install Raspberry Pi OS

Use Raspberry Pi Imager to flash Raspberry Pi OS 64-bit Desktop to your storage. Enable SSH and set username/password in the imager settings before flashing.

---

## 2. Initial System Setup

```bash
sudo apt update && sudo apt upgrade -y

# Set graphical boot target (required for display and VNC)
sudo systemctl set-default graphical.target

# Configure headless display resolution (if no monitor attached)
sudo nano /boot/firmware/config.txt
```

Add to end of config.txt:
```
hdmi_force_hotplug=1
hdmi_group=2
hdmi_mode=82
```

Set display manager to X11 (not Wayland):
```bash
sudo raspi-config
# Advanced Options → Wayland → X11
```

---

## 3. Install Dependencies

```bash
# MQTT broker
sudo apt install mosquitto mosquitto-clients -y
sudo systemctl enable mosquitto

# Python
sudo apt install python3-pip python3-venv -y

# Node.js via nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 20

# VNC server
sudo apt install realvnc-vnc-server -y
sudo systemctl enable vncserver-x11-serviced

# Kiosk utilities
sudo apt install chromium-browser unclutter -y
```

---

## 4. Clone Repositories

```bash
cd ~
git clone https://github.com/openhelmos/helmos-core.git
git clone https://github.com/openhelmos/helmos-ui.git

# Install Python dependencies
cd ~/helmos-core
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Install Node dependencies
cd ~/helmos-ui
npm install
```

---

## 5. Configure Systemd Services

### helmos-core

```bash
sudo nano /etc/systemd/system/helmos-core.service
```

```ini
[Unit]
Description=HelmOS Core FastAPI
After=network.target mosquitto.service
Wants=mosquitto.service

[Service]
User=YOUR_USERNAME
WorkingDirectory=/home/YOUR_USERNAME/helmos-core
ExecStart=/home/YOUR_USERNAME/helmos-core/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### helmos-ui

Check your exact Node version path first:
```bash
ls ~/.nvm/versions/node/
```

```bash
sudo nano /etc/systemd/system/helmos-ui.service
```

```ini
[Unit]
Description=HelmOS UI
After=network.target helmos-core.service

[Service]
User=YOUR_USERNAME
WorkingDirectory=/home/YOUR_USERNAME/helmos-ui
ExecStart=/home/YOUR_USERNAME/.nvm/versions/node/vX.X.X/bin/node \
  /home/YOUR_USERNAME/helmos-ui/node_modules/.bin/vite \
  --host 0.0.0.0 --port 5173
Restart=always
RestartSec=5
Environment=NODE_ENV=development

[Install]
WantedBy=multi-user.target
```

Enable and start all services:
```bash
sudo systemctl daemon-reload
sudo systemctl enable helmos-core helmos-ui
sudo systemctl start helmos-core helmos-ui
```

---

## 6. Configure Autologin

```bash
sudo nano /etc/lightdm/lightdm.conf
```

Under `[Seat:*]` add:
```ini
autologin-user=YOUR_USERNAME
autologin-user-timeout=0
autologin-session=LXDE-pi-x
```

Remove keyring password to prevent unlock prompt:
```bash
rm ~/.local/share/keyrings/login.keyring
```

---

## 7. Configure Kiosk Mode

```bash
mkdir -p ~/.config/autostart

# Chromium kiosk
nano ~/.config/autostart/helmos-kiosk.desktop
```

```ini
[Desktop Entry]
Type=Application
Name=HelmOS Kiosk
Exec=bash -c "sleep 5 && chromium-browser --kiosk --noerrdialogs --disable-infobars --no-first-run --disable-restore-session-state http://localhost:5173"
X-GNOME-Autostart-enabled=true
```

```bash
# Hide mouse cursor
nano ~/.config/autostart/unclutter.desktop
```

```ini
[Desktop Entry]
Type=Application
Name=Unclutter
Exec=unclutter -idle 0.1 -root
X-GNOME-Autostart-enabled=true
```

---

## 8. Reboot and Verify

```bash
sudo reboot
```

After reboot:
- Chromium should open automatically showing helmos-ui
- VNC should be accessible on port 5900
- All services should be running

Verify services:
```bash
sudo systemctl status mosquitto helmos-core helmos-ui
```
