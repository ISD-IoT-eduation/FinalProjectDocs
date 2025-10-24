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

## Inertial Measurement Unit (IMU)

## Ultrasonic Sensor