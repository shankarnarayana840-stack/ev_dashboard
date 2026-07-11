
# Real time Electric Vehicle ADAS & Dashboard System
=======
Real-Time Electric Vehicle Dashboard & ADAS Warning System


An advanced, real-time electric vehicle (EV) telemetry and Advanced Driver Assistance System (ADAS) simulated on an STM32 Blue Pill and visualised on a custom Python instrument cluster.

## 1. Project Title

 Real time Electric Vehicle ADAS & Dashboard System

## 2. Project Overview

Modern electric vehicles generate vast amounts of real-time sensor data, including speed, battery State of Charge (SOC), motor temperature, torque, and range. This project addresses the challenge of monitoring vehicle metrics and Advanced Driver Assistance Systems (ADAS) alerts simultaneously. It features an STM32F103C8T6 Blue Pill microcontroller executing a deterministic state machine to process sensor inputs, perform safety calculations, and manage faults. The processed vehicle and ADAS data are streamed over a high-speed UART connection to a real-time Python dashboard built with Matplotlib, creating a unified automotive instrumentation and safety alert system. This setup replicates a production ECU/VCU pipeline using a cost-effective desktop simulation stack.

## 3. Key Features

- **Real-Time EV Dynamics Simulation:** Models speed, torque, SOC, and range using a physics-based vehicle dynamics model.
- **Advanced Driver Assistance Systems (ADAS):** Implements Forward Collision Warning (FCW), Blind Spot Detection (BSD), Time-To-Collision (TTC) calculations, and Parking Assist (PA).
- **Deterministic State Machine:** Manages transitions between PARKED, READY, DRIVING, REGEN, and FAULT states.
- **High-Performance Telemetry:** Streams dual structured telemetry packets over UART via DMA at 10 Hz.
- **Interactive UART Command Shell:** Allows live parameter injection (speed, SOC, faults) and mode switching directly from a terminal.
- **Tiered Alarm & Fault Safeguards:** Features a 4-level alarm hierarchy (Advisory to Critical) and immediate motor PWM cut-off on critical faults.
- **Polished Python Instrument Cluster:** Renders a high-fidelity instrument panel containing a speedometer gauge, SOC bar, speed history plot, and an ADAS bird-eye view.

## 4. System Architecture

The system consists of three main layers: Perception, Control, and Application.

- **Perception Layer:** Gathers driver commands (accelerator/brake) and physical variables via ADC channels, alongside distance readings from front, left, and right ultrasonic sensors.
- **Control Layer (STM32):** Houses the core EV Control ADAS Engine, vehicle state machine, and fault handler, driving alarms and PWM outputs.
- **Application Layer (Python GUI):** Parses serial frames and renders real-time dials, charts, and spatial warning graphics.

### High-Level Architecture Block Diagram

```
+-------------------+      +-------------------+      +--------------------+
|  SENSORS (ADC)    | ---> |   STM32F103C8T6   | ---> |  USART1 Telemetry  |
|  Potentiometers & |      |   EV Control &    |      |  115200 bps (DMA)  |
|  3x HC-SR04 Echo  | <--- |   ADAS Engine     |      +---------+----------+
+-------------------+      +-------------------+                |
                                                                  v
+-------------------+                                  +--------+----------+
| Buzzer Tone (PWM) | <--------------------------------+  Python Dashboard |
| LED Indicators    |                                  |  Matplotlib GUI   |
+-------------------+                                  +-------------------+
```



## 5. Hardware Components

- **STM32F103C8T6 Blue Pill:** 32-bit ARM Cortex-M3 microcontroller operating at 72 MHz, acting as the Vehicle Control Unit (VCU).
- **HC-SR04 Ultrasonic Sensors (3x):** Simulates front, left, and right obstacle tracking (2–400 cm range, ±3 mm accuracy).
- **Passive PWM Buzzer:** Driven by Timer 4 PWM (PB9) for multi-tone alert and alarm escalation.
- **Analog Potentiometers (4x):** Simulate accelerator pedal, brake pedal, battery SOC, and motor temperature inputs.
- **Status LEDs (4x):** Visual indicators for collision warnings, left/right blind spots, and system faults.
- **ST-Link V2:** SWD programmer and debugger.

