# Motor Control
### `MotorControl.hpp`

# DC Motor
# Setting up the PWM Channel(s) for DC motor
For PWM channel, Setting as below. 
PWM: Frequency, Resolution, Dutycycle
Motor: ID, IN1 Channel, IN2 Channel, Speed 

```cpp
namespace MotorControl {
  // Configuration of DC Motor (Side Wheels)
  struct DCMotor
  {
    // PWM Configuration
    const uint16_t PWMFrequency = 2000;
    const uint8_t PWMResolution = 12;
    uint16_t PWMDuty = 0;
    uint8_t MotorID = 0; // ID = 1 (Left), = 2 (Right)
    uint8_t PWMChannelIN1 =
      0; // ensure the PWM channel for different motor is not the same
    uint8_t PWMChannelIN2 = 0;
    // Adjustable Parameter
    uint16_t Speed = 0; // Init set to 0
  }}
```

# The use of `extern`
To give them external linkage and make them accessible across multiple translation units, extern must be explicitly used in their declarations in all files, including the one where they are defined. 
### In `MotorControl.hpp`, 
```cpp
  extern DCMotor LeftWheel;
  extern DCMotor RightWheel;
```
### In `MotorControl.cpp`, 
# Global motor instances
```cpp
namespace MotorControl {
  DCMotor LeftWheel;
  DCMotor RightWheel;

}

```

# Functions for controlling DC motors
```cpp
  namespace DCMotorControl {
  /*Initialization of PWM channels for DC Motors*/
    void Init();
    void TurnClockwise(DCMotor& Motor);
    void TurnAntiClockwise(DCMotor& Motor);
    void Stop(DCMotor& Motor);
  };
```
# `void DCMotorControl::Init()`
```cpp

void MotorControl::DCMotorControl::Init()
{
  // Setup the Motor ID to the struct
  LeftWheel.MotorID = 1;
  LeftWheel.PWMChannelIN1 = 1;
  LeftWheel.PWMChannelIN2 = 2;

  RightWheel.MotorID = 2;
  RightWheel.PWMChannelIN1 = 3;
  RightWheel.PWMChannelIN2 = 4;

  // Setup PWM channels for DC Motors (Side Wheels)
  ledcAttachChannel(Pinout::LeftMotorIn1,
                    LeftWheel.PWMFrequency,
                    LeftWheel.PWMResolution,
                    LeftWheel.PWMChannelIN1);
  ledcAttachChannel(Pinout::LeftMotorIn2,
                    LeftWheel.PWMFrequency,
                    LeftWheel.PWMResolution,
                    LeftWheel.PWMChannelIN2);
  ledcAttachChannel(Pinout::RightMotorIn1,
                    RightWheel.PWMFrequency,
                    RightWheel.PWMResolution,
                    RightWheel.PWMChannelIN1);
  ledcAttachChannel(Pinout::RightMotorIn2,
                    RightWheel.PWMFrequency,
                    RightWheel.PWMResolution,
                    RightWheel.PWMChannelIN2);

  // Set all the PWM Channels' Dutycycle to 0
  ledcWriteChannel(LeftWheel.PWMChannelIN1, 0);
  ledcWriteChannel(LeftWheel.PWMChannelIN2, 0);
  ledcWriteChannel(RightWheel.PWMChannelIN1, 0);
  ledcWriteChannel(RightWheel.PWMChannelIN2, 0);
};
```

