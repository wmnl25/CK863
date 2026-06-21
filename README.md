# CK863 / INSMART Smart Coffee Scale BLE Protocol

This repository documents the reverse-engineered Bluetooth Low Energy (BLE) protocol for the CK863 smart coffee scale (often sold under the brand name INSMART). 

Included is a vanilla JavaScript Web App (`index.html`) utilizing the **Web Bluetooth API** to connect to the scale and parse real-time data directly in the browser.

<img width="200" height="200" alt="CK863" src="https://github.com/user-attachments/assets/d883b5d6-5c80-4621-9d55-b8ad81d0344f" />


## BLE Profile Overview

The scale communicates its data via a custom (non-standardized) GATT Service and Characteristic.

* **Primary Service UUID:** `0xFFF0`
* **Command TX (Write):** `0xFFF1` / `0xFFF2` (For remote tare/control - *Requires further mapping*)
* **Data RX (Notify):** `0xFFF3` (Streams weight, unit, and timer data)

## Protocol Data Structures (RX)

By subscribing to the `0xFFF3` characteristic, the scale streams packets continuously. The packet structure changes entirely depending on the physical mode the scale is currently in.

### 1. Standard Weighing Mode
When in normal weighing mode, the scale sends an 8-byte packet. 

| Byte Index | Hex Example | Description |
| :--- | :--- | :--- |
| **0** | `0x12` | Header / Mode Indicator (Standard Mode) |
| **1-3** | `0x06 0x05 0x00` | Status flags (Stabilization, battery, etc.) |
| **4-5** | `0xD4 0x00` | **Weight:** Little-Endian Signed Int. Divide by 10. (e.g., `0x00D4` = 212 = 21.2) |
| **6** | `0x05` | **Unit Indicator:** See Unit Map below. |
| **7** | `0x00` | Footer |

### 2. Espresso / Timer Mode
When switched to Espresso mode, the packet expands to 19 bytes to accommodate the built-in timer. 

| Byte Index | Hex Example | Description |
| :--- | :--- | :--- |
| **0** | `0x14` | Header / Mode Indicator (Espresso Mode) |
| **1-3** | `0x11 0x12 0x06` | Status flags |
| **4** | `0x05` | **Unit Indicator:** See Unit Map below. |
| **5** | `0x00` | *Reserved* |
| **6** | `0x00` | **Timer:** Minutes |
| **7** | `0x17` | **Timer:** Seconds (e.g., `0x17` = 23 seconds) |
| **8-10** | `0x00 0x00 0x00`| *Reserved* |
| **11-12**| `0x13 0x07` | **Weight:** Little-Endian Signed Int. Divide by 10. (e.g., `0x0713` = 1811 = 181.1) |
| **13-16**| `0x00...` | *Reserved* |
| **17-18**| `0x4C 0x06` | Checksum / Footer |

### Unit Map (Applicable to both modes)
The unit byte translates as follows:
* `0x05` = Grams (g)
* `0x08` = Milliliters (ml)
* `0x0A` = Fluid Ounces (fl oz)
* `0x07` = Ounces (oz)
* `0x01` = Pounds (lb)

## Running the Web App

1. Simply serve `index.html` via a local web server, or host it via **GitHub Pages**. 
   *(Note: The Web Bluetooth API requires a secure context, meaning it will only work over `https://` or `localhost`).*
2. Click **Connect to Scale**.
3. Select your scale from the browser's pairing menu.
4. The UI will instantly display the live weight, the active unit, and the timer (if in espresso mode).

<img width="531" height="442" alt="Scherm­afbeelding 2026-06-21 om 12 50 49" src="https://github.com/user-attachments/assets/da658bb3-b8af-4807-aeea-51fb8ffdc72e" />


## Acknowledgments
The mappings in this repository were discovered via manual byte-sniffing and reverse-engineering. Feel free to use this data to build your own ESP32, Raspberry Pi, or Home Assistant integrations. Pull requests mapping the TX commands (tare, start timer) on `0xFFF1` are welcome!
