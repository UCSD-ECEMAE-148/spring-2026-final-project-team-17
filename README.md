# ECE MAE 148 FINAL PROJECT TEAM 17: Robobutler | Follow the Leader

Small autonomous rover that follows a person while carrying lightweight items. Using onboard vision, the robot will detect and track a human target, maintain a safe following distance, and stop when the person stops. User can use hand gestures to control robot, i.e if user wants it to stop following, present items, etc.

The project is organized around two major pipelines:

1. **Autonomous Navigation:** OAK-D Lite → AprilTag detection → EKF localization → path planning / MPC control → VESC motor control.
2. **Hand Gesture Control:** OAK-D Lite → MediaPipe Hands → Raspberry Pi gesture classification → PySerial → Arduino Mega → servo / stepper control.

At the current stage, the navigation and hand gesture subsystems have been tested separately. Full integration is intended to allow the car to drive toward an AprilTag, stop near the target, then switch into hand gesture control mode.

---

## Table of Contents

- [Project Overview](#project-overview)
- [System Architecture](#system-architecture)
- [Hardware Used](#hardware-used)
- [Power Setup](#power-setup)
- [Pin Connections](#pin-connections)
- [Software Requirements](#software-requirements)
- [Installation](#installation)
- [Running Autonomous Navigation](#running-autonomous-navigation)
- [Running Hand Gesture Control](#running-hand-gesture-control)
- [Gesture Mapping](#gesture-mapping)
- [Project Structure](#project-structure)
- [Testing Procedure](#testing-procedure)
- [Troubleshooting Notes](#troubleshooting-notes)
- [Current Status](#current-status)
- [Future Work](#future-work)
- [License](#license)

---

## Project Overview

This project combines autonomous robot navigation with gesture-based motor control.

The navigation system uses an OAK-D Lite camera to detect AprilTags and estimate the target's position. The robot uses this information, along with depth data and path planning, to generate throttle and steering commands for the VESC.

The hand gesture subsystem uses the OAK-D Lite camera and MediaPipe Hands to classify simple hand gestures. These gestures are converted into serial commands and sent from the Raspberry Pi to an Arduino Mega. The Arduino controls a servo motor and a stepper motor through a DRV8825 stepper driver.

The goal is to eventually connect both systems into one full demo:

```text
Robot detects AprilTag
        ↓
Robot drives toward target
        ↓
Robot stops near target
        ↓
Robot switches to gesture mode
        ↓
User gives hand gesture
        ↓
Arduino actuates servo / stepper mechanism
```

---

## System Architecture

### Autonomous Navigation Pipeline

```text
OAK-D Lite Camera
        ↓
AprilTag Detection
        ↓
Depth / Ground / Obstacle Detection
        ↓
EKF State Estimation
        ↓
Path Planning / MPC Controller
        ↓
VESC Motor Control
        ↓
RC Car Navigation
```

### Hand Gesture Control Pipeline

```text
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
Servo Motor + Stepper Motor
```

---

## Hardware Used

### Navigation Hardware

- OAK-D Lite camera
- Raspberry Pi
- RC car platform
- VESC motor controller
- AprilTag marker
- Battery system for the RC car

### Hand Gesture / Actuation Hardware

- Arduino Mega
- DRV8825 stepper motor driver
- Stepper motor
- Servo motor
- Matek UBEC DUO DC converter
- 15V LiPo battery
- Jumper wires / breadboard for testing
- USB cable from Raspberry Pi to Arduino

---

## Power Setup

The hand gesture actuation system uses a separate power setup for the servo and stepper motor.

```text
15V LiPo Battery
        ↓
Matek UBEC DUO
        ├── OUT-1: 12V → DRV8825 VMOT
        └── OUT-2: 5V  → Servo VCC
```

Important notes:

- The Arduino Mega controls only the signal pins.
- The Raspberry Pi communicates with the Arduino through USB serial.
- All grounds must be connected together.
- The DRV8825 should have a capacitor across VMOT and GND.
- The stepper driver current limit must be tuned before running the motor.

During testing, the stepper driver current limit was tuned to around **1.2 A** using a reference voltage of approximately **0.6 V** on the driver potentiometer.

---

## Pin Connections

| Component | Connection |
|---|---|
| Servo signal | Arduino pin 5 |
| Servo VCC | UBEC OUT-2 5V |
| Servo GND | Common GND |
| DRV8825 DIR | Arduino pin 2 |
| DRV8825 STEP | Arduino pin 3 |
| DRV8825 VMOT | UBEC OUT-1 12V |
| DRV8825 GND | Common GND |
| DRV8825 RESET/SLEEP | VDD |
| DRV8825 ENABLE | GND |
| Raspberry Pi to Arduino | USB serial |
| Raspberry Pi to VESC | USB serial |
| OAK-D Lite to Raspberry Pi | USB |

---

## Software Requirements

### Main Libraries

- Python 3.8+
- DepthAI
- OpenCV
- AprilTag detection library
- NumPy / SciPy
- PySerial
- Ultralytics / FastSAM
- MediaPipe
- Flask
- Arduino Servo library

### Hand Gesture Working Setup

The hand gesture subsystem was tested with:

```text
numpy 1.26.4
opencv-contrib-python 4.11.0.86
mediapipe 0.10.18
depthai 3.6.1
```

---

## Installation

### 1. Clone the Repository

```bash
cd ~
git clone https://github.com/aalfadda-svg/VisCarPath.git
cd VisCarPath
git checkout lalo-tuning
```

Verify the branch:

```bash
git log --oneline -1
```

The demo branch should show commit `2514551 Smooth` or newer.

---

### 2. Create and Activate a Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

On Windows:

```bash
venv\Scripts\activate
```

---

### 3. Install Navigation Dependencies

```bash
pip install -r requirements.txt
pip install pyserial ultralytics
```

For headless/server environments, replace `opencv-python` with `opencv-python-headless` in `requirements.txt` before installing.

---

### 4. Check FastSAM Model

```bash
ls -lh FastSAM-s.pt
```

The file should be around `23M`.

If it is missing or too small:

```bash
wget https://github.com/ultralytics/assets/releases/download/v0.0.0/FastSAM-s.pt
```

---

### 5. Install Hand Gesture Dependencies

If the hand gesture code is stored in a separate project folder:

```bash
cd ~/hand_gesture_project
source oak_env/bin/activate
pip install depthai opencv-python mediapipe flask pyserial
```

If the hand gesture code is included inside this repo, install the extra dependencies inside the main repo environment:

```bash
cd ~/VisCarPath
source venv/bin/activate
pip install depthai opencv-python mediapipe flask pyserial
```

---

## Running Autonomous Navigation

The main navigation script is:

```bash
main_navigation.py
```

### Basic Usage

Navigate to AprilTag ID 0 in visual mode:

```bash
python main_navigation.py --target 0 --visual
```

Navigate to AprilTag ID 0 in headless mode:

```bash
python main_navigation.py --target 0 --headless
```

Run the bare control loop without display or logging:

```bash
python main_navigation.py --target 0
```

### Command Line Options

| Option | Description |
|---|---|
| `--target <id>` | Target AprilTag ID to navigate to |
| `--visual` | Show live RGB + depth windows with obstacle overlay |
| `--headless` | Save debug images to disk without display |
| `--robot-width <m>` | Robot width in meters |
| `--log-dir <path>` | Directory for headless debug images |
| `--fastsam-model <path>` | Path to FastSAM weights file |

---

## Running Hand Gesture Control

The hand gesture script is:

```bash
gesture_servo_stepper.py
```

This script starts the OAK-D Lite camera through DepthAI, processes frames with MediaPipe Hands, classifies gestures, and sends commands to the Arduino through PySerial.

Run:

```bash
cd ~/hand_gesture_project
source oak_env/bin/activate
python3 gesture_servo_stepper.py
```

Then open the Flask stream on a laptop browser:

```text
http://<raspberry-pi-ip>:5000
```

Example:

```text
http://ucsdrobocar-148-07.local:5000
```

or:

```text
http://172.20.10.6:5000
```

---

## Gesture Mapping

The gesture system uses four gestures to select a servo position and extend the stepper motor. A fist gesture returns the stepper motor to its rest position.

| Gesture | Raspberry Pi Command | Arduino Action |
|---|---|---|
| Point | `Command_1` | Servo moves to 0°, then stepper extends |
| Peace sign | `Command_2` | Servo moves to 90°, then stepper extends |
| Three fingers | `Command_3` | Servo moves to 180°, then stepper extends |
| Open palm | `Command_4` | Servo moves to 270°, then stepper extends |
| Fist | `RETURN` | Stepper reverses back to rest position |

---

## Project Structure

| File | Description |
|---|---|
| `main_navigation.py` | Main navigation pipeline integrating perception, state estimation, and control. Supports visual, headless, and bare modes. |
| `apriltag_detection.py` | AprilTag detection and pose estimation using the OAK-D camera. |
| `ground_obstacle_detection.py` | Ground plane detection, obstacle identification, and navigable path mapping using depth data and FastSAM segmentation. |
| `kalman_filter.py` | Extended Kalman Filter for vehicle state estimation using a bicycle motion model. |
| `mpc_controller.py` | Lightweight geometric path controller with Pure Pursuit and P-Control for obstacle avoidance and path following. |
| `vesc_bridge.py` | Direct serial VESC bridge for throttle and steering commands. |
| `debug_oakd_comprehensive.py` | OAK-D debugging and diagnostic suite. |
| `gesture_servo_stepper.py` | OAK-D Lite + MediaPipe hand gesture control script that sends commands to the Arduino. |
| `requirements.txt` | Python dependencies for navigation and perception. |

---

## Testing Procedure

### Navigation Testing

1. Elevate the car or hold it securely before testing the VESC.
2. Confirm the VESC is detected:

```bash
ls /dev/serial/by-id/
```

Look for an STMicroelectronics or ChibiOS entry.

3. Test steering sweep:

```bash
python3 -c "
from vesc_bridge import VESCBridge
import time
v = VESCBridge()
print('RIGHT')
v.send_command(0.0, 1.0)
time.sleep(2)
print('LEFT')
v.send_command(0.0, -1.0)
time.sleep(2)
print('CENTER')
v.send_command(0.0, 0.0)
time.sleep(2)
v.close()
"
```

4. Test throttle slowly:

```bash
python3 -c "from vesc_bridge import VESCBridge; import time; v=VESCBridge(); v.send_command(1.0,0.0); time.sleep(2); v.close()"
```

5. Place AprilTag ID 0 about 1.5m in front of the car.
6. Run:

```bash
python3 main_navigation.py --target 0 --headless
```

---

### Hand Gesture Testing

1. Verify the Arduino appears as `/dev/ttyACM0` or `/dev/ttyACM1`:

```bash
ls /dev/ttyACM*
```

2. Run a basic PySerial test to confirm Raspberry Pi to Arduino communication.
3. Test the servo and stepper from the Arduino Serial Monitor.
4. Run:

```bash
python3 gesture_servo_stepper.py
```

5. Open the Flask camera stream in a browser.
6. Show point, peace sign, three fingers, open palm, or fist gestures.
7. Confirm the correct command is printed and the Arduino responds.

---

## Troubleshooting Notes

### VESC and Arduino Serial Ports Swapping

Linux may assign `/dev/ttyACM0` and `/dev/ttyACM1` differently after rebooting or unplugging devices. For a more stable setup, use `/dev/serial/by-id/` paths or create custom udev symlinks such as:

```text
/dev/vesc
/dev/arduino
```

Recommended code setup:

```text
VisCarPath / VESCBridge  → /dev/vesc
Hand Gesture / Arduino   → /dev/arduino
OAK-D Lite               → camera USB
```

---

### OAK-D Lite Camera Conflicts

Only one program should access the OAK-D Lite at a time. Running the navigation script and gesture script simultaneously can cause camera conflicts.

Recommended mode switch:

```text
NAVIGATION MODE
        ↓
TARGET REACHED
        ↓
STOP VESC
        ↓
RELEASE OAK-D LITE
        ↓
START GESTURE MODE
```

---

### Stepper Driver Current Limit

The DRV8825 current limit must be set before running the stepper motor. During testing, too much current caused the first driver circuit to burn out. The current limit was adjusted to approximately **1.2 A** with a reference voltage of about **0.6 V**, which provided more reliable stepper operation.

---


## Current Status

### Completed

- OAK-D Lite camera pipeline for navigation.
- AprilTag detection and target selection.
- EKF state prediction/update structure.
- VESC throttle and steering command pipeline.
- Raspberry Pi to Arduino serial communication.
- MediaPipe Hands gesture detection.
- Arduino control of servo and stepper motor.
- Separate testing of navigation and gesture subsystems.

### In Progress

- Full integration between AprilTag navigation and hand gesture control.
- Automatic transition from navigation mode to gesture mode after reaching the AprilTag.
- More reliable VESC tuning for smoother throttle and steering.
---

## Future Work

- Finish automatic state transition from AprilTag navigation to gesture mode.
- Add a deadband to prevent small acceleration noise from causing unwanted forward or reverse motion.
- Tune max duty cycle, steering range, and steering gain for smoother driving.
- Disable reverse during demo mode for safety.
- Use stable device names for VESC and Arduino.
- Improve gesture stability by requiring the same gesture for multiple frames.
- Run repeated full-system tests with consistent lighting, tag placement, and power conditions.

---




[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/r686kJSN)
