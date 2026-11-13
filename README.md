# Atorch DL24 Bluetooth Monitor

Single-file web app that connects to an **Atorch DL24** USB-C power meter over Bluetooth Low Energy (Web Bluetooth API) and plots **Voltage, Current, Power, Capacity, and Temperature vs Time** live in the browser. Energy (Wh) is shown on a readout card but not plotted or exported.

**Live demo:** https://f1chter.github.io/lynk-atorch/

## Features

- Connect to Atorch DL24 via Web Bluetooth (Chrome / Edge)
- Parses the DL24 binary telemetry protocol (`0xFF 0x55` header, report packets)
- Live readouts: Voltage (V), Current (A), Power (W), Energy (Wh), Capacity (Ah), Temperature (°C), Time (HH:MM:SS)
- Real-time line chart: four parameters (V, A, Ah, °C) vs Time using packet timestamps
- Chart clear, CSV export (Power and Energy excluded)
- GATT service/characteristic discovery panel
- Pre-filled DL24 UUIDs: Service `0000ffe0…`, Characteristic `0000ffe1…`

## Protocol

The app parses the DL24 report packet format (Device Type `0x02` = DC Meter):

| Offset | Field | Size | Endian | Coefficient | Source |
|--------|-------|------|--------|-------------|--------|
| 0 | Magic Header | 2 bytes | — | `0xFF 0x55` | Packet |
| 2 | Message Type | 1 byte | — | `0x01` = Report | Packet |
| 3 | Device Type | 1 byte | — | `0x02` = DC Meter | Packet |
| 4 | Voltage | 3 bytes | Big-endian | ÷ 10 | Packet → V |
| 7 | Current | 3 bytes | Big-endian | ÷ 1000 | Packet → A |
| 0x0A | Capacity (Ah) | 3 bytes | Big-endian | ÷ 100 | Packet → Ah |
| 0x0D | Energy (Wh) | 4 bytes | Big-endian | × 10 | Packet → Wh (card only, not plotted) |
| 0x18 | Temperature | 2 bytes | Big-endian (signed) | × 1 | Packet → °C |
| 0x1A | Hour | 2 bytes | Big-endian | × 1 | Packet → HH |
| 0x1C | Minute | 1 byte | — | × 1 | Packet → MM |
| 0x1D | Second | 1 byte | — | × 1 | Packet → SS |
| last | Checksum | 1 byte | — | `(sum of bytes 2..len-2) & 0xFF ^ 0x44` | Packet |

### Derived / Calculated fields

| Field | Calculation | Source |
|-------|-------------|--------|
| **Power (W)** | `Voltage (V) × Current (A)` | Calculated from packet Voltage and Current |
| **Time (HH:MM:SS)** | `Hour:Minute:Second` | Calculated from packet time fields |

Protocol reference: https://github.com/CursedHardware/atorch-console/blob/master/docs/protocol-design.md

## Requirements

- Chrome, Edge, or another browser with **Web Bluetooth** support
- Served over a secure context: `https://` or `http://localhost`
- Android, Windows, macOS, or Linux (Web Bluetooth is not supported on iOS)

## Running locally

1. Clone or download this repository:
   ```bash
   git clone <your-repo-url>
   cd lynk-atorch
   ```

2. Serve the folder over HTTP (Web Bluetooth requires localhost/https):
   ```bash
   python3 -m http.server 8000
   ```
   Or with Node.js:
   ```bash
   npx http-server -p 8000
   ```

3. Open in your browser:
   ```
   http://localhost:8000
   ```

4. Click **Connect**, select your `DL24_BLE` device from the Bluetooth picker. Voltage, Current, Power, Energy, Capacity, and Temperature will plot automatically every second.

## Usage

| Control | Action |
|---------|--------|
| **Connect** | Open the Bluetooth device picker and pair with the DL24 |
| **Clear** | Reset the chart and clear all samples |
| **Export CSV** | Download `dl24_telemetry.csv` with all measured and derived fields |
| **Discover Services** | List all GATT services and characteristics for debugging |

## Notes

- The DL24 streams telemetry notifications automatically (~1 Hz), so data appears immediately after connecting.
- Power is read directly from packet offset `0x0A` (÷10), not calculated as V×A.
- Energy is read directly from packet offset `0x0D` (÷100).
- Capacity (Ah) is derived as Energy (Wh) ÷ Voltage (V).
- The characteristic on most DL24 units is notify-only.

## License

MIT