### Spinning the DC motor
# Refer to the datasheet of the motor sheet (Motor driver)
# `void MotorControl::DCMotorControl::TurnClockwise(MotorControl::DCMotor& Motor)`
# URL: https://www.ti.com/lit/ds/symlink/drv8871.pdf?ts=1763316592601&ref_url=https%253A%252F%252Fwww.google.com%252F
![MotorDriverDatashhet](assets/MotorDriverLogic.png)
```cpp
/**
 * @brief Spin the motor clockwise, package the function in struct dependent.

 * @param Motor (MotorControl::DCMotor& Motor) (i.e. LeftWheel / RightWheel)

 */

```cpp
void MotorControl::DCMotorControl::TurnClockwise(MotorControl::DCMotor& Motor)
{
  ledcWriteChannel(Motor.PWMChannelIN1, Motor.Speed);
  ledcWriteChannel(Motor.PWMChannelIN2, 0);
};
```
# `void MotorControl::DCMotorControl::TurnAntiClockwise(MotorControl::DCMotor& Motor)`
```cpp
/**
 * @brief Spin the motor anti-clockwise, package the function in struct dependent.

 * @param Motor (MotorControl::DCMotor& Motor) (i.e. LeftWheel / RightWheel)

 */

```

```cpp
void MotorControl::DCMotorControl::TurnAntiClockwise(MotorControl::DCMotor& Motor)
{

  ledcWriteChannel(Motor.PWMChannelIN1, 0);
  ledcWriteChannel(Motor.PWMChannelIN2, Motor.Speed);
};


# `void MotorControl::DCMotorControl::Stop(MotorControl::DCMotor& Motor)`
```cpp
/**
 * @brief Stop the motor, package the function in struct dependent.

 * @param Motor (MotorControl::DCMotor& Motor) (i.e. LeftWheel / RightWheel)

 */
```
# Refer to the datasheet, to stop the DC motor, both of IN1 and IN2 should be pull-high (100% dutycycle, i.e. 4096 for 12-bit PWM resolution)

```cpp
void MotorControl::DCMotorControl::Stop(MotorControl::DCMotor& Motor)
{
  ledcWriteChannel(Motor.PWMChannelIN1, 4096);
  ledcWriteChannel(Motor.PWMChannelIN2, 4096);
};
```

# Encoder for DC Motor
### Setting up the encoder for DC motor (n20)
![DataSheet of N20 Motor](N20MotorSpec.png)


```cpp
// Create a struct to handle 2 motors encoder
struct Encoder_t
{
  int pinAState;
  int pinBState;
  int Encoder_A;
  int Encoder_B;
};
```
```cpp
/*Constants for Encoder
  Find out the encoder resolution by yourself */
const int encoderResolution = 320; // Number of pulses per revolution
const unsigned long interval = 50; // Time interval in milliseconds 50ms
```

```cpp
// Global motor encoder
extern Encoder_t EncoderLeft;
extern Encoder_t EncoderRight;



/*Encoder to RPM Function and Settings
  Creating RPMCounter_t for 2 Wheel Setting
  */
struct RPMCounter_t
{
  volatile int encoderPulses;
  unsigned long previousMillis;
  volatile float rpm;
};

extern RPMCounter_t LeftWheelRPM;
extern RPMCounter_t RightWheelRPM;

namespace Encoder {
  /*Interrupt for the encoder for both left & right wheel*/
  void handleLeftEncoderInterrupt();
  void handleRightEncoderInterrupt();
  void Init();
  void RPMCounterFromEncoder(RPMCounter_t& Counter);
};
```



# Setting up Servo Motor PWM Channel
For Servo Motor, Frequency MUST be 50Hz
Servo Motor: Target angle
```cpp
  // Configuration of Servo Motor (Front Wheel)
  struct ServoMotor
  {
    const uint8_t PWMFrequency = 50; // PWM must be in 50Hz
    const uint8_t PWMResolution = 12;
    uint16_t PWMDuty = 0;
    const uint8_t PWMChannel = 8; // Ideally select between 5-10
    float TargetAngle = 0.0f;
  };

  ServoMotor FrontWheel;

  namespace ServoMotorControl {
    /*Initialization of PWM Channel for Servo Motor*/
    void Init();
    void TurnDeg(ServoMotor& Motor); // in deg
  }
  // Global motor instances


  extern ServoMotor FrontWheel;
```

