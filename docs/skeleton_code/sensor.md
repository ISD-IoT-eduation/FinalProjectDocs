# Sensors

## Infrared Sensor (IR Sensor) 

### IR Configuration
```cpp
 // IR Sensor Configuration
    /*
           L   M   R  
            \  |  /              
             \ | /       ^
              \|/        |
              Car      Front
    
    */
```

### `IRSensor.hpp`

Define the namespace `IRSensors` to encapsulate all IR sensor related functions and variables, thus to use functions and variables in the main code file `.ino`, need to add prefix with `IRSensors::`.

```cpp
namespace IRSensors {
    // ...
}
```

Define the `RobotState` enum (list of `uint8_t` numbers) to represent the various states of the robot based on the readings from the three IR sensors (Left, Middle, Right).

```cpp
// Robot States
    enum RobotState : uint8_t {
        /* The logic of the IR sensor and its output
        |    Tile Colour   |  digitalRead() |
        |      Black       |    1 (HIGH)    |
        |      White       |     0 (LOW)    |
         */
        /*  Logic Reference Table
        |    uint8_t     |     boolean ( digitalRead() )  |
        |   RobotState    |   IR_L   |   IR_M   |   IR_R   |
        |        0        |     0    |     1    |     0    |
        |        1        |     1    |     1    |     0    |
        |        2        |     1    |     1    |     1    |
        |        3        |     1    |     0    |     1    |
        |        4        |     0    |     1    |     1    |
        |        5        |     0    |     0    |     1    |
        |        6        |     1    |     0    |     0    |
        |        7        |     0    |     0    |     0    |
        
        |*/
        Middle_ON_Track,        // 0 
        Left_Middle_ON_Track,   // 1
        ALL_ON_Track,           // 2
        Left_Right_ON_Track,    // 3
        Middle_Right_ON_Track,   // 4
        Right_ON_Track,         // 5
        Left_ON_Track,          // 6
        All_OFF_Track           // 7 
    };
```

Define the `IRSensorData` structure to hold the readings from the three IR sensors and the current state of the robot.

```cpp
 // IR Sensor Structure
struct IRSensorData {
    // Sensor states
    bool Read_IR_L;
    bool Read_IR_M;
    bool Read_IR_R;
    uint8_t state;
};
```

Declare a global instance of `IRSensorData` that can be accessed across different files.

```cpp
// Global IR sensor instance
extern IRSensorData IRData;
```

Declare the initialization and sensor reading functions for the IR sensors. The actual implementations will be in the `IRSensors.cpp` file.

```cpp
// Function declarations
void Init();
uint8_t ReadSensorState(IRSensorData& IRData);
```

### `IRSensors.cpp`

Initialize the global IR sensor instance with default values of `false` for each sensor reading and `Middle_ON_Track` for the robot state.
```cpp
// Global IR sensor instance
IRSensorData IRData = {
    false,          //bool Read_IR_L;
    false,          //bool Read_IR_M;
    false,          //bool Read_IR_R;
    Middle_ON_Track //RobotState state;
    };
```

Configure the GPIO setup for the IR Sensor pins
```cpp
void Init() {
    // Initialize IR sensor pins
    pinMode(Pinout::IRLeft, INPUT);
    pinMode(Pinout::IRMiddle, INPUT);
    pinMode(Pinout::IRRight, INPUT);
};
```

Function to read the current state of the IR sensors and match to the robot's state based on the sensor readings.
```cpp
uint8_t ReadSensorState(IRSensorData& IRData){
    /*Scan all the IR sensors to get the current status of the car*/
    IRData.Read_IR_L = digitalRead(Pinout::IRLeft);
    IRData.Read_IR_M = digitalRead(Pinout::IRMiddle);
    IRData.Read_IR_R = digitalRead(Pinout::IRRight);

    /*According to the current status of the IR Sensors, determine the RobotState*/
    if(IRData.Read_IR_L == 0 && IRData.Read_IR_M == 1 && IRData.Read_IR_R == 0 ){
        return Middle_ON_Track;
    } else if (...){
        // ...
    }
    // Continue for other states...

    return 10; //Error Code if none of condition fits.
}
```

## RFID Reader
The code of the RFID Reader (MFRC522) is based on an open-source repo in github.

