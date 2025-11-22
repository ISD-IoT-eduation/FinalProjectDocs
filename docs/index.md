# Welcome

Welcome to the ISDN2602 Final Project Website


## Changelog

#### [2025-11-22] Update [Control Panel Section](/controlPanel/) 

To include Traffic Light System Control and Timer Control note.


#### [2025-11-21] RFID Task 

Change 1 - In files `CodeWithFirebase/CodeWithFirebase.ino` and `CodeWithoutFireBase/CodeWithoutFireBase.ino` at lines 363 and 372, modify the RFID tag reader task to store the current tag UID in a global variable:
```diff
/*-------------RFID Tag Reader Task-------------*/
TaskHandle_t RFIDTagReaderTaskHandle = NULL;
StaticTask_t xRFIDTagReaderTCB;
+ String currenttagUID = "";
void RFIDTagReaderTask(void* pvPara) {
  while (true) {
    // If no new card or read failed, wait and continue
    if (!RFIDReader::mfrc522.PICC_IsNewCardPresent() || !RFIDReader::mfrc522.PICC_ReadCardSerial()) {
      vTaskDelay(pdMS_TO_TICKS(50));
      continue;
    }

-   String currenttagUID = RFIDReader::GetTagUID();
+   currenttagUID = RFIDReader::GetTagUID();
```

Change 2 - In files `CodeWithFirebase/RFIDReader.cpp` and `CodeWithoutFirebase/RFIDReader.cpp` at line 9, set the RFID antenna gain to maximum for better reading range:

```diff
  // Initialize RFID
  Wire.begin(Pinout::RFID_SDA, Pinout::RFID_SCL);
  mfrc522.PCD_Init();
+ mfrc522.PCD_SetAntennaGain(mfrc522.RxGain_max);
  Serial.println("RFID Initialized");
```


Change 3 - In files `CodeWithFirebase/CodeWithFirebase.ino` and `CodeWithoutFireBase/CodeWithoutFireBase.ino`, please move the `RFIDTagReaderTask` definition above the `MovementTask`, so that the `currenttagUID` variable can be accessed in the `MovementTask`:


That is to move the entire `RFIDTagReaderTask` code block up to be before the `MovementTask` code block, like so:

```diff
+ /*-------------RFID Tag Reader Task-------------*/
+ TaskHandle_t RFIDTagReaderTaskHandle = NULL;
+ StaticTask_t xRFIDTagReaderTCB;
+ String currenttagUID = "";
+ void RFIDTagReaderTask(void* pvPara) {
+   // ...
+ }

/*-------------LED Blinking Task-------------*/
TaskHandle_t LEDBlinkingTaskHandle = NULL;
StaticTask_t xLEDBlinkingTCB;
void LEDBlinkingTask(void* pvPara) {
 // ...
};

TaskHandle_t MovementTaskHandle = NULL;
StaticTask_t xMovementTCB;
void MovementTask(void* pvPara) {
    // ...
}

- /*-------------RFID Tag Reader Task-------------*/
- TaskHandle_t RFIDTagReaderTaskHandle = NULL;
- StaticTask_t xRFIDTagReaderTCB;
- String currenttagUID = "";
- void RFIDTagReaderTask(void* pvPara) {
-   // ...
- }
```





#### [2025-11-19 8:10 PM] Servo and RFID Fixes

In files `CodeWithFirebase/CodeWithFirebase.ino` and `CodeWithoutFireBase/CodeWithoutFireBase.ino` at line 377 to increase the delay time for RFID tag reading:

```diff
void RFIDTagReaderTask(void* pvPara) {
    while (true) {
        // ...
-       vTaskDelay(pdMS_TO_TICKS(10));
+       vTaskDelay(500/portTICK_PERIOD_MS);
    }
}
```

In files `CodeWithFirebase/MotorControl.hpp` and `CodeWithoutFireBase/MotorControl.hpp` at lines 28 and 30 to modify the PWM resolution and channel for the servo motor:

```diff
struct ServoMotor
{
    const uint8_t PWMFrequency = 50; // PWM must be in 50Hz
-   const uint8_t PWMResolution = 12;
+   const uint8_t PWMResolution = 10;
    uint16_t PWMDuty = 0;
-   const uint8_t PWMChannel = 8; // Ideally select between 5-10
+   const uint8_t PWMChannel = 6; // Ideally select between 5-10
    float TargetAngle = 0.0f;
};
```

#### [2025-11-18 7:35 PM] Ultrasonic Sensor Pin Change

In files `CodeWithFirebase/Pinout.hpp` and `CodeWithoutFireBase/Pinout.hpp` at lines 20 and 21, the ultrasonic sensor pins have been changed as follows:
    
```diff
// Ultrasonic Sensor 
- const uint8_t UltrasonicTrigPin = 6;
+ const uint8_t UltrasonicTrigPin = 39;
- const uint8_t UltrasonicEchoPin = 4;
+ const uint8_t UltrasonicEchoPin = 38;
```

#### [2025-11-18] Release of Final Project


## Notes

If you have already cloned the repository, you will receive a pull request with the updated changes, please refer to the image below to accept and merge the changes. Or you may choose to manually update the code that have been changed.


![accept pr](assets/accept_pr.jpeg)