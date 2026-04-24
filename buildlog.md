# Build Log

## [04/24/2026] - ESP32 core v3 breaks BleKeyboard
- Soldered Hall effect sensor wires
- Tested all sensors after the heat from soldering
- Mounted sensors onto the glove with

## [04/23/2026] - v1 milestone: pinch + BLE + haptic working
- Got all 4 pinch pads typing letters into Notepad over BLE
- Added 40ms haptic confirmation on pinch
- Known issues: some pinches flakier than others, needs more robust contact
- Next: mount hall sensors once new magnets arrive

## 04/22/2026] - Hall sensor range issue
- Single 4x2mm N42 doesn't trigger sensor through glove fabric
- All 4 pinch pads properly detected
- 2-stack of same magnets: triggers reliably
- Decision: bought new 10x3 mm magnets

## [04/21/2026] - ESP32 core v3 breaks BleKeyboard
- Core 3.3.8 changed std::string → String in BLE API
- T-vK's BleKeyboard library fails to compile
- Tried blackketter fork: still failed  
- Fix: downgraded ESP32 core to v2.0.17, compiles clean

## [04/18/2026] - ESP32 core v3 breaks BleKeyboard
- Parts all ordered
- Parts list is in hardware.md

## [04/17/2026] - ESP32 core v3 breaks BleKeyboard
- Planned out the usage of copper pads for an action
- Magnets + hall sensor for action
- IMU for the wrist rotation action
- Protoboard for final product
