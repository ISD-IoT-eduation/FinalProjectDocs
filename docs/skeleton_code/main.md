# Main Code `.ino` File
This section include the basic of FreeRTOS, Tasks, PID Control and detail of the code in `.ino` file.

Most of the code will be written in this file for your project. 

## FreeRTOS 
For detail, please visit FreeRTOS official website: https://www.freertos.org/Documentation/02-Kernel/04-API-references/01-Task-creation/00-TaskHandle


### Task
Take blinking a LED as an example, a “Task” consists of:

- Stack
- TCB
- Handle

Inside the task function, there are 2 major parts: setup (only run once) and loop (run forever)

Please be reminded that vTaskDelay() MUST be added inside the while loop. 
```cpp
/*------------LED Blinking Task-------------
  --------Stack and Handle Settings---------
To ensure there is visualization that the program is running*/
StackType_t uxBlinkTaskStack[configMINIMAL_STACK_SIZE];
StaticTask_t xBlinkTaskTCB;
TaskHandle_t BlinkTaskTCB;



void Blink(void *pvPara)
{
  /*Setup for the task*/
  pinMode(LED1, OUTPUT);
  /*DO*/
  while(true){
  digitalWrite(LED1,HIGH);
  vTaskDelay(100);
  digitalWrite(LED1, LOW);
  vTaskDelay(200);
  }
}
```

### Core Configuration of ESP32S3

| Core | Purpose                     |
| ---- | --------------------------- |
| 0    | Wifi / Bluetooth Connection |
| 1    | Local Control               |

### `xTaskCreatePinnedToCore()`
Create FreeRTOS tasks with proper namespaced functions
Task creation function:
```cpp
  TaskHandle_t xTaskCreateStaticPinnedToCore(
      TaskFunction_t pxTaskCode,
      const char *const pcName,
      const uint32_t ulStackDepth,
      void *const pvParameters,
      UBaseType_t uxPriority,
      StackType_t *const puxStackBuffer,
      StaticTask_t *const pxTaskBuffer,
      const BaseType_t xCoreID
  );
```

Example of creating the task of blinking and pin to core 1 of ESP32S3. 
```cpp
xTaskCreatePinnedToCore(Blink, "Blink", 2048, NULL, 1 , &BlinkTaskTCB, 1 );
```

## `ChassisSkeletonCode.ino` 
These files should be included: 
```cpp
#include <Arduino.h>
#include "Pinout.hpp"
#include "MotorControl.hpp"
#include "IRSensors.hpp"
#include "Movement.hpp"
#include "MFRC522_I2C.hpp" //Driver for RFID Reader
#include "UltrasonicSensor.hpp"
#include "Buzzer.hpp" //Buzzer in case you want to debug using buzzer
#include "pitches.h" //For Buzzer
#include "IMU.h" //Optional IMU driver used in sensor lab, 
                  // Please refer to the Sensor lab code for how to use IMU API 
```

### PID Control
This section will explain and guide you how to use the PID driver provided in the skeleton code. 
#### PID_t (struct)
Using struct for handling multiple PIDs (i.e. Speed of Left & Right wheel motor)

Also, create a member function to compute the PID using different set of PID para. 
```cpp
{
  /*Creating the parameters for PID*/
  volatile float Kp;
  volatile float Ki;
  volatile float Kd;

  volatile float target_val; // The target RPM
  float actual_val;          // Actual RPM Reading
  float err;                 // Error
  float err_last;
  float integral;

  /*General PID Function*/
  float PID_realize(float temp_val)
  {
    this->err = this->target_val - temp_val;

    this->integral += this->err;

    this->actual_val = this->Kp * this->err + this->Ki * this->integral +
                       this->Kd * (this->err - this->err_last);

    this->err_last = this->err;

    return this->actual_val;
  }

} PID;
```

Create the PID struct (Gobally)
```cpp
/*Global PID controllers for left and right wheels*/
PID_t LeftWheelPID;
PID_t RightWheelPID;
```

### `void SpeedControlTask(void* pvPara)`

Before entering the loop, initialize all the PID value 

In `void SpeedControlTask(void* pvPara)`
```cpp
  /*Setup for the Task*/
  /*----------------------------------------------------*/
  /*Initalize the Speed of the motor*/
  MotorControl::LeftWheel.Speed = 0;
  MotorControl::RightWheel.Speed = 0;

  /*Initialize PID Parameters*/
  /*LeftMotor PID*/
  LeftWheelPID.Kp = 0.0f;
  LeftWheelPID.Ki = 0.0f;
  LeftWheelPID.Kd = 0.0f;
  LeftWheelPID.target_val = 150.0f;
  LeftWheelPID.err = 0.0f;
  LeftWheelPID.err_last = 0.0f;
  LeftWheelPID.integral = 0.0f;

  /*RightMotor PID*/
  RightWheelPID.Kp = 0.0f;
  RightWheelPID.Ki = 0.0f;
  RightWheelPID.Kd = 0.0f;
  RightWheelPID.target_val = 150.0f;
  RightWheelPID.err = 0.0f;
  RightWheelPID.err_last = 0.0f;
  RightWheelPID.integral = 0.0f;
  /*----------------------------------------------------*/
```
After initializing all the para., get the current RPM of the wheels and store them in struct RPMCounter_t. 