Repo URL: [miguelbalboa/rfid](https://github.com/miguelbalboa/rfid)

The datasheet of the MFRC522 can be found here: ([MFRC522 Datasheet](https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/5531/4411_CN0090%20other%20related%20document%20%281%29.pdf))

The driver in skeleton code is modified for running I2C protocol, which the repo above is for SPI protocol. 

The driver consist of the following files:
* `MFRC522_I2C.hpp`
* `MFRC522_I2C.cpp`

This manual will ONLY guide you how to call the API in your `.ino` file. 


## Inertial Measurement Unit (IMU)
The code of the IMU (ICM42688-P) is based on an open-source repo in github. 

Repo URL: [finani/ICM42688](https://github.com/finani/ICM42688) 

For detail please refer to the github page. 

The datasheet of the ICM42688-P can be found here: [ICM42688-P Datasheet](https://product.tdk.com/system/files/dam/doc/product/sensor/mortion-inertial/imu/data_sheet/ds-000347-icm-42688-p-v1.6.pdf) 
The driver consist of the following files: 
* `IMU.h`
* `IMU.cpp`
* `register.h`

This manual will ONLY guide you how to call the API in your `.ino` file. 


## Ultrasonic Sensor
In `UltrasonicSensor.hpp`, 
```cpp 
namespace UltrasonicSensor{
  void Init();
  float GetDistance();
}
```

In `UltrasonicSensor.cpp` 
Setting the constant value of speed of sound: 
```cpp
#define SOUND_SPEED 340
```

## `void UltrasonicSensor::Init()`
Initialization of the ultrasonic sensor pin, echo and trig pin. 
```cpp
void UltrasonicSensor::Init(){ 
  pinMode(Pinout::UltrasonicTrigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(Pinout::UltrasonicEchoPin, INPUT); // Sets the echoPin as an Input
  Serial.println("Ultrasonic Sensor is set");
}
```


## `float UltrasonicSensor::GetDistane()`
@brief Ouput the distance of the ultrasonic sensor measured. (in meter)

@return Distance (in float)

```cpp
float UltrasonicSensor::GetDistance(){
  long duration = 0;  //initalize the temp para. 
  float distance = 0; 
  // Clears the trigPin
  digitalWrite(Pinout::UltrasonicTrigPin, LOW);
  delayMicroseconds(2);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(Pinout::UltrasonicTrigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(Pinout::UltrasonicTrigPin, LOW);
  
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(Pinout::UltrasonicEchoPin, HIGH);
  
  // Calculate the distance (in m)
  distance = (duration * SOUND_SPEED/100)/2;
  return distance; 
  //For Debug Use
  // Prints the distance in the Serial Monitor
  // Serial.print("Distance (cm): ");
  // Serial.println(distance/100);
}
```

## Buzzer 

The buzzer used on the board is Passive Buzzer which the buzzer is controlled by the PWM signal from the MCU. 

The driver consist of the following files: 
* `Buzzer.hpp`
* `Buzzer.cpp`
* `pitches.h`

In `Buzzer.hpp`,
A mdelody with the respective duration of each note is created for demo purpose. 
```cpp
extern int DemoMelody[];
extern int DemoDurations[];
```

In `Buzzer.cpp`, 
```cpp
//Here is a reference for the melody and duration format, also refer to the pitches.h
int DemoMelody[] = {
  // Phrase 1: Intro Fanfare (16th notes)
  NOTE_FS4, NOTE_G4, NOTE_A4, NOTE_B4, NOTE_C5, NOTE_D5, NOTE_E5, NOTE_FS5,
  // Phrase 2: Countdown Motif
  NOTE_B4, NOTE_C5, NOTE_B4, NOTE_A4, NOTE_FS4, NOTE_A4, NOTE_B4,

  NOTE_B4, NOTE_B4, NOTE_B4, NOTE_B6
};

// Note Durations: 
// 2 = half, 4 = quarter, 8 = eighth, 16 = sixteenth
int DemoDurations[] = { 
  // Durations for Intro Fanfare (all 16th notes)
  16, 16, 16, 16, 16, 16, 16, 16,
  // Durations for Countdown Motif
  8, 8, 8, 8, 4, 8, 2,

  2, 2, 2, 1
};
```
### Fucntion for Buzzer
In the `Buzzer` namespace, 
```cpp
namespace Buzzer {
void Init();
void PlayMelody(int* melody, int* durations);
}
```

### `void Buzzer::Init()` 
Initialization of the Buzzer pin to `OUTPUT` 
```cpp
void Buzzer::Init(){
  pinMode(Pinout::Buzzer, OUTPUT);
}
```


### `void PlayMelody(int* melody, int* durations)`
@brief Play the melody with the respective duration of each note once

@para `int* melody` Pointer of the melody (e.g. DemoMelody)

@para `int* durations` Pointer of the duration (e.g. DemoDurations)

* Noted that the length of the `int melody` and `int durations` MUST BE THE SAME, otherwise the function will not work properly, 

```cpp
void Buzzer::PlayMelody(int *melody, int *durations){
      // Calculate the number of notes in the melody
    int numNotes = sizeof(melody) / sizeof(melody[0]);
    
    // Iterate over the notes
    for (int thisNote = 0; thisNote < numNotes; thisNote++) {
      
      // To calculate the note duration, take one second divided by the note type.
      // e.g., quarter note = 1000 / 4, eighth note = 1000/8, etc.
      int noteDuration = 1000 / durations[thisNote];
      
      // Play the note using the tone() function
      tone(Pinout::Buzzer, melody[thisNote], noteDuration);
      
      // To distinguish between notes, set a minimum time between them.
      // The note's duration + 30% (to create a short pause) works well.
      int pauseBetweenNotes = noteDuration * 1.30;
      delay(pauseBetweenNotes);
      
      // Stop the tone playing (creates a cleaner staccato effect)
      noTone(Pinout::Buzzer);
    }
    // Final cleanup - ensure the PWM channel is detached
    ledcDetach(Pinout::Buzzer);

}
```

