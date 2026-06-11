# Oak-D Lite Hand Gesture Control 
## Project Overview
This part of the project uses an OAK-D Lite camera connected to a Raspberry Pi to detect hand gestures using MediaPipe Hands. The Raspberry Pi sends gesture-based commands to an Arduino Mega through USB serial communication using PySerial. The Arduino then controls a servo motor and stepper motor.

**System Flow**

OAK-D Lite Camera

        ↓

Raspberry Pi + MediaPipe Hands

        ↓

Gesture Classification

        ↓

PySerial USB Communication

        ↓

Arduino Mega

        ↓

Servo / Stepper Motor Control

## Purpose

The goal of this module is to prove that hand gestures can be used as a high-level control input for motor actuation. The current focus is on verifying the software pipeline and communication between the Raspberry Pi and Arduino before fully integrating it into the robot system.

## Additional Hardware Used
* Arduino Mega
* DRV8825 Stepper motor driver
* Stepper Motor
* Servo Motor
* Matek UBEC DUO DC convertor

# Power Setup

15V LiPo battery → Matek UBEC DUO

OUT-1: 12V output → DRV8825 stepper motor driver

OUT-2: 5V output → servo motor

Arduino Mega controls the signal pins.
Raspberry Pi communicates with Arduino through USB serial.
All grounds are connected together.

# Pin Connections 
| Component               | Connection     |
| ----------------------- | -------------- |
| Servo signal            | Arduino pin 5  |
| Servo VCC               | UBEC OUT-2 5V  |
| Servo GND               | Common GND     |
| DRV8825 DIR             | Arduino pin 2  |
| DRV8825 STEP            | Arduino pin 3  |
| DRV8825 VMOT            | UBEC OUT-1 12V |
| DRV8825 GND             | Common GND     |
| DRV8825 RESET/SLEEP     | VDD            |
| DRV8825 ENABLE          | GND            |
| Raspberry Pi to Arduino | USB serial     |

# Software Dependenices 
**Main libaries:**
* Python 3
* DepthAI
* OpenCV
* MediaPipe
* Flask
* PySerial
* Arduino Servo libary

Install on the RPI inside the virtual environment

    cd ~/hand_gesture_project
    source oak_env/bin/activate
    pip install depthai opencv-python mediapipe flask pyserial

Working setup used:

    numpy 1.26.4
    opencv-contrib-python 4.11.0.86
    mediapipe 0.10.18
    depthai 3.6.1


## Raspberry Pi Code Description

gesture_servo_stepper.py starts the OAK-D Lite camera through DepthAI, processes each frame with MediaPipe Hands, classifies the hand gesture, and sends a command to the Arduino through PySerial.

## Arduino Code Description
The Arduino Mega waits for serial commands from the Raspberry Pi. When it receives a command, it moves the servo to a target angle or sends step pulses to the DRV8825 stepper driver.

## Gesture Mapping 
The current gesture system uses four gesture commands to select a servo position and extend the stepper motor. A separate fist gesture is used to return the stepper motor back to its rest position.

| Gesture | Raspberry Pi Command | Arduino Action |
|---|---|---|
| Point | `Command_1` | Servo moves to 0°, then stepper extends |
| Peace sign | `Command_2` | Servo moves to 90°, then stepper extends |
| Three fingers | `Command_3` | Servo moves to 180°, then stepper extends |
| Open palm | `Command_4` | Servo moves to 270°, then stepper extends |
| Fist | `RETURN` | Stepper reverses back to rest position |


## Testing Procedure
1. Verify the Arduino appears as /dev/ttyACM0 or /dev/ttyACM1.
2. Run a basic PySerial test to confirm Raspberry Pi → Arduino communication.
3. Test the servo and stepper from Arduino Serial Monitor.
4. Run gesture_servo_stepper.py.
5. Open the Flask camera stream in a browser.
6. Show thumb, pointer, peace sign, or fist gestures.
7. Confirm the correct command is printed and the Arduino responds.