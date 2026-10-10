# Hand Gesture Recognition Glove - Complete Setup Guide

## Project Overview
This project uses Arduino Uno to detect hand gestures through flex sensors, transmits them via Bluetooth (HC-05), and an Android app receives and speaks the gesture name using Text-to-Speech (TTS).

---

## Hardware Components

### Arduino Setup
- **Arduino Uno**
- **4 Flex Sensors** (for Index, Middle, Ring, Little fingers)
- **HC-05 Bluetooth Module**
- **16x2 I2C LCD Display**
- **5 LEDs** (with 220Ω resistors)
- **Resistors** (for flex sensor voltage dividers)
- **Breadboard & Jumper Wires**
- **Power Supply** (5V USB or battery)

---

## Wiring Diagram

### Flex Sensors (with 10kΩ voltage dividers)
```
FLEX_INDEX (A0)  -> Flex Sensor + 10kΩ pull-down resistor
FLEX_MIDDLE (A1) -> Flex Sensor + 10kΩ pull-down resistor
FLEX_RING (A2)   -> Flex Sensor + 10kΩ pull-down resistor
FLEX_LITTLE (A3) -> Flex Sensor + 10kΩ pull-down resistor
```

### I2C LCD (16x2)
```
SDA (A4)  -> LCD SDA
SCL (A5)  -> LCD SCL
VCC       -> 5V
GND       -> GND
```

### HC-05 Bluetooth Module
```
TX   -> Arduino RX (D0) via voltage divider
RX   -> Arduino TX (D1) 
VCC  -> 5V
GND  -> GND
```
**Note:** For hardware serial, or use SoftwareSerial on D10 (RX) and D11 (TX) as in the code.

### LEDs
```
LED_OPEN (D2)     -> LED + 220Ω resistor to GND (Open Hand indicator)
LED_FIST (D3)     -> LED + 220Ω resistor to GND (Fist indicator)
LED_ONE (D4)      -> LED + 220Ω resistor to GND (One Finger indicator)
LED_TWO (D5)      -> LED + 220Ω resistor to GND (Two Fingers indicator)
LED_THREE (D6)    -> LED + 220Ω resistor to GND (Three Fingers indicator)
```

---

## Arduino Installation & Upload

