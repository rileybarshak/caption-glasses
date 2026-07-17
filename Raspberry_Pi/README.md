# Raspberry Pi

This directory contains the Raspberry Pi BLE display app and a `systemd` service unit.

- App script: `ble_text_display.py`
- Service file: `ble_text_display.service`

## Python App:

- Runs a fullscreen `pygame` overlay for captions.
- Exposes a BLE peripheral named `CGPI`.
- Uses one BLE service and three write characteristics:
  - Caption text UUID: `6E400002-B5A3-F393-E0A9-E50E24DCCA9E`
  - Sound effect UUID: `6E400003-B5A3-F393-E0A9-E50E24DCCA9E`
  - Position config UUID: `6E400004-B5A3-F393-E0A9-E50E24DCCA9E`
- Sound effect behavior:
  - If received text is `Silence` or `Speech` (case-insensitive), nothing is shown.
  - Any other text is shown above the caption text.
- Displayed text is capped to 150 characters.

## Caption Position BLE Config

Caption placement can be adjusted by writing UTF-8 text to the position config characteristic:

- UUID: `6E400004-B5A3-F393-E0A9-E50E24DCCA9E`
- `x`: horizontal center as a screen ratio. Default: `0.5`
- `y`: vertical center as a screen ratio. Default: `0.83`
- `margin`: left/right wrapping margin in pixels. Default: `40`
- `gap`: sound-effect gap above captions in line-height units. Default: `0.5`

UTF-8 Format:

```text
x=0.5,y=0.75,margin=40,gap=0.5
```

Values can be partial, so `y=0.7` updates only the vertical position.

## Pi Setup

1. Update packages and install system dependencies:
```bash
sudo apt update
sudo apt install -y python3-pygame python3-gi python3-dbus python3-venv
```

2. Create and prepare virtual environment:
```bash
cd /home/user
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install bluezero pygame
```

3. Copy the app script to the expected runtime path:
```bash
cp /path/to/repo/Raspberry_Pi/ble_text_display.py /home/user/ble_text_display.py
```

## Run Manually (Dev/Test)

```bash
cd /home/user
source .venv/bin/activate
python ble_text_display.py
```

## Install As a systemd Service

The included file expects:
- User: `user`
- Working directory: `/home/user`
- Python: `/home/user/.venv/bin/python`
- Script: `/home/user/ble_text_display.py`

1. Install the file:
```bash
sudo cp /path/to/repo/Raspberry_Pi/ble_text_display.service /etc/systemd/system/ble_text_display.service
```

2. Reload systemd and start service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable ble_text_display.service
sudo systemctl restart ble_text_display.service
```

3. Check service status/logs:
```bash
sudo systemctl status ble_text_display.service
journalctl -u ble_text_display.service -f
```

If your Pi username/home path differs from `user`/`/home/user`, edit `User=`, `WorkingDirectory=`, and `ExecStart=` in `ble_text_display.service` before installing.

## Local Pi Development With Pi Connect

Enable:
```bash
rpi-connect on
rpi-connect signin
```

Disable:
```bash
rpi-connect off
```