Then set the target value of the PID as `MotorControl::LeftWheel.Speed` & `MotorControl::RightWheel.Speed`, actual value as `LeftWheelRPM.rpm` & `RightWheelRPM.rpm`. 

Finally, run the PID function to mantain the speed of wheels. 
```cpp
  while (true) {
    /*----------------------------------------------------*/
    /*Get the RPM from Encoder*/
    Encoder::RPMCounterFromEncoder(LeftWheelRPM);
    Encoder::RPMCounterFromEncoder(RightWheelRPM);
    /*----------------------------------------------------*/
    /*Compute the PID and Write the Result to Speed of the Wheel*/
    MotorControl::LeftWheel.Speed = LeftWheelPID.PID_realize(LeftWheelRPM.rpm);
    MotorControl::RightWheel.Speed =RightWheelPID.PID_realize(RightWheelRPM.rpm);
    /*----------------------------------------------------*/
    /*FOR DEBUG USAGE*/
    Serial.print("Speed Left: ");
    Serial.println(MotorControl::LeftWheel.Speed);

    Serial.print("Speed Right: ");
    Serial.println(MotorControl::RightWheel.Speed);
    /*----------------------------------------------------*/
    /*A delay must be added inside each User Task*/
    vTaskDelay(50);
    }
```

### `void MovementTask(boid* para)`
This task is for line tracking (by default)

You can  also write your RFID nagivation code here. 
```cpp
 while (true) {
    // Get the status of the IR sensor and store in IRData
    IRSensors::IRData.state = IRSensors::ReadSensorState(IRSensors::IRData);
    /*-------For Debug Use------*/
    // Serial.print("IR Status: ");
    // Serial.print(IRSensors::IRData.Read_IR_L);
    // Serial.print(IRSensors::IRData.Read_IR_M);
    // Serial.println(IRSensors::IRData.Read_IR_R);

    // Serial.print("Condition: ");
    // Serial.println(IRSensors::IRData.state);
    // Depend on the condition, do line tracking
    // Using switch case for the movement control
    switch (IRSensors::IRData.state) {
      case IRSensors::Middle_ON_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::MoveForward();
        vTaskDelay(100);
        break;

      case IRSensors::Left_Middle_ON_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::RotateLeft();
        vTaskDelay(100);
        break;

      case IRSensors::ALL_ON_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::Stop();
        vTaskDelay(100);
        break;

      case IRSensors::Left_Right_ON_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::Stop();
        vTaskDelay(100);
        break;

      case IRSensors::Middle_Right_ON_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::RotateRight();
        vTaskDelay(100);
        break;

      case IRSensors::Right_ON_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::RotateRight();
        vTaskDelay(100);
        break;

      case IRSensors::Left_ON_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::RotateLeft();
        vTaskDelay(100);
        break;

      case IRSensors::All_OFF_Track:
        LeftWheelPID.target_val = 150.0f;
        RightWheelPID.target_val = 150.0f;
        Movement::Stop();
        vTaskDelay(100);
        break;
    }
        // /*-------For Debug Use------*/
        // Serial.print("LeftWheel.Speed: ");
        // Serial.println(MotorControl::LeftWheel.Speed);
        // Serial.print("RightWheel.Speed: ");
        // Serial.println(MotorControl::RightWheel.Speed);

        // /*-------For Debug Use------*/
        // Serial.print("LeftWheel.PWMIN1: ");
        // Serial.print(MotorControl::LeftWheel.PWMChannelIN1);
        // Serial.println(MotorControl::LeftWheel.PWMChannelIN2);

        // Serial.print("RightWheel.PWMIN2: ");
        // Serial.print(MotorControl::RightWheel.PWMChannelIN1);
        // Serial.println(MotorControl::RightWheel.PWMChannelIN2);
    vTaskDelay(10);
  }
};
```
## `void setup()` Without Firebase
In `void setup()`, 

Initalize all the driver. 
```cpp
  // Initialize Serial communication
  Serial.begin(115200);
  IRSensors::Init();
  MotorControl::DCMotorControl::Init();
  MotorControl::ServoMotorControl::Init();
  Encoder::Init();
  Serial.println("Motors Pin Initialized.");
  UltrasonicSensor::Init();
  Buzzer::Init();
```

Then create the tasks and pin them to the core. 

```cpp
  xTaskCreatePinnedToCore(
    LEDBlinkingTask, "Blinking", 2000, NULL, 1, &LEDBlinkingTaskHandle, 1);

  xTaskCreatePinnedToCore(
    MovementTask, "Movement", 12000, NULL, 2, &MovementTaskHandle, 1);

  xTaskCreatePinnedToCore(
    SpeedControlTask, "SpeedControl", 10000, NULL, 2, &SpeedControlTaskTCB, 1);

  Serial.println("Robot initialized and running");
  vTaskDelay(10);
```


## `void loop()`
THERE SHOULD BE NOTHING INSIDE `loop()`!!!

MCU WILL NOT RUN ANY CODE INSIDE `loop()`!!!