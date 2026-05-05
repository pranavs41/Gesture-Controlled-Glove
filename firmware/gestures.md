# Gestures

The glove translates physical hand gestures into keyboard input over Bluetooth Low Energy (BLE). It pairs with a PC as a generic BLE keyboard named **`Glove-L`** and works with any application that accepts keyboard input — most testing has been done in Minecraft, but the gestures map cleanly to typical FPS controls and other WASD-based games.

## Gesture map

| Gesture | Action | Key sent | Behavior |
| --- | --- | --- | --- |
| Tilt hand left | Strafe left | `A` | Held while tilted |
| Tilt hand right | Strafe right | `D` | Held while tilted |
| Tilt hand forward (down) | Walk forward | `W` | Held while tilted |
| Tilt hand backward (up) | Walk backward | `S` | Held while tilted |
| Thumb + index pinch | Jump | `Space` | Held while pinched |
| Thumb + middle tap | Sprint toggle | `Left Shift` | Tap to toggle on/off |
| Thumb + ring tap | Inventory | `E` | Single tap |
| Thumb + pinky tap | Sneak toggle | `C` | Single tap (game-side toggle) |
| Fist (full hand clench) | Pause / menu | `Esc` | Single tap, debounced 1s |

## How each gesture is detected

### Pinches (thumb to fingertip contact)

Each fingertip has a copper foil pad wired to a GPIO pin configured with `INPUT_PULLUP`. The thumb has its own copper pad wired directly to ground. When the thumb touches a fingertip pad, that GPIO is pulled `LOW`, and the firmware registers a pinch on that finger.

| Finger | GPIO | Default mapping |
| --- | --- | --- |
| Index | 13 | Jump (held) |
| Middle | 14 | Sprint (toggle) |
| Ring | 27 | Inventory (tap) |
| Pinky | 26 | Sneak (tap) |
| Thumb | GND | (common ground) |

### Fist (curl detection via hall sensors)

Two AH3144E hall effect sensors are mounted on the proximal phalanx of the index and ring fingers, with neodymium magnets on the corresponding fingertips. When all four fingers curl into a fist, both magnets approach their respective sensors and the sensors' outputs go `LOW`. Both sensors must trigger simultaneously to register a fist, which makes accidental triggers rare. A 1-second debounce prevents rapid retriggering.

The middle and pinky finger positions also have hall sensors physically mounted, but their magnets are reversed relative to the working sensors, so they are intentionally unused in the firmware. Two working sensors out of four is enough to reliably detect a clenched fist.

### Tilt detection (IMU orientation)

A BNO085 9-DOF IMU is mounted on the back of the hand. The firmware reads the rotation vector (a quaternion) over I²C and converts it into roll and pitch angles. Because the IMU is not aligned perfectly with the wrist's natural axes on this build, the firmware computes virtual axes by combining roll and pitch:

- **Strafe signal** = `(pitch − pitch_neutral) + (roll − roll_neutral)`
- **Walk signal** = `(pitch − pitch_neutral) − (roll − roll_neutral)`

When either signal exceeds its threshold (`±40°` by default), the corresponding direction key is held until the hand returns to neutral.

Calibration values are tuned for the current build and live at the top of the firmware:

```cpp
const float ROLL_NEUTRAL = 5.6f;
const float PITCH_NEUTRAL = -21.2f;
const float STRAFE_THRESHOLD = 40.0f;
const float WALK_THRESHOLD = 40.0f;
```

If the IMU is remounted or the glove fits a different hand, these values will need to be retuned. To recalibrate, upload the diagnostic sketch in `firmware/imu_diagnostic/`, log roll and pitch at rest and at each tilt extreme, and update the constants.

## Behavior details

### Held vs tap vs toggle

- **Held** gestures press a key on detection and release it when the gesture ends. Used for movement (WASD) and jumping. Letting go of the gesture immediately stops the action.
- **Tap** gestures send a single press-and-release. Used for one-shot actions like opening inventory.
- **Toggle** gestures flip a held state on/off with each tap. Sprint uses this because the middle copper pad has unreliable release detection on this build, and because sprint-while-jumping is a common Minecraft action that's awkward to hold.

### Combinations

All gesture inputs run independently each loop iteration, so combinations work naturally:

- Tilt forward + thumb-index pinch → walk forward + jump (e.g., parkour)
- Tilt left + sprint toggle on → strafe left while sprinting
- Multiple tilts together (e.g., forward + right) → diagonal movement (`W` and `D` both held)

### Debounce and false positives

- **Fist:** 1000 ms debounce between triggers, since accidental fist detection causes a pause that's disruptive mid-game.
- **Pinches:** no debounce; they fire immediately on contact transition. The physical thumb-to-finger contact provides natural debouncing.

## Customizing keybindings

All keys are defined as constants near the top of the main firmware sketch. To rebind a gesture, edit the corresponding `bleKeyboard.press(...)` or `bleKeyboard.write(...)` call. For example, to change the inventory key from `E` to `Tab`:

```cpp
// Before
if (bleKeyboard.isConnected()) bleKeyboard.write('e');

// After
if (bleKeyboard.isConnected()) bleKeyboard.write(KEY_TAB);
```

A future version will move these into a profile system editable via a web interface served by the ESP32, so the glove can switch between game-specific keybindings without re-flashing.

## Game compatibility

The gesture map was designed around Minecraft's default controls, but works in any game that accepts standard keyboard input. For other games with different default keybinds, either rebind in firmware or remap the game's controls to match the glove's output.

A few notes for specific games:

- **Minecraft:** default sprint is `Ctrl`, not `Shift`. Either change the firmware to send `KEY_LEFT_CTRL` for the middle pinch, or change Minecraft's sprint binding to `Shift` in Options → Controls.
- **Roblox / generic FPS:** WASD + Space + Shift + E typically map directly with no changes.
- **Strategy / RTS:** the inventory key `E` may conflict with unit commands; rebind as needed.

## Known limitations

- The middle pinch pad sometimes stays in contact after release, which is why sprint is a toggle rather than a held key. A v2 build using conductive fabric pads would address this.
- IMU thresholds are hand-specific; another user's natural neutral position will require recalibration.
- The glove is left-hand only. Camera/aim control still requires a mouse in the right hand. A right-hand companion glove with mouse-replacement gestures is planned for v2.
- BLE has occasional reconnect delays after the PC sleeps. Re-pairing or toggling Bluetooth on the PC fixes it.
