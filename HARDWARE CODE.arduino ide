#include <BluetoothSerial.h>
#include <HardwareSerial.h>
#include <DFRobotDFPlayerMini.h>

// ==================== PIN CONFIGURATION ====================
#define FLEX_SENSOR_1  34  // Thumb (Blue & Yellow)
#define FLEX_SENSOR_2  35  // Index (Red & White)
#define FLEX_SENSOR_3  32  // Middle (White & Grey)
#define FLEX_SENSOR_4  33  // Ring (Red & Orange)
#define SPEAKER_PIN    25  // DAC1 (not used with DFPlayer)

// ==================== INDIVIDUAL SENSOR THRESHOLDS ====================
const int THRESHOLD_1 = 3900;  // Thumb: < 3500 = BENT
const int THRESHOLD_2 = 3500;  // Index: < 3000 = BENT
const int THRESHOLD_3 = 3900;  // Middle: < 3500 = BENT
const int THRESHOLD_4 = 3500;  // Ring: < 3000 = BENT

// ==================== BLUETOOTH CONFIGURATION ====================
BluetoothSerial SerialBT;
const char* BT_DEVICE_NAME = "ESP32_GestureControl";

// ==================== DFPLAYER CONFIGURATION ====================
HardwareSerial MP3Serial(2);      // UART2 on ESP32
DFRobotDFPlayerMini myDFPlayer;

const int DF_RX = 16;  // ESP32 RX2 (connect to DFPlayer TX)
const int DF_TX = 17;  // ESP32 TX2 (connect to DFPlayer RX via 1k resistor)

// ==================== DEBOUNCE ====================
unsigned long lastGestureTime = 0;
const int DEBOUNCE_TIME = 2000;

// ==================== SETUP ====================
void setup() {
  Serial.begin(115200);
  delay(1000);
  
  SerialBT.begin(BT_DEVICE_NAME);
  
  analogSetAttenuation(ADC_11db);
  analogReadResolution(12);
  
  Serial.println("\n╔════════════════════════════════════════╗");
  Serial.println("║  Sign Language to Speech Converter     ║");
  Serial.println("║   DFPlayer Voice Output Version        ║");
  Serial.println("╚════════════════════════════════════════╝\n");
  
  Serial.println("Individual Sensor Thresholds:");
  Serial.println("• Sensor 1 (THUMB):  < 3899 = BENT → I need water");
  Serial.println("• Sensor 2 (INDEX):  > 3500 = BENT → I need washroom");
  Serial.println("• Sensor 3 (MIDDLE): < 3899 = BENT → I need help");
  Serial.println("• Sensor 4 (RING):   < 3499 = BENT → I have pain");
  Serial.println("• THUMB + INDEX BENT → Emergency help needed");
  Serial.println("• MIDDLE + RING BENT → Thank you\n");
  
  // ----- DFPlayer init -----
  MP3Serial.begin(9600, SERIAL_8N1, DF_RX, DF_TX);
  Serial.println("Initializing DFPlayer Mini...");
  delay(1000);
  
  if (!myDFPlayer.begin(MP3Serial)) {
    Serial.println("DFPlayer init failed! Check wiring and SD card.");
    while (true) {
      delay(1000);
    }
  }
  Serial.println("✓ DFPlayer Mini online.");
  myDFPlayer.volume(25);  // 0–30, adjust volume as needed
  
  Serial.println("\nAudio file mapping (on SD card root):");
  Serial.println("0001.mp3 → I need water");
  Serial.println("0002.mp3 → I need washroom");
  Serial.println("0003.mp3 → I need help");
  Serial.println("0004.mp3 → I have pain");
  Serial.println("0005.mp3 → Emergency help needed");
  Serial.println("0006.mp3 → Thank you\n");
}

