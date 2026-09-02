# 🔥 Fire Extinguisher Robot Car

An autonomous robot car that detects fire, drives toward it, sprays water to put it out, and streams a live camera feed to a mobile phone over the Blynk IoT platform.

Built as a project for **Embedded System and Internet of Things Laboratory (CSE 3106)**, Department of Computer Science and Engineering, Khulna University of Engineering & Technology (KUET).

## Team

| # | Name | Student ID |
|---|------|-----------|
| 1 | Mukta Rani Baishnob | 2207056 |
| 2 | Shohana Akter Rabina | 2207057 |

## Features

- 🔥 Detects flame presence and direction using three flame sensors (left, front, right)
- 🚗 Drives toward the fire using a differential-drive chassis (L298N + BO motors)
- 💧 Sprays water through a servo-aimed nozzle once the fire is close
- 🛑 Stops spraying automatically once the flame is no longer detected
- 📷 Streams a real-time camera view to a phone via ESP32-CAM
- ☁️ Reports live status (fire / spraying / idle) to the Blynk app
- 🎮 Manual drive control from the app 

## System Architecture

![System block diagram](docs.pdf)

Two controllers split the work:
- **Arduino UNO** — handles everything real-time and safety-critical: reading the flame sensors, driving the motors, aiming the servo, and switching the pump.
- **ESP32-CAM** — handles everything Wi-Fi related: streaming live video and running the Blynk client, so it doesn't compete with the Arduino for timing.

The two boards talk over a simple UART link: the Arduino sends a one-character status flag (`F` = fire seen, `S` = spraying, `N` = idle) that the ESP32-CAM forwards to Blynk.

## How It Works

1. On power-up, the servo centers, the pump stays off, and the motors are stopped.
2. The Arduino continuously reads all three flame sensors.
3. No flame detected → stay stopped and keep scanning.
4. Flame detected on one sensor → steer toward it (forward for the front sensor, pivot turn for a side sensor).
5. Flame detected on **all three** sensors → the fire is judged close and centered, so the car stops.
6. The servo sweeps the nozzle side to side while the pump switches on and sprays.
7. Once no sensor has seen a flame for 1.5 seconds straight (debounced, so a flicker doesn't cut it short), the pump switches off, the nozzle re-centers, and the car resumes scanning.
8. In parallel, the status flag is sent to the ESP32-CAM every cycle, which relays it to Blynk alongside the live video feed.

## Hardware Components

| Component | Role |
|---|---|
| Arduino UNO | Main controller — sensors, motors, pump, servo |
| Flame sensors ×3 | Detect presence and rough direction of a flame |
| L298N motor driver | Drives the two BO motor channels |
| BO motors + wheels ×4 | Differential-drive locomotion |
| Mini servo (SG90) | Aims and sweeps the sprayer nozzle |
| 5V water pump | Pushes water from the bottle through the sprayer |
| Transistor / MOSFET | Low-side switch for the pump |
| Small water bottle + sprayer nozzle | Water reservoir and delivery |
| ESP32-CAM | Live video streaming + Blynk client |
| Breadboard, jumper wires | Prototyping and interconnects |
| Capacitor | Smooths the supply rail against pump/servo current spikes |
| Battery pack | Powers motors, logic, pump, and ESP32-CAM |

## Pin Connections

**Arduino UNO**

| Pin | Connects to |
|---|---|
| D2 | ESP32-CAM TX (Arduino RX) |
| D3 | ESP32-CAM RX (Arduino TX) |
| D5 | L298N ENB |
| D6 | L298N IN4 |
| D7 | L298N IN3 |
| D8 | L298N IN2 |
| D9 | L298N IN1 |
| D10 | L298N ENA |
| A0 | Left flame sensor (DO) |
| A1 | Front flame sensor (DO) |
| A2 | Right flame sensor (DO) |
| A4 | Servo signal |
| A5 | Pump driver signal |
| 5V | Sensors + servo supply |
| GND | Common ground for all modules |

**L298N motors:** Left motor pair → OUT1/OUT2 · Right motor pair → OUT3/OUT4 · `+12V`/`GND` → battery

**ESP32-CAM:** `U0T (TX)` → Arduino D2 · `U0R (RX)` → Arduino D3 · `GND` → common ground · `5V` → its own stable supply (don't share the Arduino's 5V rail — the camera needs more current than that can reliably provide)

> **Note:** Arduino GND, L298N GND, and ESP32-CAM GND must all be tied together as a common ground.

## Getting Started

### Requirements

- [Arduino IDE](https://www.arduino.cc/en/software) (1.8.x or 2.x)
- `Servo` library (bundled with the Arduino IDE)
- ESP32 board package (for the ESP32-CAM sketch) — install via **Boards Manager** → search "esp32"
- Blynk library + a Blynk account/template for the ESP32-CAM

### 1. Wire the hardware

Follow the [Pin Connections](#pin-connections) table and the [system diagram](docs.pdf) above. Double-check the common ground before powering anything on.

### 2. Flash the Arduino

1. Open `firmware/fire_extinguisher_car.ino` in the Arduino IDE.
2. Select **Board: Arduino UNO** and the correct COM port.
3. Upload.

### 3. Set up the ESP32-CAM + Blynk

1. Create a Blynk template for the project and note the template ID / auth token.
2. Flash the ESP32-CAM with your Wi-Fi credentials and Blynk auth token.
3. In the Blynk app, add widgets for: fire status, spraying status, and a link/webview to the ESP32-CAM's video stream.

### 4. Power on and test

Power the motors/pump circuit and the ESP32-CAM from adequately rated supplies (see the [report](#report) for the power plan), then bring a small flame near the sensors to test detection, steering, and spraying.

## Repository Structure

```
fire-extinguisher-robot-car/
├── README.md
├── fire_extinguisher_car.ino     # Arduino sketch (sensing, driving, spraying)
└── docs/
    ├── system_diagram.png            # System block diagram
```

## Report

A full project report — objective, introduction, theory, workflow, circuit diagrams, discussion, and conclusion — is available separately as a Word document.

## Future Work

- Manual drive control from the Blynk app
- Switch flame sensors to analog reading for graded distance/intensity, instead of the current digital on/off detection
- Add a water-level check so the robot flags when the tank runs low

## License

Add a license of your choice (e.g. MIT) before making the repo public.

