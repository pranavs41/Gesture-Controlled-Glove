#include <Wire.h>
#include <BleKeyboard.h>
#include <Adafruit_BNO08x.h>

BleKeyboard bleKeyboard("Glove-L", "DIY Gesture Glove", 100);
Adafruit_BNO08x bno08x;
sh2_SensorValue_t imuValue;

const int PINCH_INDEX  = 13;
const int PINCH_MIDDLE = 14;
const int PINCH_RING   = 27;
const int PINCH_PINKY  = 26;

const int HALL_INDEX = 32;
const int HALL_RING  = 25;

const float ROLL_NEUTRAL = 5.6f;
const float PITCH_NEUTRAL = -21.2f;
const float STRAFE_THRESHOLD = 40.0f;
const float WALK_THRESHOLD = 40.0f;

float currentRoll = 0.0f;
float currentPitch = 0.0f;
bool gotImuData = false;

bool isStrafingLeft = false;
bool isStrafingRight = false;
bool isWalkingFwd = false;
bool isWalkingBack = false;

bool isJumping = false;
bool isSprinting = false;
bool wasMiddle = false;
bool wasRing = false;
bool wasPinky = false;

bool wasFist = false;
unsigned long lastFistTrigger = 0;
const unsigned long FIST_DEBOUNCE_MS = 1000;

void setup() {
  Serial.begin(115200);
  delay(500);
  
  pinMode(PINCH_INDEX, INPUT_PULLUP);
  pinMode(PINCH_MIDDLE, INPUT_PULLUP);
  pinMode(PINCH_RING, INPUT_PULLUP);
  pinMode(PINCH_PINKY, INPUT_PULLUP);
  pinMode(HALL_INDEX, INPUT_PULLUP);
  pinMode(HALL_RING, INPUT_PULLUP);
  
  Wire.begin();
  delay(100);
  
  if (!bno08x.begin_I2C(0x4B)) {
    Serial.println("BNO085 not found");
  } else {
    bno08x.enableReport(SH2_ROTATION_VECTOR, 50000);
    Serial.println("BNO085 ready");
  }
  
  bleKeyboard.begin();
  Serial.println("Glove ready. Pair 'Glove-L'.");
}

void loop() {
  // === IMU ===
  if (bno08x.wasReset()) {
    bno08x.enableReport(SH2_ROTATION_VECTOR, 50000);
  }
  
  if (bno08x.getSensorEvent(&imuValue)) {
    if (imuValue.sensorId == SH2_ROTATION_VECTOR) {
      float qw = imuValue.un.rotationVector.real;
      float qx = imuValue.un.rotationVector.i;
      float qy = imuValue.un.rotationVector.j;
      float qz = imuValue.un.rotationVector.k;
      
      float sinr = 2.0f * (qw * qx + qy * qz);
      float cosr = 1.0f - 2.0f * (qx * qx + qy * qy);
      currentRoll = atan2(sinr, cosr) * 180.0f / 3.14159265f;
      
      float sinp = 2.0f * (qw * qy - qz * qx);
      if (sinp >= 1.0f) sinp = 1.0f;
      if (sinp <= -1.0f) sinp = -1.0f;
      currentPitch = asin(sinp) * 180.0f / 3.14159265f;
      
      gotImuData = true;
    }
  }
  
  // === Tilts (with coordinate transform) ===
  if (gotImuData) {
    float pitchDelta = currentPitch - PITCH_NEUTRAL;
    float rollDelta = currentRoll - ROLL_NEUTRAL;
    float strafeSignal = pitchDelta + rollDelta;
    float walkSignal = pitchDelta - rollDelta;
    
    bool wantLeft = (strafeSignal < -STRAFE_THRESHOLD);
    bool wantRight = (strafeSignal > STRAFE_THRESHOLD);
    bool wantFwd = (walkSignal > WALK_THRESHOLD);
    bool wantBack = (walkSignal < -WALK_THRESHOLD);
    
    if (wantLeft && !isStrafingLeft) {
      if (bleKeyboard.isConnected()) bleKeyboard.press('a');
      isStrafingLeft = true;
      Serial.println("LEFT (A)");
    } else if (!wantLeft && isStrafingLeft) {
      if (bleKeyboard.isConnected()) bleKeyboard.release('a');
      isStrafingLeft = false;
    }
    
    if (wantRight && !isStrafingRight) {
      if (bleKeyboard.isConnected()) bleKeyboard.press('d');
      isStrafingRight = true;
      Serial.println("RIGHT (D)");
    } else if (!wantRight && isStrafingRight) {
      if (bleKeyboard.isConnected()) bleKeyboard.release('d');
      isStrafingRight = false;
    }
    
    if (wantFwd && !isWalkingFwd) {
      if (bleKeyboard.isConnected()) bleKeyboard.press('w');
      isWalkingFwd = true;
      Serial.println("FWD (W)");
    } else if (!wantFwd && isWalkingFwd) {
      if (bleKeyboard.isConnected()) bleKeyboard.release('w');
      isWalkingFwd = false;
    }
    
    if (wantBack && !isWalkingBack) {
      if (bleKeyboard.isConnected()) bleKeyboard.press('s');
      isWalkingBack = true;
      Serial.println("BACK (S)");
    } else if (!wantBack && isWalkingBack) {
      if (bleKeyboard.isConnected()) bleKeyboard.release('s');
      isWalkingBack = false;
    }
  }
  
  // === Pinches ===
  bool index = (digitalRead(PINCH_INDEX) == LOW);
  bool middle = (digitalRead(PINCH_MIDDLE) == LOW);
  bool ring = (digitalRead(PINCH_RING) == LOW);
  bool pinky = (digitalRead(PINCH_PINKY) == LOW);
  
  // Index = JUMP (held)
  if (index && !isJumping) {
    if (bleKeyboard.isConnected()) bleKeyboard.press(' ');
    isJumping = true;
    Serial.println("JUMP press");
  } else if (!index && isJumping) {
    if (bleKeyboard.isConnected()) bleKeyboard.release(' ');
    isJumping = false;
  }
  
  // Middle = SPRINT toggle (tap to flip on/off)
  if (middle && !wasMiddle) {
    isSprinting = !isSprinting;
    if (bleKeyboard.isConnected()) {
      if (isSprinting) {
        bleKeyboard.press(KEY_LEFT_SHIFT);
        Serial.println("Sprint ON");
      } else {
        bleKeyboard.release(KEY_LEFT_SHIFT);
        Serial.println("Sprint OFF");
      }
    }
  }
  wasMiddle = middle;
  
  // Ring = INVENTORY (tap)
  if (ring && !wasRing) {
    if (bleKeyboard.isConnected()) bleKeyboard.write('e');
    Serial.println("INVENTORY (E)");
  }
  wasRing = ring;
  
  // Pinky = SNEAK (tap)
  if (pinky && !wasPinky) {
    if (bleKeyboard.isConnected()) bleKeyboard.write('c');
    Serial.println("SNEAK (C)");
  }
  wasPinky = pinky;
  
  // === Fist -> ESC ===
  bool isFist = (digitalRead(HALL_INDEX) == LOW) && 
                (digitalRead(HALL_RING) == LOW);
  
  if (isFist && !wasFist) {
    if (millis() - lastFistTrigger > FIST_DEBOUNCE_MS) {
      if (bleKeyboard.isConnected()) bleKeyboard.write(KEY_ESC);
      Serial.println("FIST -> ESC");
      lastFistTrigger = millis();
    }
    wasFist = true;
  } else if (!isFist && wasFist) {
    wasFist = false;
  }
  
  delay(20);
}
