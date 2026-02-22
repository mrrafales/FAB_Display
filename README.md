# ESP32 LED Matrix Controller — 23×46 WS2811

## Project Structure
```
esp32_led_matrix/
├── esp32_led_matrix.ino    ← ESP32 firmware (upload to board)
├── data/
│   └── index.html          ← Web interface (upload to LittleFS)
└── README.md
```

## Required Libraries
Install via **Arduino Library Manager** (Sketch → Include Library → Manage Libraries):
1. **FastLED** by Daniel Garcia
2. **ESPAsyncWebServer** by Me-No-Dev  
3. **AsyncTCP** by Me-No-Dev

> **Note:** ESPAsyncWebServer and AsyncTCP may need to be installed from GitHub:
> - https://github.com/me-no-dev/ESPAsyncWebServer  
> - https://github.com/me-no-dev/AsyncTCP  
> Download the ZIP and use *Sketch → Include Library → Add .ZIP Library*

## Wiring Diagram
```
ESP32                WS2811 LED Strip
─────                ───────────────
GPIO 16 ──[330Ω]──→ DATA IN
GND ──────────────→ GND
                     
5V Power Supply      WS2811 LED Strip
───────────────      ───────────────
+5V ──────────────→ VCC (+5V)
GND ──────────────→ GND
                     
ESP32 GND ←──────→ Power Supply GND  (IMPORTANT: common ground!)
```

### Power Considerations
- 1058 WS2811 LEDs × 60mA max = **~63A at full white!**
- Typical mixed-color usage: **20–30A** is usually sufficient
- Use a **5V power supply rated for your expected load**
- **Inject power** every 100–150 LEDs to prevent voltage drop
- Add a **1000µF capacitor** across +5V and GND near the LEDs

## Upload Steps

### Step 1: Configure WiFi
Edit `esp32_led_matrix.ino` and change:
```cpp
const char* WIFI_SSID     = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
```

Or set `AP_MODE = true` to create a standalone WiFi access point.

### Step 2: Configure LED Wiring (if needed)
```cpp
#define LED_PIN         16        // Change if using different GPIO
#define SERPENTINE      true      // true for zig-zag wiring, false for parallel
#define FIRST_ROW_AT_BOTTOM  false  // true if row 0 is at the bottom
#define COLOR_ORDER     RGB       // Change to GRB, BRG, etc. if colors are swapped
```

### Step 3: Upload the Web Interface to LittleFS

**Arduino IDE 2.x:**
1. Install the [LittleFS Upload Plugin](https://github.com/earlephilhower/arduino-littlefs-upload)
2. Make sure `data/index.html` is in the sketch folder
3. Use *Tools → ESP32 Sketch Data Upload*

**PlatformIO:**
1. Place the `data/` folder in your project root
2. Run: `pio run --target uploadfs`

### Step 4: Upload the Firmware
1. Select your ESP32 board in Arduino IDE
2. Select the correct COM port
3. Click Upload

### Step 5: Connect!
1. Open Serial Monitor at 115200 baud to see the IP address
2. Open that IP in a web browser on the same network
3. The web interface should auto-connect via WebSocket

## Communication Protocol

### WebSocket (ws://[IP]/ws)

**Binary commands (web → ESP32):**
| Cmd Byte | Payload | Description |
|----------|---------|-------------|
| `0x01` | 3174 bytes (1058 × RGB) | Full frame update |
| `0x02` | N × (index_hi, index_lo, R, G, B) | Partial update |
| `0x03` | 1 byte (brightness 0–255) | Set brightness |
| `0x04` | (none) | Clear all LEDs |
| `0x05` | R, G, B | Fill solid color |
| `0xFF` | (none) | Ping/keepalive |

**Text commands (web → ESP32):**
- `status` — Request status
- `ping` — Keepalive
- `info` — Request device info
- `brightness:N` — Set brightness (0–255)
- `clear` — Clear all LEDs

**REST API (for external tools):**
- `GET /api/status` — JSON status
- `POST /api/clear` — Clear LEDs
- `POST /api/brightness?value=N` — Set brightness

## Web Interface Features
- **Auto-Sync**: When checked, every visual change is pushed to hardware in real-time (~30fps max)
- **Push Frame**: Manually push the current design to the LEDs
- **Hardware Brightness**: Separate from the drawing brightness — controls actual LED output
- **Clear HW**: Turn off all physical LEDs without affecting the design
- **Hardware Panel**: Shows ESP32 connection info (IP, free RAM, FPS)
- All original drawing tools, text layers, VU meters, images, and fundraiser mode work as before

## Troubleshooting

**Colors look wrong:** Change `COLOR_ORDER` in the `.ino` file. Try `GRB`, `BRG`, etc.

**LEDs are in the wrong order:** Toggle `SERPENTINE` between `true` and `false`. The startup animation (green sweep) will help you see the wiring pattern.

**First row shows at the bottom:** Set `FIRST_ROW_AT_BOTTOM = true`.

**Can't connect to WiFi:** The ESP32 will fall back to AP mode after 30 seconds. Connect to the "LED-Matrix" network (password: `ledmatrix123`).

**Web page won't load:** Make sure you uploaded the `data/index.html` to LittleFS (Step 3).

**WebSocket disconnects frequently:** The web interface auto-reconnects every 3 seconds. Check that the ESP32 has stable power and WiFi signal.
