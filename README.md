# Smart EV Charging Station Simulation ⚡🚗

This project simulates an Electric Vehicle charging station with safety locks and serial telemetry output using a PIC18F4580 microcontroller. 

**Why this is important:** EV chargers lock the cable plug into the vehicle socket during active high-power current flow to prevent arcing. Releasing the cable requires a coordinated mechanical lock disengagement.

## 🚀 Core Features

* **Plug Connection Detection:** Simulates plugging the charger into the car. The DC motor locks the plug in place (runs forward).
* **UART Telemetry:** While charging, the system continuously broadcasts voltage, current, and state-of-charge (SOC %) information over serial TX.
* **Manual Overrides:** Toggles the charge speed (Normal vs Fast charging rates).
* **Safety Cut-off:** Emergency Stop halts the process instantly, unlocks the plug (runs motor backward for 2 seconds), and alerts the operator.

## 🔌 Hardware Connections (Matrix)

| PIC18 Pin | Target Component | Functional Purpose |
| :--- | :--- | :--- |
| **RB0** | Connector Plug Insert Switch | Simulates plug connection detection (Active LOW) |
| **RB1** | Charge Speed Button | Normal vs Fast rate selector (Active LOW) |
| **RB2** | Emergency Stop Button | Safety shutdown override (Active LOW) |
| **RB4-RB7** | LCD D4-D7 | 4-bit mode data lines (Conflict-free PORTB) |
| **RA0, RA1** | LCD RS, EN | LCD Register Select and Enable control pins |
| **RC6 / TX** | Virtual Terminal RXD | Serial telemetry broadcaster (9600 Baud) |
| **RC7 / RX** | Virtual Terminal TXD | Command input link (Unused in this simulation) |
| **RD0, RD1** | L293D IN1, IN2 | Plug Solenoid Latch Actuator (DC Motor) |
| **RD4** | Green LED | Station Available / Fully Charged (Add 220Ω resistor) |
| **RD5** | Red LED | System Fault / Lock Error (Add 220Ω resistor) |
| **RD6** | Yellow LED | Charging Active status (Blinks during charging, add 220Ω) |

## 🧠 State-Machine Architecture

The firmware implements an Event-Driven Finite State Machine (FSM) using a simple control variable: `charge_state`. The microcontroller continuously loops, executing specific actions depending on the active state and polling the button pins and UART registers for transitions.

| State Index | Nominal Description | Actions | Next State Transition |
| :--- | :--- | :--- | :--- |
| **State 0** | Standby (Ready) | Green LED ON, Red & Yellow OFF, Motor stopped. LCD prints "STATUS: READY". Polls plug sensor. | Move to State 1 if RB0 goes LOW (Plug inserted). |
| **State 1** | Locking Cable | Runs cable lock DC motor Forward (RD0=1, RD1=0) for 2s. LCD prints "LOCKING PLUG". | Move to State 2 automatically after motor stops. |
| **State 2** | Charging Active | Green LED OFF. Yellow LED toggles/flashes. Broadcasts telemetry logs over UART TX. Polls charge speed toggle. | Move to State 3 when battery level reaches 100%. |
| **State 3** | Charge Complete | Yellow LED OFF, Green LED ON. Runs cable lock DC motor in Reverse (RD0=0, RD1=1) for 2s to release cable. | Move back to State 0 after a 4-second delay. |
| **State 4** | Emergency Fault | Triggered immediately if E-Stop button (RB2) is pressed. Stops motor, unlocks cable, and locks program. | System locked in infinite hazard loop. |

*Note: This code utilizes beginner-friendly direct pin control (e.g., `LATDbits.LATD0 = 1;`) rather than binary bit masks for maximum readability.*

## 🎮 How to Use & Test the Project

You can test the simulation using two distinct interactive methods inside Proteus:

### Method 1: Interactive Serial Keyboard Inputs
1. Run the Proteus simulation and locate the **Virtual Terminal** window. (Right-click device -> *Display Virtual Terminal* if hidden).
2. Click inside the terminal to focus.
3. Press `S` to simulate plugging in the cable.
4. Press `F` while charging to toggle Normal (AC) / Fast (DC) rates.
5. Press `K` to simulate aborting the process (motor safely unlocks).
6. Press `E` to trigger an immediate emergency stop routine.

### Method 2: Physical Hardware Push-Buttons
* Press **RB0** to simulate physical connector insertion.
* Press **RB1** to manually toggle between AC and DC charging speeds.
* Press **RB2** to initiate a physical E-Stop. The motor reverses immediately to release the connector, and the system shuts down.

## Circuit Diagram