// ==================== MAIN LOOP ====================
void loop() {
  // Read sensors with averaging
  int sensor1 = 0, sensor2 = 0, sensor3 = 0, sensor4 = 0;
  
  for (int i = 0; i < 5; i++) {
    sensor1 += analogRead(FLEX_SENSOR_1);
    sensor2 += analogRead(FLEX_SENSOR_2);
    sensor3 += analogRead(FLEX_SENSOR_3);
    sensor4 += analogRead(FLEX_SENSOR_4);
    delayMicroseconds(500);
  }
  
  sensor1 /= 5;
  sensor2 /= 5;
  sensor3 /= 5;
  sensor4 /= 5;
  
  delay(20);
  
  // Determine BENT (1) or STRAIGHT (0) based on INDIVIDUAL THRESHOLDS
  int thumb = (sensor1 < THRESHOLD_1) ? 1 : 0;    // BENT if < 3500
  int index = (sensor2 > THRESHOLD_2) ? 1 : 0;    // BENT if < 3000
  int middle = (sensor3 < THRESHOLD_3) ? 1 : 0;   // BENT if < 3500
  int ring = (sensor4 < THRESHOLD_4) ? 1 : 0;     // BENT if < 3000
  
  // Print raw and normalized values every 500ms
  static unsigned long lastPrint = 0;
  if (millis() - lastPrint > 500) {
    Serial.print("[");
    Serial.print(millis());
    Serial.print("ms] Raw: [");
    Serial.print(sensor1);
    Serial.print(", ");
    Serial.print(sensor2);
    Serial.print(", ");
    Serial.print(sensor3);
    Serial.print(", ");
    Serial.print(sensor4);
    Serial.print("] → [T:");
    Serial.print(thumb);
    Serial.print(" I:");
    Serial.print(index);
    Serial.print(" M:");
    Serial.print(middle);
    Serial.print(" R:");
    Serial.print(ring);
    Serial.println("]");
    lastPrint = millis();
  }
  
  // ==================== GESTURE RECOGNITION WITH IF-ELSE ====================
  
  // GESTURE 1: ONLY THUMB BENT → track 1
  if (thumb == 1 && index == 0 && middle == 0 && ring == 0) {
    if (millis() - lastGestureTime > DEBOUNCE_TIME) {
      sendGesture("WATER", "I need water", 1);
      lastGestureTime = millis();
      delay(2000);
    }
  }
  
  // GESTURE 2: ONLY INDEX BENT → track 2
  else if (thumb == 0 && index == 1 && middle == 0 && ring == 0) {
    if (millis() - lastGestureTime > DEBOUNCE_TIME) {
      sendGesture("WASHROOM", "I need washroom", 2);
      lastGestureTime = millis();
      delay(2000);
    }
  }
  
  // GESTURE 3: ONLY MIDDLE BENT → track 3
  else if (thumb == 0 && index == 0 && middle == 1 && ring == 0) {
    if (millis() - lastGestureTime > DEBOUNCE_TIME) {
      sendGesture("HELP", "I need help", 3);
      lastGestureTime = millis();
      delay(2000);
    }
  }
  
  // GESTURE 4: ONLY RING BENT → track 4
  else if (thumb == 0 && index == 0 && middle == 0 && ring == 1) {
    if (millis() - lastGestureTime > DEBOUNCE_TIME) {
      sendGesture("PAIN", "I have pain", 4);
      lastGestureTime = millis();
      delay(2000);
    }
  }
  
  // GESTURE 5: THUMB + INDEX BENT → track 5
  else if (thumb == 1 && index == 1 && middle == 0 && ring == 0) {
    if (millis() - lastGestureTime > DEBOUNCE_TIME) {
      sendGesture("EMERGENCY", "Emergency help needed", 5);
      lastGestureTime = millis();
      delay(2000);
    }
  }
  
  // GESTURE 6: MIDDLE + RING BENT → track 6
  else if (thumb == 0 && index == 0 && middle == 1 && ring == 1) {
    if (millis() - lastGestureTime > DEBOUNCE_TIME) {
      sendGesture("THANK_YOU", "Thank you", 6);
      lastGestureTime = millis();
      delay(2000);
    }
  }
}

// ==================== SEND GESTURE ====================
void sendGesture(String gestureName, String message, int trackNumber) {
  // Print to Serial
  Serial.println("\n╔════════════════════════════════════════╗");
  Serial.println("║     ✓ GESTURE RECOGNIZED!              ║");
  Serial.println("╚════════════════════════════════════════╝");
  Serial.print("Gesture: ");
  Serial.println(gestureName);
  Serial.print("Message: ");
  Serial.println(message);
  Serial.print("Playing Track: ");
  Serial.println(trackNumber);
  Serial.println("");
  
  // Send via Bluetooth
  String btMessage = "GESTURE:" + message;
  SerialBT.println(btMessage);
  Serial.println("[BT] Sent: " + btMessage);
  
  // Play voice from DFPlayer
  myDFPlayer.play(trackNumber);   // plays 000X.mp3 from root
  Serial.println("Playing audio...");
}

// ==================== END OF CODE ====================
