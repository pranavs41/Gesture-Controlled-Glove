# Gesture-Controlled-Glove
Gesture Controlled glove made for gaming utilizing magnets, IMU, and copper pad circuits to emulate a keyboard keybinds.

A DIY Bluetooth gesture glove for PC gaming. Left-hand glove sends keystrokes to a paired PC based on finger pinches, fist curls, and wrist tilts.

## Status (v1 in progress)
- [x] Pinch detection (thumb to each fingertip)
- [x] BLE keyboard output to PC
- [x] Haptic feedback via vibration motor
- [x] IMU (BNO085) wired, not implemented yet but tested
- [X] Hall sensors for fist detection (awaiting stronger magnets)
- [ ] Wireless battery power (LiPo + TP4057 + MT3608)
- [ ] Final perfboard build

## Hardware
- ESP32 DevKit V1
- BNO085 9-DOF IMU
- 4× AH3144E hall sensors + N42 neodymium magnets
- 5× copper tape pads (fingertips) for pinch detection
- 2N2222A + vibration motor for haptic feedback
- MT3608 boost converter + TP4057 charger + 3.7V LiPo (for wireless)

## Build platform
- Arduino IDE 2.x
- ESP32 core v2.0.17 
- Libraries: ESP32-BLE-Keyboard (T-vK), Adafruit BNO08x

## Wiring
See [hardware.md](hardware.md) for full pin map.

## Gestures
See [gestures.md](gestures.md) for the current gesture-to-key mapping. (Not mapped yet so nothing for now)
