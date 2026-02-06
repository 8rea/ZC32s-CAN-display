# ZC32s-CAN-display
CAN display showing parameters like RPM, Speed, Coolant temp and Acceleration %

## Hardware

For this project I used:

- ESP32 WROOM — main microcontroller that runs the firmware.
- MCP2515 CAN controller — used to connect to the car's CAN network.
- SSD1306 OLED display — used to show the vehicle data.
- Rotary encoder — used to switch between display modes and adjust settings.
- Shift light composed of 10 WS2812 LEDs — used as the RPM/shift indicator.

## Build & Upload

Required libraries (install via Arduino Library Manager):

- Adafruit SSD1306
- Adafruit GFX
- Adafruit NeoPixel
- MCP2515 (MCP2515 CAN controller library)

Steps:

1. Open the Arduino IDE and install the libraries listed above.
2. Select board: `ESP32 Dev Module` (or the ESP32 WROOM variant you use).
3. Connect the ESP32 and upload the sketch located at `src/main.ino`.

Wiring notes (summary):

- MCP2515 CS -> GPIO5, SCK -> GPIO18, MISO -> GPIO19, MOSI -> GPIO23
- SSD1306 I2C -> SDA/SCL (Wire), typically GPIO21 (SDA) and GPIO22 (SCL)
- NeoPixel data -> GPIO15 (use a level shifter if required)
- Rotary encoder -> CLK/S1: GPIO32, DT/S2: GPIO33, SW/KEY: GPIO25

If you prefer PlatformIO, place the sketch in `src/main.ino` and configure a project for `esp32dev`.

## CAN Frames & Decoding

The firmware expects these CAN message IDs and decodes fields as shown below. DLC checks in the code require the minimum bytes indicated.

- RPM
	- CAN ID: `0x124` (DLC >= 3)
	- Raw: `(data[1] << 8) | data[2]`
	- RPM: `RPM = raw / 4`

- Wheel Speed (per-axle, averaged)
	- CAN ID: `0x1B8` (DLC >= 8)
	- raw1: `(data[0] << 8) | data[1]`  (front/left axle)
	- raw2: `(data[2] << 8) | data[3]`  (rear/right axle)
	- Convert: `w1 = raw1 * 0.0276923; w2 = raw2 * 0.0276923`
	- Speed (km/h): `Speed = (w1 + w2) / 2`

- Coolant Temperature
	- CAN ID: `0x310` (DLC >= 2)
	- Temp (°C): `TempC = data[1] - 40`

- Accelerator Pedal
	- CAN ID: `0x122` (DLC >= 2)
	- Pedal %: `Pedal% = data[1] * 100.0 / 255.0`

These match the decoding logic implemented in `src/main.ino`.
