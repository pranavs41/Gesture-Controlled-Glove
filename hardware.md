ESP32 DevKit V1 pin assignments (Left-hand glove v1)
===================================================

Pinch detection (copper pads, INPUT_PULLUP):
  GPIO 13 → Index fingertip
  GPIO 14 → Middle fingertip
  GPIO 27 → Ring fingertip
  GPIO 26 → Pinky fingertip
  GND     → Thumb fingertip

IMU (BNO085 over I²C):
  GPIO 21 → SDA
  GPIO 22 → SCL
  3V3     → VIN
  GND     → GND

Motor driver (via 1kΩ → 2N2222A base):
  GPIO 16 → Transistor base (through 1kΩ)
  5V      → Motor (+) via flyback diode
  Collector → Motor (-)
  Emitter   → GND

Hall sensors (INPUT_PULLUP, powered 5V):
  GPIO 32 → Index sensor OUT
  GPIO 33 → Middle sensor OUT
  GPIO 25 → Ring sensor OUT
  GPIO 4  → Pinky sensor OUT
