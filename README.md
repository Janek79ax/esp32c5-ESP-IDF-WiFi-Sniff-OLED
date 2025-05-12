# ⚠️ Proof of Concept – Passive Wi-Fi Sniffer

> This is a **proof-of-concept** project that demonstrates how to passively detect Wi-Fi clients near ESP32-C5 using promiscuous mode.  
> Use responsibly. In many jurisdictions, **sniffing Wi-Fi traffic** may require **authorization** or **consent** from network owners. This code is for **educational purposes only**.


## ESP32-C5 WiFi Client Sniffer

This project scans nearby Wi-Fi access points and detects how many clients are connected to each individual BSSID using an ESP32-C5 chip.  
It operates in a loop: scans visible APs, then switches into promiscuous (sniffer) mode and captures data frames to count clients per BSSID.

---

## 🔧 Requirements

- **ESP32-C5 DevKitC-1** board
- **ESP-IDF v5.5.0+** (master branch)
- **Python 3.11+**
- A serial terminal (e.g., `idf.py monitor` or `minicom`)

---

## 📦 Setup

1. Install [ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/latest/esp32c5/get-started/index.html) with version **5.5.0 or later** (required for ESP32-C5 support).
2. Set target to `esp32c5`:

```bash
idf.py --preview set-target esp32c5
```

3. Build and flash:

```bash
idf.py fullclean build flash monitor
```

Make sure the board is in download mode and connected to the right COM port (`idf.py -p COMx ...`).

---

## 📡 Output Example

Sample scan result with client count per BSSID after 3 seconds of sniffing each:

```
| SSID              | Band | Chan  | Clients | BSSID             |
|-------------------|------|-------|---------|-------------------|
| Home5G            | 5G   | 128   | 0       | 7C:2B:8F:AA:BB:01 |
|                   | 5G   | 128   | 1       | 7C:2B:8F:AA:BB:02 |
|                   | 5G   | 128   | 1       | 7C:2B:8F:AA:BB:03 |
|                   | 5G   | 128   | 0       | 7C:2B:8F:AA:BB:04 |
|                   | 5G   | 128   | 0       | 7C:2B:8F:AA:BB:05 |
| Home2.4G          | 2.4G | 11    | 0       | 7C:2B:8F:AA:BC:01 |
|                   | 2.4G | 11    | 2       | 7C:2B:8F:AA:BC:02 |
|                   | 2.4G | 11    | 1       | 7C:2B:8F:AA:BC:03 |
|                   | 2.4G | 11    | 0       | 7C:2B:8F:AA:BC:04 |
|                   | 2.4G | 11    | 0       | 7C:2B:8F:AA:BC:05 |
```

---

## 🧠 How it works

1. Performs a Wi-Fi scan (active scan) and collects AP info (SSID, channel, BSSID, RSSI)
2. For each AP, switches to its channel and enters promiscuous mode
3. Sniffs for `data` frames and counts unique client MAC addresses seen communicating with that BSSID
4. After scanning all APs, displays a table in the console

---

## 📁 Structure

- `main.c` – core logic: scanning, sniffing, display
- Uses ESP-IDF Wi-Fi APIs (`esp_wifi_...`) and `esp_wifi_set_promiscuous_rx_cb()` for sniffer mode

---

## 🧭 Optional: GPS Integration (ATGM336H)

This project optionally supports **GPS position logging** via the **ATGM336H** module.  
The device outputs NMEA sentences (e.g. `$GPGGA`) over UART and a PPS signal for accurate timing.

### 🔌 Wiring (ESP32-C5 DevKitC-1)

| GPS Pin   | Connect to ESP32-C5 |
|-----------|---------------------|
| **TX**    | GPIO24              |
| **PPS**   | GPIO23              |
| **VCC**   | 3.3V                |
| **GND**   | GND                 |
| **RX**    | _Not connected_     |

> 📌 Only GPS TX (UART output) and PPS (Pulse-Per-Second) are used. GPS RX is not required.

### ⚙️ Features

- Automatically detects if GPS is present and active
- Displays latest latitude/longitude if available
- Shows last PPS timestamp (in milliseconds)
- If GPS is not detected, displays "GPS: N/A"

### 💡 Notes

- UART baud rate: **9600**
- PPS signal must be active (1 pulse per second) on rising edge
- GPS fix may take time (especially indoors)

---

## 📍 Notes

- Limited to max 10 APs and 10 clients per BSSID (changeable via `#define`)
- Sniffing duration per AP is configurable (default: 3000ms)
- Requires active traffic to detect clients — idle clients may not be seen

---

## 🛠️ TODO

- [ ] Display summary on 1.8" SPI TFT screen (ST7735S)
- [ ] Add rotary encoder support for navigating results
- [ ] Live update of top APs with most clients
- [X] GPS support



Apache License 2.0 – see [`LICENSE`](LICENSE) file for full details.
---
