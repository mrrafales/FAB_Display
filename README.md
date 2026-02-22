# ESP32 LED Matrix Controller — 23×46 WS2811 (12V)

## Project Structure
```
esp32_led_matrix/
├── esp32_led_matrix.ino    ← ESP32 firmware (upload to board)
├── data/
│   └── index.html          ← Web interface (upload to LittleFS)
└── README.md
```

## Hardware Summary
- **ESP32-WROOM-32** microcontroller
- **1058 WS2811 12V LEDs** in a 23×46 matrix
- **4-section parallel output** via GPIOs 16, 17, 18, 19
- **Horizontal serpentine** wiring pattern
- **Level shifter** (3.3V → 5V) on all 4 data lines
- **75% brightness cap** (191/255) for power safety
- **AP mode** by default — no router needed

## 4-Section Layout
```
GPIO 16 →  Rows 0–5   (276 LEDs)  Section 1
GPIO 17 →  Rows 6–11  (276 LEDs)  Section 2
GPIO 18 →  Rows 12–17 (276 LEDs)  Section 3
GPIO 19 →  Rows 18–22 (230 LEDs)  Section 4

Within each section (horizontal serpentine):
  Row 0:  → → → → → → (left to right)
  Row 1:  ← ← ← ← ← ← (right to left)
  Row 2:  → → → → → →
  ...etc
```

## Wiring
```
ESP32 GPIO 16 → Level Shifter Ch1 → Section 1 Data In (Row 0)
ESP32 GPIO 17 → Level Shifter Ch2 → Section 2 Data In (Row 6)
ESP32 GPIO 18 → Level Shifter Ch3 → Section 3 Data In (Row 12)
ESP32 GPIO 19 → Level Shifter Ch4 → Section 4 Data In (Row 18)

Level Shifter LV  → ESP32 3.3V
Level Shifter HV  → 5V (buck converter)
Level Shifter GND → Common Ground

12V Power Supply → WS2811 VCC (with injection every ~100 LEDs)
12V Power Supply → Buck Converter → ESP32 5V/VIN

ALL GROUNDS TIED TOGETHER (ESP32 + PSU + Level Shifter + LEDs)
```

## Power
- 1058 LEDs × 60mA = **63.5A max** @ 12V
- With 75% cap: ~47.6A realistic max
- Recommended: **12V 75–80A supply** (or 2× 40A)
- Power injection every ~100 LEDs to prevent voltage drop

## Required Libraries
1. **FastLED** by Daniel Garcia
2. **ESPAsyncWebServer** by Me-No-Dev (may need GitHub install)
3. **AsyncTCP** by Me-No-Dev (may need GitHub install)

## Upload Steps

### Step 1: Upload Web Interface to LittleFS
- **Arduino IDE 2.x:** Use the [LittleFS Upload Plugin](https://github.com/earlephilhower/arduino-littlefs-upload), then *Tools → ESP32 Sketch Data Upload*
- **PlatformIO:** `pio run --target uploadfs`

### Step 2: Upload Firmware
Select "ESP32 Dev Module" and upload the sketch.

### Step 3: Connect
1. Connect your phone/computer to WiFi network **"LED-Matrix"** (password: `ledmatrix123`)
2. Open **http://192.168.4.1** in a browser
3. The web interface auto-connects via WebSocket — green dot in the header = connected

## Startup Animation
On boot, the matrix sweeps each section in a different color to verify wiring:
- Section 1 (Rows 0–5): Green
- Section 2 (Rows 6–11): Blue
- Section 3 (Rows 12–17): Orange
- Section 4 (Rows 18–22): Purple

If a section doesn't light up or shows the wrong color, check that GPIO's data connection.

## Troubleshooting

**Colors wrong:** Change `COLOR_ORDER` in the `.ino`. Try `GRB`, `BRG`, etc.

**LEDs in wrong order within a row:** Toggle `sectionRow % 2 == 0` logic in `mapLED()` — your serpentine direction may be reversed.

**Section boundary looks offset:** Verify your physical wiring matches the row assignments (Sec1=0-5, Sec2=6-11, Sec3=12-17, Sec4=18-22).

**WebSocket disconnects:** The web interface auto-reconnects every 3 seconds. Check ESP32 serial monitor for error messages.

**Page won't load at 192.168.4.1:** Make sure you uploaded `data/index.html` to LittleFS (Step 1).

**Can I use a single data line instead of 4?** Yes, but you'd be limited to ~31 fps with 1058 LEDs. The 4-section approach gives ~120+ fps headroom, which matters for smooth animations and multi-user responsiveness.
