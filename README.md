<div align="center">

# 🌫 Air Monitoring System

**Two ESP32s, one wireless air-quality link — sensor pod sends, display pod warns.**

[![Language](https://img.shields.io/badge/C%2B%2B-Arduino-22D3EE?style=for-the-badge&labelColor=0E1626)](https://www.arduino.cc/)
[![MCU](https://img.shields.io/badge/ESP32-Dual%20Node-5A8DFF?style=for-the-badge&labelColor=0E1626)](https://www.espressif.com/en/products/socs/esp32)
[![Build](https://img.shields.io/badge/PlatformIO-espressif32-8A2BE2?style=for-the-badge&labelColor=0E1626)](https://platformio.org/)
[![Link](https://img.shields.io/badge/ESP--NOW-Peer%20to%20Peer-FB7185?style=for-the-badge&labelColor=0E1626)](https://www.espressif.com/en/solutions/low-power-solutions/esp-now)
[![License](https://img.shields.io/badge/License-MIT-34D399?style=for-the-badge&labelColor=0E1626)](./LICENSE)

</div>

---

## 🎯 What is it

A two-node air-quality monitor. An **Air Sensor** node samples particulates, CO₂, temperature and humidity, then unicasts the reading over **ESP-NOW** to an **Air Monitor** node that renders it on a 320×240 TFT (LVGL UI), flashes a WS2811 LED bar in an AQI color, and exposes a TCP socket so a laptop can log everything to text files.

---

## ✨ Features

- 📡 **ESP-NOW** peer-to-peer link between the two ESP32s (no router required)
- 🌬 **PMS3003** particulate sensor — PM1 / PM2.5 / PM10 (µg/m³)
- 🫧 **TGS4161** CO₂ sensor
- 🌡 **DHT22** temperature & humidity
- 🖼 **LVGL** UI on **TFT 320×240** with live arcs for PM2.5, CO₂, temp, humidity
- 💡 **WS2811** 8-LED strip — color-codes air quality (green → yellow → red → purple) with brightness bumped on hazardous readings
- 🔔 Buzzer pin reserved for alert tones
- 🛜 Monitor node also runs as a **Wi-Fi AP** (`ESP-32-Physics`) with a TCP server on `:12345`
- 🐍 `wifi.py` — Python client that connects to the AP, parses the stream, and writes `pm2.5.txt`, `co2.txt`, `temp.txt`

---

## 🔌 Hardware

### 🌬 Air Sensor node

| Role        | Part      | ESP32 pin |
|-------------|-----------|-----------|
| PM sensor   | PMS3003 (UART) | SoftwareSerial RX=21, TX=22 |
| CO₂ sensor  | TGS4161   | ADC GPIO33 |
| Temp / RH   | DHT22     | GPIO32 |

### 📺 Air Monitor node

| Role        | Part            | ESP32 pin |
|-------------|-----------------|-----------|
| Display     | TFT 320×240 (driven by TFT_eSPI) | per `User_Setup.h` |
| LED bar     | WS2811 ×8       | GPIO32 |
| Buzzer      | Piezo           | GPIO33 |
| Wi-Fi AP    | SSID `ESP-32-Physics` / pass `12345678912345` | TCP `:12345` |

> The monitor's MAC is hard-coded in the sensor sketch as the ESP-NOW peer address — replace `broadcastAddress[]` in `Air Sensor/src/main.cpp` if you flash to different boards.

---

## 🚦 LED color scale (PM2.5, µg/m³)

| Range      | Color    | Brightness |
|------------|----------|------------|
| 0–12       | 🟢 Green   | 50 |
| 12–55      | 🟡 Yellow  | 50 |
| 55–250     | 🔴 Red     | 50 |
| > 250      | 🟣 Purple  | 100 |

---

## 🚀 Build & flash

Both folders are independent PlatformIO projects targeting `board = fm-devkit` on the `espressif32` platform.

```bash
git clone https://github.com/Luck-ai/air-monitoring-system
cd air-monitoring-system

# Flash the sensor node
pio run -d "Air Sensor" -t upload

# Flash the monitor node
pio run -d "Air Monitor" -t upload
```

Library dependencies pulled by PlatformIO:

- **Air Sensor** — Adafruit DHT sensor library, Adafruit Unified Sensor
- **Air Monitor** — LVGL, TFT_eSPI, FastLED, ESP-NOW (built-in), DHT (TFT_eSPI's `User_Setup.h` must match your wiring)

---

## 🪵 Log to your laptop

1. Power on both nodes — the monitor brings up the `ESP-32-Physics` Wi-Fi AP.
2. Join it from your laptop.
3. Run the logger:

```bash
python wifi.py
```

It opens a TCP socket to `192.168.4.1:12345` and appends each reading to `pm2.5.txt`, `co2.txt`, and `temp.txt`.

---

## 🗂 Project structure

```text
air-monitoring-system/
├── Air Sensor/             # ESP32 — sensors + ESP-NOW sender
│   ├── platformio.ini
│   ├── include/  lib/  test/
│   └── src/main.cpp
├── Air Monitor/            # ESP32 — TFT (LVGL) + LEDs + ESP-NOW receiver + Wi-Fi AP
│   ├── platformio.ini
│   ├── include/  test/
│   └── src/
│       ├── main.cpp
│       ├── ui*.c           # SquareLine Studio export
│       └── ui_img_*.c      # bundled image assets
└── wifi.py                 # TCP client that logs the stream to text files
```

---

<div align="center">

**Air Monitoring System** · [GitHub](https://github.com/Luck-ai/air-monitoring-system) · [Issues](https://github.com/Luck-ai/air-monitoring-system/issues)

</div>