1. **Install Arduino IDE** from [arduino.cc](https://www.arduino.cc/en/software)

2. **Install Required Libraries:**
   - Open Arduino IDE
   - Go to **Sketch** → **Include Library** → **Manage Libraries**
   - Search and install:
     - `LiquidCrystal_I2C` by Frank de Brabander
     - SoftwareSerial (built-in)

3. **Download the Code:**
   - Clone this repository
   - Open `gesture_recognition_glove.ino` in Arduino IDE

4. **Configure Settings:**
   - Update `HC05_MAC` if using I2C LCD address other than `0x27`
   - Calibrate flex sensor thresholds (see Calibration section below)

5. **Upload to Arduino:**
   - Select **Tools** → **Board** → **Arduino Uno**
   - Select correct **COM Port**
   - Click **Upload**

---

## Flex Sensor Calibration

1. **Open Serial Monitor** (Tools → Serial Monitor, 9600 baud)

2. **Record values for each gesture:**
   - Open hand (all fingers straight)
   - Fist (all fingers bent)
   - One finger
   - Two fingers
   - Three fingers

3. **Update these values in the code:**
   ```cpp
   const int BEND_THRESHOLD = 150;          // Adjust based on your sensor
   const int STRAIGHT_THRESHOLD = 500;      // Adjust based on your sensor
   const bool BENDING_INCREASES = true;     // Set to false if values decrease when bending
   ```

4. **Test and refine** until all gestures are recognized correctly.

---

## Android App Setup

### Prerequisites
- Android Studio (download from [developer.android.com](https://developer.android.com/studio))
- Android device with API 21+ and Bluetooth support
- Bluetooth enabled on your Android device

### Steps

1. **Create New Android Project in Android Studio:**
   - File → New → New Project
   - Choose "Empty Activity"
   - Minimum SDK: API 21 (Android 5.0)

2. **Copy Code Files:**
   - Copy `MainActivity.java` to `app/src/main/java/com/example/handgesturerecognition/`
   - Copy `activity_main.xml` to `app/src/main/res/layout/`
   - Copy `AndroidManifest.xml` to `app/src/main/`
   - Copy `build.gradle` to `app/`

3. **Update HC-05 MAC Address:**
   - In `MainActivity.java`, replace:
   ```java
   private static final String HC05_MAC = "98:D3:31:F5:D6:02";
   ```
   - With your actual HC-05 MAC address (find in Android Bluetooth Settings)

4. **Sync and Build:**
   - Android Studio → File → Sync Now
   - Build → Make Project

5. **Run on Device:**
   - Connect Android device via USB
   - Run → Run 'app'

---

## How It Works

### Arduino Flow
1. Reads flex sensor values from A0-A3
2. Detects hand gesture (Open Hand, Fist, 1-3 Fingers)
3. Waits 500ms to confirm stable gesture
4. Sends `GESTURE:OPEN_HAND` via Bluetooth to HC-05
5. Displays gesture on I2C LCD
6. Lights up corresponding LED

### Android App Flow
1. Connects to HC-05 Bluetooth module
2. Listens for incoming messages
3. Parses gesture format: `GESTURE:OPEN_HAND`
4. Displays gesture on screen
5. Uses Text-to-Speech to speak: "Open hand"

---

## Gesture Recognition Mapping

| Gesture | Arduino LED | Android Display | TTS Output |
|---------|-------------|-----------------|-----------|
| All fingers straight | LED_OPEN (D2) | OPEN_HAND | "Open hand" |
| All fingers bent | LED_FIST (D3) | FIST | "Fist" |
| 1 finger straight | LED_ONE (D4) | ONE_FINGER | "One finger" |
| 2 fingers straight | LED_TWO (D5) | TWO_FINGERS | "Two fingers" |
| 3 fingers straight | LED_THREE (D6) | THREE_FINGERS | "Three fingers" |

---

## Troubleshooting

### Arduino Issues

**Problem:** Flex sensors not reading correctly
- **Solution:** Check voltage divider resistor values and sensor connections
- Verify readings with Serial Monitor

**Problem:** LCD not displaying
- **Solution:** Check I2C address with `i2c_scanner` sketch
- Update address in code: `LiquidCrystal_I2C lcd(0x27, 16, 2);`

**Problem:** Bluetooth not connecting
- **Solution:** Check HC-05 baud rate (usually 9600)
- Verify TX/RX connections
- Use voltage divider on RX pin if needed (5V → 3.3V)

### Android Issues

**Problem:** App can't find Bluetooth device
- **Solution:** Pair HC-05 in Android settings first
- Update MAC address in `MainActivity.java`

**Problem:** No sound from app
- **Solution:** Check phone volume and Text-to-Speech engine
- Grant microphone permissions

**Problem:** Permission errors on Android 12+
- **Solution:** Ensure `BLUETOOTH_CONNECT` and `BLUETOOTH_SCAN` permissions are granted
- Check `AndroidManifest.xml` has all required permissions

---

## Performance Tips

1. **Gesture Detection Stability:**
   - Increase `HOLD_TIME` for more stable detection (currently 500ms)
   - Fine-tune flex sensor thresholds for your specific sensors

2. **Bluetooth Range:**
   - HC-05 has ~10 meter range in open space
   - Remove obstacles for better connection

3. **Battery Life:**
   - Use power-efficient Arduino settings
   - Consider battery monitoring for mobile use

---

## Future Enhancements

- Add thumb sensor for 5-finger detection
- Implement machine learning for better gesture recognition
- Add gesture recording and playback
- Wireless power charging for glove
- Mobile app UI improvements with gesture history
- Multi-language Text-to-Speech support

---

## Files Included

```
gesture-recognition-glove/
├── gesture_recognition_glove.ino          (Arduino sketch)
├── Android/
│   ├── MainActivity.java                   (Android app main activity)
│   ├── activity_main.xml                   (Android layout)
│   ├── AndroidManifest.xml                 (Android permissions & config)
│   └── build.gradle                        (Android build config)
├── SETUP_GUIDE.md                          (This file)
└── README.md                               (Project overview)
```

---

## License

This project is open-source and available for educational and personal use.

---

## Support

For issues or questions:
1. Check the Troubleshooting section above
2. Review Arduino Serial Monitor output
3. Check Android Logcat output
4. Ensure all connections are secure
5. Verify calibration values for your specific sensors

---

**Happy Gesture Recognizing! 🎉**
