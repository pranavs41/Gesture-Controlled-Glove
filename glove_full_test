/*
 * Gesture Glove v1 - Full System Test
 * 
 * Tests all hardware subsystems on the left-hand glove:
 *  - 4 pinch pads (copper tape fingertips)
 *  - BNO085 IMU over I2C  
 *  - Vibration motor via 2N2222A
 *  - BLE keyboard output
 *  - 4 hall sensors (safe to leave unplugged — will read HIGH)
 * 
 * Target: ESP32 DevKit V1, ESP32 core v2.0.17
 * 
 * Libraries required:
 *   - ESP32 BLE Keyboard (by T-vK)
 *   - Adafruit BNO08x
 *   - Adafruit Unified Sensor
 *   - Wire (built-in)
 */

#include <Wire.h>
#include <BleKeyboard.h>
#include <Adafruit_BNO08x.h>

// ---------- Pin assignments ----------
// Pinch detection (INPUT_PULLUP, pads connect to GND when thumb touches)
const int PIN_INDEX  = 13;
const int PIN_MIDDLE = 14;
const int PIN_RING   = 27;
const int PIN_PINKY  = 26;

// Hall sensors (INPUT_PULLUP, OUT goes LOW when magnet is near)
const int HALL_INDEX  = 32;
const int HALL_MIDDLE = 33;
const int HALL_RING   = 25;
const int HALL_PINKY  = 4;

// Motor driver (GPIO -> 1kΩ -> 2N2222A base)
const int MOTOR_PIN = 16;

// I2C for IMU (default ESP32 pins)
// SDA = GPIO 21
// SCL = GPIO 22

// ---------- Objects ----------
BleKeyboard bleKeyboard("Glove-L", "DIY Gesture Glove", 100);
Adafruit_BNO08x bno08x;
sh2_SensorValue_t imuValue;

// ---------- State tracking ----------
bool pinchState[4] = {false, false, false, false};
bool hallState[4]  = {false, false, false, false};
const int pinchPins[4] = {PIN_INDEX, PIN_MIDDLE, PIN_RING, PIN_PINKY};
const int hallPins[4]  = {HALL_INDEX, HALL_MIDDLE, HALL_RING, HALL_PINKY};
const char* fingerNames[4] = {"INDEX", "MIDDLE", "RING", "PINKY"};
const char pinchKeys[4] = {'e', 'r', 'q', 'f'};

unsigned long lastImuPrint = 0;
const unsigned long IMU_PRINT_INTERVAL = 500;  // ms

// ---------- Helpers ----------
void buzz(int ms) {
  digitalWrite(MOTOR_PIN, HIGH);
  delay(ms);
  digitalWrite(MOTOR_PIN, LOW);
}

void setReports() {
  // Request rotation vector (fused quaternion) at 50Hz
  if (!bno08x.enableReport(SH2_ROTATION_VECTOR, 20000)) {
    Serial.println("IMU: failed to enable rotation vector report");
  }
}

// ---------- Setup ----------
void setup() {
  Serial.begin(115200);
  delay(500);
  Serial.println();
  Serial.println("=== Gesture Glove v1 - Full System Test ===");

  // Pinch pads
  for (int i = 0; i < 4; i++) {
    pinMode(pinchPins[i], INPUT_PULLUP);
  }
  Serial.println("Pinch pads configured (GPIO 13, 14, 27, 26)");

  // Hall sensors
  for (int i = 0; i < 4; i++) {
    pinMode(hallPins[i], INPUT_PULLUP);
  }
  Serial.println("Hall sensors configured (GPIO 32, 33, 25, 4)");

  // Motor
  pinMode(MOTOR_PIN, OUTPUT);
  digitalWrite(MOTOR_PIN, LOW);
  Serial.println("Motor driver configured (GPIO 16)");

  // I2C + IMU
  Wire.begin();
  if (!bno08x.begin_I2C()) {
    Serial.println("IMU: BNO085 NOT FOUND on I2C. Check wiring.");
  } else {
    Serial.println("IMU: BNO085 detected.");
    setReports();
  }

  // BLE keyboard
  bleKeyboard.begin();
  Serial.println("BLE: Advertising as 'Glove-L'. Pair from PC Bluetooth settings.");

  // Startup buzz: 3 short pulses to confirm motor works
  for (int i = 0; i < 3; i++) {
    buzz(60);
    delay(100);
  }

  Serial.println();
  Serial.println("Ready. Actions:");
  Serial.println("  Pinch thumb+finger -> BLE key (e/r/q/f) + short buzz");
  Serial.println("  Hall sensor LOW    -> prints finger curl detected");
  Serial.println("  IMU data           -> printed every 500ms");
  Serial.println();
}

// ---------- Main loop ----------
void loop() {
  // --- Pinch detection ---
  for (int i = 0; i < 4; i++) {
    bool nowPressed = (digitalRead(pinchPins[i]) == LOW);
    if (nowPressed && !pinchState[i]) {
      // Just pressed
      Serial.print("PINCH: ");
      Serial.print(fingerNames[i]);
      Serial.print(" -> '");
      Serial.print(pinchKeys[i]);
      Serial.println("'");
      if (bleKeyboard.isConnected()) {
        bleKeyboard.press(pinchKeys[i]);
      }
      buzz(40);
      pinchState[i] = true;
    } else if (!nowPressed && pinchState[i]) {
      // Just released
      Serial.print("release: ");
      Serial.println(fingerNames[i]);
      if (bleKeyboard.isConnected()) {
        bleKeyboard.release(pinchKeys[i]);
      }
      pinchState[i] = false;
    }
  }

  // --- Hall sensor detection (no BLE action yet, just reporting) ---
  for (int i = 0; i < 4; i++) {
    bool nowCurled = (digitalRead(hallPins[i]) == LOW);
    if (nowCurled && !hallState[i]) {
      Serial.print("CURL: ");
      Serial.println(fingerNames[i]);
      hallState[i] = true;
    } else if (!nowCurled && hallState[i]) {
      Serial.print("uncurl: ");
      Serial.println(fingerNames[i]);
      hallState[i] = false;
    }
  }

  // --- IMU readout (periodic) ---
  if (bno08x.wasReset()) {
    Serial.println("IMU: reset detected, re-enabling reports");
    setReports();
  }
  if (bno08x.getSensorEvent(&imuValue)) {
    if (imuValue.sensorId == SH2_ROTATION_VECTOR &&
        millis() - lastImuPrint > IMU_PRINT_INTERVAL) {
      Serial.print("IMU quat: w=");
      Serial.print(imuValue.un.rotationVector.real, 3);
      Serial.print(" x=");
      Serial.print(imuValue.un.rotationVector.i, 3);
      Serial.print(" y=");
      Serial.print(imuValue.un.rotationVector.j, 3);
      Serial.print(" z=");
      Serial.println(imuValue.un.rotationVector.k, 3);
      lastImuPrint = millis();
    }
  }

  delay(10);
}
