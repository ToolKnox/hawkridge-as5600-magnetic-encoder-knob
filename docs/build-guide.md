# AS5600 Magnetic Encoder Knob with ESP32 T-Display - Build Guide

https://github.com/ToolKnox/hawkridge-as5600-magnetic-encoder-knob

Remix of AS5600Knob by Scott Bezek (https://github.com/scottbez1/AS5600Knob), updated/refurbished for modern use. Print files unchanged.

## What you are building

A desk knob that knows its absolute angle. A small magnet sits in the printed knob, the knob turns on a 608 skateboard bearing, and an AS5600 magnetic angle sensor underneath reads the magnet. An ESP32 LILYGO TTGO T-Display shows the angle in degrees with a pointer and a colour that follows the angle, and prints the angle to the serial port. It is a building block for a volume knob, a camera dial or a robot joint encoder.

Parts are in the Bill of Material PDF. You also need a computer with VS Code + PlatformIO, a soldering iron and a 3D printer.

## 1. Flash the firmware first

Test the board before it disappears into the knob.

1. Install VS Code and the PlatformIO extension.
2. Download the upstream project at the pinned commit: https://github.com/scottbez1/AS5600Knob/tree/5c34c24576ac14bb009d6ec1f84694211dc64253 (Code > Download ZIP) and open the folder in PlatformIO.
3. The project has one environment, `tdisplay`. It builds for `esp32doit-devkit-v1` and sets the T-Display screen pins itself (ST7789, 135 x 240, MOSI 19, SCLK 18, CS 5, DC 16, RST 23, backlight 4) and pins the display library to `TFT_eSPI@2.3.59`. The AS5600 library is already included in `lib/`.
4. Plug the T-Display in with a USB data cable and run **Upload** (`pio run -e tdisplay -t upload`).
5. Open the serial monitor at **921600 baud** (`pio device monitor`). The screen should fill with a colour and show a number and a white pointer; until the sensor is wired the number is not a real angle.

Note: the project does not pin the `espressif32` platform version. If a current platform release fails to build it, pin an older `espressif32` release in `platformio.ini` and build again.

## 2. Print the parts

| File | Notes |
|---|---|
| AS5600BasePlate.stl | 0.2 mm layers |
| AS5600BearingMount.stl | 0.15 mm layers for the small tabs |
| AS5600Knob.stl | 0.2 mm layers; put the seam at the rear so it lines up with the index notch |

PLA is fine. The dimensions were tuned on the designer's printer, so test-fit the bearing and magnet before final assembly; a little sanding or a slower outer wall can help.

## 3. Wire the sensor (3.3 V only)

Solder the header that comes with the AS5600 module, then use four female-female jumper leads:

| AS5600 module | T-Display |
|---|---|
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO 21 |
| SCL | GPIO 22 |

- Power the module from **3.3 V**. The module carries its own I2C pull-up resistors; at 5 V they pull SDA and SCL to 5 V, which can damage the ESP32. The designer notes that some modules work more reliably after removing R1 (which sets them for 5 V), but then the sensor needs 5 V and a level shifter - skip that unless you have trouble.
- The firmware uses the ESP32 default I2C pins (`Wire.begin()` with no arguments), which are GPIO 21 (SDA) and GPIO 22 (SCL) on this board definition.
- If your module brings out a DIR pin, tie it to GND so the counting direction is fixed.

## 4. Assemble the knob

1. Press the 4 x 2 mm magnet into the centre of the knob. It is an axial magnet (poles on the flat faces); the designer used it because his module did not include a proper diametric magnet.
2. Press the 608 bearing into the bearing mount and push the knob's centre post through it.
3. Stack base plate, AS5600 board (chip facing up, under the magnet) and bearing mount, and fix them with four M3 x 6 mm countersunk screws. They thread straight into the plastic of the base plate, so tighten gently.
4. The parts list includes one small (about 2 mm wide) cable tie; use it as shown in the upstream assembly view and video to secure the stack.
5. Turn the knob: it should spin freely with the magnet centred over the sensor chip and not touching it.

Optional upgrades from the designer's notes: a thicker base plate with heat-set inserts or captive nuts for stronger screws, or a smaller knob if you drop the outer mounting holes.

## 5. First run and check

1. Plug in USB. The screen shows the angle in degrees, a white pointer and a background colour that changes with the angle.
2. In the serial monitor at 921600 baud, each line shows the angle in degrees (-180 to 180) and the sensor's AGC (automatic gain) value.
3. Turn the knob slowly. The angle should change smoothly over a full turn. The firmware averages the reading (a running average with factor 0.8), so it lags very slightly.
4. If the angle jumps or does not move: check SDA/SCL, check that the magnet is centred and 1-3 mm above the chip, and watch the AGC value - a value pinned at one end of its range means the magnet is too close or too far.

## Credits

Original design, firmware and photos: Scott Bezek, AS5600Knob (https://github.com/scottbez1/AS5600Knob), Apache License 2.0 (http://www.apache.org/licenses/LICENSE-2.0). The bundled AS5600 library is from Seeed Studio (MIT). This guide is new text written for this remix; the print files are unchanged.