## 6. Software Stack

- **STM32 Firmware:** Built using STM32CubeIDE, utilising the STM32 Hardware Abstraction Layer (HAL).
- **Peripheral Drivers:** Built-in drivers for 12-bit ADC (PA0-PA2), Timer Interrupts (TIM1, TIM3), Input Capture (TIM2), PWM (TIM4), and USART1 with DMA.
- **Python Dashboard Application:** A real-time visualisation app executing at 10 fps (100 ms intervals).
- **Dashboard Libraries:** Matplotlib (gauges and rolling charts), NumPy (data handling), and PySerial (robust serial connection).

## 7. Technologies Used

- **Languages:** C / C++ (Embedded Firmware development), Python (Desktop visualization)
- **Communications:** UART / USART Serial Protocol
- **DMA Controller:** Direct Memory Access for low-CPU serial transfers
- **Peripherals:** ADC (Analog to Digital Conversion)
- **Tools:** PICSimLab (Hardware Simulation environment),vspe tool (for virtual com pairs)

## 8. Folder Structure

The repository is organised as follows:

```
STM32-EV-ADAS/
├── README.md
├── images/
│   ├── architecture_diagram.png
│   ├── dashboard_normal.png
│   ├── picsimlab.png
│   ├── dashboard_collision.png
│   └── uart_output.png

## 9. Project Workflow / Data Flow

The system processes data sequentially to ensure safety-critical reactions occur within one loop cycle (≤ 10 ms):

1. **Sensor Acquisition:** The STM32 reads analog inputs (pedals, temp) at 100 Hz using 12-bit ADC via TIM1, and triggers HC-SR04 ultrasonic sensors sequentially.
2. **Vehicle Dynamics Processing:** The EV Control module computes torque, speed (clamped 0-200 km/h), power, SOC, and remaining range every 10 ms.
3. **ADAS Proximity Evaluation:** Side blind spots are monitored, and front obstacles are analysed to compute Time-To-Collision (TTC = distance / speed).
4. **Alarm & Safeguard Logic:** Based on alarm priorities (P0 to P3), the system drives physical LEDs and the buzzer PWM, shifting states to FAULT and cutting motor PWM if thresholds are breached.
5. **Telemetry Streaming:** Telemetry is serialised into formatted frames and dispatched via USART1 DMA at 10 Hz.
6. **GUI Rendering:** Python reads, parses, and updates the local state dictionary, refreshing the user-facing Matplotlib dashboard at 10 Hz.

## 10. Core Functionalities

- **EV Dynamics Engine:** Simulates a physics-based inertia model. Integrates motor torque and vehicle mass to update speed, while State of Charge (SOC) is tracked across a simulated 20 kWh battery.
- **Regenerative Braking:** Engages negative motor torque (up to -80 Nm) when the brake pedal is pressed past 5% while in motion, replenishing battery SOC.
- **Shiftable Drive Profiles:** Supports ECO (scaled torque at 0.6x, 25 Wh/km), NORMAL (1.0x torque), and SPORT (increased torque at 1.3x, 35 Wh/km) drive modes.
- **ADAS Obstacle Detection:** Tracks front proximity and side clearances. Left and right Blind Spot Detection (BSD) is active above 20 km/h.
- **UART Command Shell:** Offers an interactive command interface for diagnostics, testing, parameter injection (such as overriding SOC or speed), and drive-mode toggling.

## 11. Safety Features

- **Motor Over-Temperature:** Triggers a critical FAULT_OT if the motor temp exceeds 90 °C, turning off PWM output.
- **Low Battery Safeguard:** Triggers a critical FAULT_SOC if battery State of Charge falls below 2%, cutting motor power.
- **Critical Proximity / Collision:** Enforces immediate transition to FAULT_COL and cuts motor PWM if front obstacle distance falls under 20 cm or Time-To-Collision (TTC) is less than 1.5 seconds.
- **Mute & Graceful Degradation:** Monitors sensor timeouts (FAULT_SEN) and communication losses (FAULT_COM), logging warning entries rather than disabling vehicle movement.

## 12. UART Telemetry Overview

The microcontroller transmits two structured ASCII lines every 100 ms at 115200 bps:

- **Line 1 (EV Telemetry):** Streams SPD (Speed), SOC (State of Charge), TRQ (Torque), TMP (Temperature), RNG (Range), ACC (Accelerator), and BRK (Brake).
- **Line 2 (ADAS Proximity):** Streams side clearances (L, R), front clearance (F), computed TTC, collision indicators (COL), blindspot state bits (BSD), alarm levels (ALM), and fault registers (FLT).

For detailed packet layouts, refer to the documentation in the `docs/` folder.

## 13. Python Dashboard Overview

A professional dark-themed Matplotlib HMI that processes incoming telemetric frames at 10 fps:

- **Circular Speedometer:** Arc dial changing colour dynamically (green to orange to red) up to 200 km/h.
- **Battery & Range Panel:** Displays battery SOC with an integrated estimated range and drive mode badges.
- **ADAS Bird-Eye View:** Renders ego vehicle layout, highlights left/right blind spots in amber on side threat, and draws front obstacle collision levels.
- **Rolling Speed Graph:** Plotting speed trends over a 60-sample window.
- **EV Metrics Table:** Text readout of auxiliary measurements including brake, throttle, alarm thresholds, and UART connection stability.

## 14. Quick Start Guide

### Firmware Setup

1. Import the `firmware/` directory into STM32CubeIDE.
2. Compile and flash the binary onto your STM32 Blue Pill using an ST-Link V2.
3. (Optional Simulation) Import configuration into PICSimLab and load the generated `.hex` file.

### Dashboard Execution

1. Ensure Python 3.x is installed.
2. Navigate to the dashboard directory and install requirements:

   ```
   pip install -r requirements.txt
   ```

3. Run with physical/simulated hardware (e.g., on COM3):

   ```
   python dashboard.py --port COM3
   ```

4. Run in offline presentation/demo mode:

   ```
   python dashboard.py --demo
   ```

For detailed troubleshooting, refer to the documentation in the `docs/` folder.

## 15. Expected Output / Screenshots

The following screenshots demonstrate the key features of the project:

- **System Architecture:** `images/architecture_diagram.png` – High-level block diagram showing the interaction between the STM32 controller, sensors, UART communication, and the Python dashboard.

- **Python Dashboard:** `images/dashboard_normal.png` – Real-time dashboard displaying vehicle speed, battery State of Charge (SOC), range, motor temperature, and ADAS indicators.

- **PICSimLab Simulation:** `images/picsimlab.png` – STM32 Blue Pill simulation environment with connected peripherals used for testing the embedded firmware.

- **ADAS Warning Demonstration:** `images/dashboard_collision.png` – Dashboard showing an active Forward Collision Warning (FCW) when the vehicle approaches an obstacle within the critical threshold.

- **UART Telemetry Output:** `images/uart_shell_demo.png` – Serial terminal displaying real-time EV and ADAS telemetry transmitted from the STM32 over UART.

## 16. Skills Demonstrated

- **Bare-Metal Firmware:** C development, STM32 HAL, NVIC priority grouping, and hardware register configurations.
- **Hardware Timers:** Configuring high-resolution timers for input capture, multi-channel ADC triggering, and PWM generation.
- **Low-Overhead Telemetry:** Designing packet frames with serial-ring buffers, error-checking, and DMA transfers.
- **Safety-Critical Engineering:** Building safety-critical automotive control loops with deterministic state logic and watchdogs.
- **Python GUI Programming:** Matplotlib animation frameworks, asynchronous data queues, and PySerial data management.

## 17. Future Improvements

- Upgrade serial physical layer from UART to robust, automotive-standard CAN Bus protocol.
- Move the Python dashboard to a standalone UI library (such as PyQt/PySide) for smoother render performance.
- Transition STM32 software scheduling to a real-time OS (FreeRTOS) for enhanced task isolation.
- Integrate active steering-assist simulation onto the Bird-Eye diagram.
