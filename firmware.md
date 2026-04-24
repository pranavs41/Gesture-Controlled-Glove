# Firmware

current code for v1 prototype

#include <BleKeyboard.h>

BleKeyboard bleKeyboard("Glove-L", "DIY", 100);

const int FINGER_COUNT = 4;
const int fingerPins[FINGER_COUNT] = {13, 14, 27, 26};
const char fingerKeys[FINGER_COUNT] = {'e', 'r', 'q', 'f'};
const char* fingerNames[FINGER_COUNT] = {"INDEX", "MIDDLE", "RING", "PINKY"};
const int MOTOR_PIN = 16;

bool lastState[FINGER_COUNT] = {true, true, true, true};

void buzz(int ms) {
  digitalWrite(MOTOR_PIN, HIGH);
  delay(ms);
  digitalWrite(MOTOR_PIN, LOW);
}

void setup() {
  Serial.begin(115200);
  for (int i = 0; i < FINGER_COUNT; i++) {
    pinMode(fingerPins[i], INPUT_PULLUP);
  }
  pinMode(MOTOR_PIN, OUTPUT);
  digitalWrite(MOTOR_PIN, LOW);
  bleKeyboard.begin();
  Serial.println("Glove ready. Pair with 'Glove-L' from PC Bluetooth.");
}

void loop() {
  for (int i = 0; i < FINGER_COUNT; i++) {
    bool pressed = (digitalRead(fingerPins[i]) == LOW);
    if (pressed && lastState[i]) {
      Serial.print("PINCH: ");
      Serial.print(fingerNames[i]);
      Serial.print(" -> ");
      Serial.println(fingerKeys[i]);
      if (bleKeyboard.isConnected()) {
        bleKeyboard.press(fingerKeys[i]);
      }
      buzz(40);  // short confirmation buzz
    } else if (!pressed && !lastState[i]) {
      Serial.print("release: ");
      Serial.println(fingerNames[i]);
      if (bleKeyboard.isConnected()) {
        bleKeyboard.release(fingerKeys[i]);
      }
    }
    lastState[i] = !pressed;
  }
  delay(20);
}
