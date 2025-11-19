# Welcome

Welcome to the ISDN2602 Final Project Website


## Changelog


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