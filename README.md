# XeWe LED - Bed — Alexa-controlled addressable LED lights behind a bed

Personal project · Summer 2021 (uploaded to GitHub 2023-11-05) · Solo: Max Dokukin · Status: Completed

![Bed Lights ESP — lit wall behind the headboard](static/media/resources/IMG_1682.webp)

## Overview

LED lights controlled by Amazon Alexa. An ESP board drives a 382-pixel addressable strip mounted on the back of a bed
headboard and registers itself with Alexa as a smart light named "Bed Lights" through the Espalexa library, which
emulates a Philips Hue device. Because that interface only exposes on/off and brightness, the sketch uses the brightness
value to set the lights mode, due to the library limitations: low values select one of nine colors or animations, and
values from 230 up set the actual brightness. Two of the modes are animated fades driven by FastLED's Perlin noise.

## Highlights

- 382 addressable LEDs on one data pin (`TOTAL_LED_NUM 382`; `SIDE_LED_NUM 75`, `TOP_LED_NUM 232`) — `Bed-Lights-ESP.ino`
- Voice control without a custom Alexa skill: Espalexa device "Bed Lights", brightness value reused as a mode selector — `WifiCommunication.h`
- 9 modes: 7 solid colors and 2 Perlin-noise fades ("purple fade", "blue fade") — `setBedLights()` and `Modes.h`
- Compiles for ESP32 or ESP8266 (`#ifdef ARDUINO_ARCH_ESP32`) — `WifiCommunication.h`
- Whole firmware is 354 lines across 4 files

## How it works

```
"Alexa, set Bed Lights to …" → Alexa → Espalexa (Hue-style device on the LAN) → alexaAction(brightness)
      brightness ≥ 230 → strip brightness 50–255          brightness < 230 → mode = brightness
loop(): espalexa.loop() → setBedLights() → Adafruit NeoPixel strip.show() on pin 4
```

- **`Bed-Lights-ESP.ino`** — `setup()` starts the strip (brightness 255, boot mode 20 = purple fade), connects to Wi-Fi,
  registers the "Bed Lights" device and starts Espalexa; `loop()` services Alexa and redraws the strip every pass.
- **`WifiCommunication.h`** — Wi-Fi station connect (22 polls of 500 ms, then the board restarts) and `alexaAction()`,
  which splits the brightness value into a brightness command (≥ 230, mapped 230–255 → 50–255) or a mode code (< 230).
- **`Modes.h`** — `perlin(hueStart, hueGap, fireStep, minSat)`: FastLED `inoise8` sampled along the strip and advanced
  by 5 per frame, mapped to hue, saturation and value with `strip.ColorHSV`; also an unused random-walk red flicker.
- **`Memory.h`** — writes the last mode (address 0) and brightness (address 1) to the ESP's emulated EEPROM.

### Modes

| Brightness value from Alexa | Mode |
|---|---|
| 5 | red |
| 7 | green |
| 10 | blue |
| 12 | magenta |
| 15 | cyan |
| 17 | yellow |
| 20 | purple fade — `perlin(25000, 35000, 30, 180)` (default at boot) |
| 22 | white |
| 25 | blue fade — `perlin(41000, 5500, 30, 180)` |
| 230–255 | sets brightness (50–255) instead of the mode |
| anything else | white |

The mode codes are Espalexa's 0–255 brightness values; they appear to correspond to about 2 %–10 % in the Alexa app
(and 230+ to about 91 % and up) — check on your device.

## Results

| Metric | Value | Note |
|---|---|---|
| LEDs driven | 382 | `TOTAL_LED_NUM`; side and top constants 75 and 232 (2 × 75 + 232 = 382) |
| Voice-selectable modes | 9 | 7 solid colors, 2 animated noise fades |
| Firmware size | 354 lines, 4 files | `wc -l` |

A home build rather than an experiment: the outcome is the installation in the photos below.

![Strip mounted on the back of the headboard](static/media/resources/IMG_1677.webp)
![Controller mounted inside a second-hand PC power supply](static/media/resources/IMG_1674.webp)

![IMG_1681](https://github.com/xeweva/Bed-Lights-ESP/assets/54597813/afbee172-9095-40ab-9c1b-175023b7f66b)

## Getting started

```text
1. Arduino IDE with the ESP32 (or ESP8266) board package installed
2. Libraries: Espalexa, Adafruit NeoPixel, FastLED
3. Enter your network name and password in WifiCommunication.h (ssid / password) — do not commit them
4. Adjust LED_PIN (4) and TOTAL_LED_NUM (382) in Bed-Lights-ESP.ino for your strip
5. Open Bed-Lights-ESP.ino, select the board and port, upload
6. "Alexa, discover devices" → a light called "Bed Lights" appears
```

Requirements: an ESP32/ESP8266 board, a GRB 800 kHz addressable strip (NeoPixel-compatible) on pin 4 and a power
supply sized for the strip; serial log at 9600 baud. The last mode and brightness are written to EEPROM but are not
committed or read back at boot, so the lights always start in the purple fade.

## Documents

- Photos: [lit wall](static/media/resources/IMG_1682.webp) · [headboard wiring](static/media/resources/IMG_1677.webp) · [controller in the PSU](static/media/resources/IMG_1674.webp)
- Related: [XeWe LED OS](https://maxdokukin.com/projects/xewe-led-os) — project page · [xewe-labs/xewe-led-os](https://github.com/xewe-labs/xewe-led-os) — repository
- Project page: [maxdokukin.com/projects/xewe-led-bed](https://maxdokukin.com/projects/xewe-led-bed)
