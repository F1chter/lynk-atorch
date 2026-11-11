# Atorch DL24 Bluetooth Monitor

Single-file web app that connects to an **Atorch DL24** USB-C power meter over Bluetooth Low Energy (Web Bluetooth API) and plots **Voltage and Current vs Time** live in the browser.

## Features

- Connect to Atorch DL24 via Web Bluetooth (Chrome / Edge)
- Parses the DL24 binary telemetry protocol (`0xFF 0x55` header, report packets)
- Live readouts: Voltage (V), Current (A), Power (W)
- Real-time line chart: Voltage vs Time + Current vs Time
- Polling interval control, manual read, chart clear, CSV export
- GATT service/characteristic discovery panel
- Pre-filled DL24 UUIDs: Service `0000ffe0…`, Characteristic `0000ffe1…`

## Protocol

The app parses the DL24 report packet format:

| Offset | Field | Size | Note |
|--------|-------|------|------|
| 0 | Magic Header | 2 bytes | `0xFF 0x55` |
| 2 | Message Type | 1 byte | `0x01` = Report |
| 3 | Device Type | 1 byte | `0x02` = DC, `0x03` = USB |
| 4 | Voltage | 3 bytes | 24-bit big-endian, /100 |
| 7 | Current | 3 bytes | 24-bit big-endian, /100 |
| last | Checksum | 1 byte | `(sum of bytes 2..len-2) & 0xFF ^ 0x44` |

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

4. Click **Connect**, select your `DL24_BLE` device from the Bluetooth picker, then click **Start**. Voltage and Current will plot automatically every second.

## Usage

| Control | Action |
|---------|--------|
| **Connect** | Open the Bluetooth device picker and pair with the DL24 |
| **Start / Stop** | Begin or stop automatic polling |
| **Read Once** | Request a single telemetry read (if the characteristic supports read) |
| **Clear** | Reset the chart and clear all samples |
| **Export CSV** | Download `dl24_telemetry.csv` with time, voltage, current, power |
| **Discover Services** | List all GATT services and characteristics for debugging |
| **Polling interval** | Set how often to poll (ms) |

## Notes

- The DL24 streams telemetry notifications automatically (~1 Hz), so data appears even without clicking Start.
- If values look wrong, use the **GATT UUIDs** and **Value Parser** panels to adjust scale or endianness for your firmware revision.
- The characteristic on most DL24 units is notify-only; `Read Once` may report "not readable" — this is normal.

## License

MIT