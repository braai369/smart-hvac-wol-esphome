# 📡 Smart HVAC UART Control & Intelligent WoL Gateway

Hardware reverse-engineering project and custom firmware integration for controlling split HVAC units (**Philco / Midea UART display protocol**) natively inside **Home Assistant** using ESP8266/ESP32 microcontrollers flashed with **ESPHome**, alongside an automated Wake-on-LAN (WoL) server gateway.

---

## 📌 Project Overview

Commercial smart HVAC adapters are often expensive, closed-source, or cloud-dependent. This project bypasses cloud lock-in by interfacing directly with the internal UART serial lines of the indoor unit's display board, giving local, bi-directional control over temperature, fan speed, operating modes, and internal sensors.

Additionally, the microcontroller acts as a low-power, always-on **Wake-on-LAN (WoL)** trigger node for remote homelab server power management.

---

## 🏗️ Hardware Architecture & Wiring

```
┌──────────────────────────────────────┐
│       Philco / Midea AC Unit         │
│     (Indoor Board / Display Port)    │
└──────────────────┬───────────────────┘
                   │  (UART 5V RX/TX + 5V VCC)
                   ▼
┌──────────────────────────────────────┐
│  Logic Level Shifter / Optocoupler   │ (5V <-> 3.3V Signal Matching)
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│      D1 Mini / ESP8266 Node          │
│       (Flashed with ESPHome)         │
└──────────────────┬───────────────────┘
                   │
                   ▼ (Native API / Wi-Fi)
┌──────────────────────────────────────┐
│      Home Assistant Automation       │
└──────────────────────────────────────┘
```

---

## ✨ Features & Hardware Specs

* **🔌 Native Local Control:** Zero-cloud dependency; total local control via ESPHome Native API.
* **⚡ Bi-Directional State Sync:** Reads real-time state changes made from the original IR physical remote control.
* **🖥️ Smart Wake-on-LAN (WoL):** Sends magic packets via local Ethernet/Wi-Fi to wake sleeping homelab nodes or workstation PCs.
* **🌡️ Thermal Monitoring:** Exposes indoor room temperatures and target setpoints directly to Home Assistant dashboards.

---

## 🛠️ Technical Stack

* **Hardware:** Wemos D1 Mini (ESP8266) / ESP32, Logic Level Shifters, Custom PCB/Peridex Headers.
* **Firmware Platform:** ESPHome Framework (YAML configuration + C+ Custom Components).
* **Protocols:** UART Serial (9600 baud), Wi-Fi 802.11 b/g/n, WoL UDP Magic Packet.
* **Integration:** Home Assistant Native API / MQTT.

---

## ⚙️ ESPHome Configuration Example (`hvac-node.yaml`)

```yaml
esphome:
  name: ac-living-room
  platform: ESP8266
  board: d1_mini

  # --- DISPARADOR DE WOL AL RECONECTAR ---
  on_connect:
    then:
      - button.press: wake_debian_server

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

captive_portal:

# --- COMPONENTE DE BOTONES (WOL) ---
button:
  - platform: wake_on_lan
    target_mac_address: "AA:BB:CC:DD:EE:FF" # MAC Address del servidor
    name: "Wake Debian Server"
    id: wake_debian_server

# --- CONFIGURACIÓN UART ---
uart:
  id: ac_uart
  tx_pin: GPIO5
  rx_pin: GPIO4
  baud_rate: 9600

# --- COMPONENTE CLIMATE NATIVO ---
climate:
  - platform: midea
    name: "AC Philco Inverter"
    uart_id: ac_uart
    supported_modes:
      - 'HEAT_COOL'
      - 'COOL'
      - 'HEAT'
      - 'DRY'
      - 'FAN_ONLY'
    custom_fan_modes:
      - 'SILENT'
      - 'TURBO'
    supported_swing_modes:
      - 'VERTICAL'
```

---

## 👤 Author

* **Brian Alexander Salvagni Orozco** (`braai369`)
* *AI Specialist & Process Automation Engineer*
