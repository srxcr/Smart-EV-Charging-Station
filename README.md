# Smart EV Charging Station Simulation ⚡🚗

> **Quick Start:** This project simulates an intelligent Electric Vehicle charging station with automated cable locking, real-time telemetry, and safety controls. A PIC18F4580 microcontroller manages charging sequences, monitors state-of-charge, broadcasts UART telemetry, and implements emergency stop functionality with motor-driven solenoid unlock mechanisms.

## 💡 Project Description

This advanced embedded systems project simulates a complete Electric Vehicle (EV) charging station with real-world safety considerations. The system implements an event-driven finite state machine (FSM) that manages the complete charging lifecycle: plug detection → cable locking → active charging with telemetry → charge completion → safe cable release. The project integrates multiple PIC18F4580 features including GPIO input/output, PWM motor control, LCD interfacing, and UART serial communication—making it ideal for learning complex embedded systems architecture and safety-critical design patterns.

**Why This Project Matters:** Modern EV chargers use solenoid/motor-driven locking mechanisms to prevent accidental cable disconnection during high-power charging (potentially 350A DC). This project demonstrates the coordinated control logic, safety interlocks, and real-time monitoring required in production charging systems.

---

## 📁 Repository Structure

```
Smart-EV-Charging-Station/
├── main.c                    # Core FSM implementation and main loop
├── fsm.c / fsm.h            # Finite State Machine logic (if separate)
├── uart.c / uart.h          # UART telemetry functions
├── motor.c / motor.h        # DC motor control (L293D driver)
├── lcd.c / lcd.h            # LCD display functions
├── adc.c / adc.h            # ADC for battery voltage/current sensing
├── timers.c / timers.h      # Timer interrupts for state timing
├── EVSchematic.png          # Circuit diagram/schematic
├── Proteus_Simulation.pdsprj # Proteus simulation file
├── README.md                # This file
└── [Additional configuration files]
```

---

## ⚡ System Overview

The Smart EV Charging Station simulates a real-world charging system with the following capabilities:

- **Automatic Cable Locking:** DC motor driven solenoid engages when charging starts
- **Real-Time Telemetry:** UART broadcasts voltage, current, and state-of-charge over serial
- **Dual Charging Modes:** Normal (AC simulation) and Fast (DC simulation) charging rates
- **Emergency Stop:** Safety-critical E-Stop button with instant motor unlock and system halt
- **Multi-LED Status Display:** Visual indication of system state (ready, charging, fault)
- **4-Bit LCD Interface:** Real-time display of charging status and metrics
- **Finite State Machine:** Event-driven architecture for robust state management

---

## 🛠️ Hardware Components

### Microcontroller & Core
* **PIC18F4580** – Main microcontroller (40-pin DIP)
* **20 MHz Crystal Oscillator** – System clock
* **100nF Bypass Capacitor** – Power supply decoupling (VDD/VSS)
* **10μF Bulk Capacitor** – Power supply filtering
* **MCLR Pull-up Resistor** – 10kΩ to +5V (reset circuit)

### Motor & Drive Control
* **DC Motor** – 5V-rated, 500-1000 RPM (simulates solenoid latch)
* **L293D H-Bridge Driver** – Bidirectional motor control IC (IN1, IN2 control)
* **L293D Pin Configuration:**
  - Pin 1 (EN1/2): Motor enable (can be tied HIGH for always-on)
  - Pin 2 (IN1): Forward control (RD0)
  - Pin 7 (IN2): Reverse control (RD1)
  - Pin 3,6: Motor output terminals
  - Pin 4,5,12,13: GND
  - Pin 8,16: VCC (+5V)

### Sensing & Input
* **Plug Connection Switch** (RB0) – Normally HIGH, pulled LOW when plug inserted
* **Charge Speed Toggle Button** (RB1) – Normal vs Fast mode selector
* **Emergency Stop Button** (RB2) – Safety-critical override (Active LOW)
* **10kΩ Pull-up Resistors** – For all three input buttons
* **200ms Debounce Delay** – Software debouncing for button inputs

### Output & Display
* **16×2 Character LCD** – Status and charging information display
* **LCD Connection Pins:**
  - RS (Register Select): RA0
  - EN (Enable): RA1
  - D4-D7 (Data lines): RB4-RB7 (4-bit mode)
  - VSS: GND
  - VDD: +5V
  - VO (Contrast): 10kΩ potentiometer

### Status Indicators
* **Green LED** (RD4) – System ready / Fully charged (220Ω series resistor)
* **Red LED** (RD5) – Fault / Error condition (220Ω series resistor)
* **Yellow LED** (RD6) – Charging active / In progress (220Ω series resistor)

### Communication
* **UART/Serial** (RC6 TX, RC7 RX) – 9600 Baud serial telemetry
* **Virtual Terminal** – Proteus simulation or RS-232 serial monitor
* **Telemetry Data:** Voltage (V), Current (A), SOC (%), Charging Mode

### Power Supply
* **5V Regulated Power Supply** – ≥1A capacity
* **Microcontroller current:** ~50mA typical
* **Motor current (peak):** ~500mA
* **Total system capacity:** 1-2A recommended

---

## 🔌 Hardware Connections (Detailed Pin Map)

### Input Buttons (PORTB)

| PIC Pin | Signal Name | Component | Logic | Function |
|---------|-------------|-----------|-------|----------|
| **RB0** | PLUG_IN | Connector Switch | Active LOW | Detect plug insertion |
| **RB1** | CHARGE_SPEED | Mode Toggle | Active LOW | Switch AC/DC charging |
| **RB2** | E_STOP | Emergency Button | Active LOW | Immediate system halt |

### LCD Display (PORTB & PORTA)

| PIC Pin | Signal Name | Component | Purpose |
|---------|-------------|-----------|---------|
| **RA0** | LCD_RS | LCD Register Select | Command/Data select (0=Cmd, 1=Data) |
| **RA1** | LCD_EN | LCD Enable | Pulse to latch data (active HIGH) |
| **RB4** | LCD_D4 | LCD Data line 4 | Bit 0 of data nibble |
| **RB5** | LCD_D5 | LCD Data line 5 | Bit 1 of data nibble |
| **RB6** | LCD_D6 | LCD Data line 6 | Bit 2 of data nibble |
| **RB7** | LCD_D7 | LCD Data line 7 | Bit 3 of data nibble |

### Motor Control (PORTD)

| PIC Pin | Signal Name | L293D Pin | Function | State |
|---------|-------------|-----------|----------|-------|
| **RD0** | MOTOR_FWD | IN1 (Pin 2) | Motor forward control | HIGH = forward |
| **RD1** | MOTOR_REV | IN2 (Pin 7) | Motor reverse control | HIGH = reverse |
| **RD4** | LED_GREEN | - | Ready / Charged status | HIGH = ON |
| **RD5** | LED_RED | - | Fault / Error status | HIGH = ON |
| **RD6** | LED_YELLOW | - | Charging active | HIGH = ON (blink during charge) |

### UART Serial Communication

| PIC Pin | Signal | Baud Rate | Purpose |
|---------|--------|-----------|---------|
| **RC6/TX** | Transmit | 9600 Baud | Send telemetry (V, I, SOC) |
| **RC7/RX** | Receive | 9600 Baud | Command input (unused in this project) |

### Power Distribution

| Component | VDD | GND |
|-----------|-----|-----|
| PIC18F4580 | +5V | GND |
| LCD Module | +5V | GND |
| L293D Driver | +5V, +12V | GND |
| DC Motor | +5V (via L293D) | GND |
| LEDs | Through 220Ω | GND |
| Buttons | +5V (pull-up) | GND (when pressed) |

### ASCII Wiring Diagram

```
                          PIC18F4580 (40-pin DIP)
                          ┌──────────────────────┐
                   +5V ──┤VDD (9,19)             │
                   GND ──┤VSS (8,18)             │
                          │                       │
        Plug In Switch ───┤RB0 (33)     RA0 (2)├─── LCD RS
   Charge Speed Button ───┤RB1 (34)     RA1 (3)├─── LCD EN
    Emergency Stop Button┤RB2 (35)     RC6 (25)├─── UART TX
                          │              RC7 (26)├─── UART RX
                 LCD D4 ──┤RB4 (37)     RD0 (19)├─── Motor FWD (L293D IN1)
                 LCD D5 ──┤RB5 (38)     RD1 (20)├─── Motor REV (L293D IN2)
                 LCD D6 ──┤RB6 (39)     RD4 (27)├─── Green LED
                 LCD D7 ──┤RB7 (40)     RD5 (28)├─── Red LED
                          │              RD6 (29)├─── Yellow LED
                          └──────────────────────┘

L293D Motor Driver Connections:
                    ┌─────────────────┐
              +5V ─┤1 EN1/2      GND 4├─ GND
      Motor FWD ─┤2 IN1         GND 5├─ GND
      Motor OUT ─┤3 OUT1      OUT2 6├─ Motor OUT
                ├┤4 GND        GND 12├─ GND
                ├┤13 GND    +12V 16├─ Motor Supply (optional boost)
      Motor REV ─┤7 IN2     OUT3 9├─ Motor OUT
      Motor OUT ─┤6 OUT2     OUT4 11├─ Motor OUT
                └─────────────────┘

Button Debouncing Circuit:
    +5V
     │
   10kΩ (Pull-up)
     │
     ├────o────┬────► to RB0/RB1/RB2
     │         │
    GND   Push Button

LCD 16x2 Contrast Control:
    +5V
     │
    [10kΩ Pot]
     │ (wiper to LCD pin 3)
    GND
```

---

## 🧠 Finite State Machine (FSM) Architecture

The system uses an event-driven state machine with 5 distinct states. The main loop continuously checks input conditions and transitions between states based on user input or completion conditions.

### State Transition Diagram

```
                    ┌─────────────────┐
                    │   STATE 0       │
                    │   STANDBY       │◄──────┐
                    │  (Green ON)     │       │
                    └────────┬────────┘       │
                             │ RB0 = LOW     │ After 4s delay
                       (plug inserted)      │ (after unlock)
                             │              │
                    ┌────────▼────────┐     │
                    │   STATE 1       │     │
                    │  LOCKING_CABLE  │     │
        (Motor FWD for 2s) │     │
                    └────────┬────────┘     │
                             │              │
                    ┌────────▼────────┐     │
                    │   STATE 2       │     │
                    │  CHARGING       │     │
                    │ (Yellow blinking)     │
                    │ Broadcast UART  │     │
                    └────────┬────────┘     │
                             │ SOC == 100%  │
                       (or user action)    │
                             │              │
                    ┌────────▼────────┐     │
                    │   STATE 3       │     │
                    │  CHARGE_COMPLETE│     │
        (Motor REV for 2s) └────────┬────────┘
                             │
                             └─────────────┘

Emergency Path (ANY STATE):
    RB2 = LOW (E-Stop pressed)
         │
         ▼
    ┌─────────────────────────────┐
    │  STATE 4: EMERGENCY_FAULT   │
    │  • Motor reversed (2s)      │
    │  • All LEDs might flash     │
    │  • Red LED stays ON         │
    │  • Infinite halt loop       │
    └─────────────────────────────┘
```

### State Descriptions & Actions

| State | Name | LCD Display | LED Status | Motor | UART | Transitions |
|-------|------|-------------|-----------|-------|------|------------|
| **0** | STANDBY | "STATUS: READY" | Green ON, Red/Yellow OFF | Stopped | None | → State 1 if RB0=LOW (plug in) |
| **1** | LOCKING_CABLE | "LOCKING PLUG" | Green OFF, Yellow ON | FWD 2s | None | → State 2 after 2s timer |
| **2** | CHARGING | "CHARGING..." + SOC% | Yellow blinking | Stopped | Active (every 500ms) | → State 3 if SOC=100% |
| **3** | CHARGE_COMPLETE | "CHARGE DONE" | Green ON, Yellow OFF | REV 2s | Final stats | → Delay 4s, then State 0 |
| **4** | EMERGENCY_FAULT | "EMERGENCY STOP" | Red ON (blinking) | REV 2s | Error alert | Infinite loop (system halt) |

### FSM Implementation Pattern

```c
// Global state variable
unsigned char charge_state = STATE_STANDBY;

// Main loop
void main(void) {
    Initialize_All();
    
    while(1) {
        Check_Emergency_Stop();  // Highest priority
        
        switch(charge_state) {
            case STATE_STANDBY:
                Handle_Standby();
                break;
            case STATE_LOCKING_CABLE:
                Handle_Locking();
                break;
            case STATE_CHARGING:
                Handle_Charging();
                break;
            case STATE_CHARGE_COMPLETE:
                Handle_Completion();
                break;
            case STATE_EMERGENCY_FAULT:
                Handle_Emergency();
                break;
        }
        
        Update_Display();
        Check_Button_Inputs();
    }
}
```

---

## 📝 Code Overview

### Microcontroller Configuration

```c
// Configuration Pragmas
#pragma config OSC = INTIO67      // Internal 8MHz oscillator (can use external 20MHz)
#pragma config WDT = OFF           // Watchdog timer disabled
#pragma config PWRT = ON           // Power-up timer enabled
#pragma config DEBUG = ON          // In-circuit debugger enabled
#pragma config LVP = ON            // Low-voltage programming enabled
#pragma config XINST = OFF         // Extended instruction set disabled
#pragma config CP0 = OFF           // Code protection off
#pragma config FCMEN = ON          // Fail-safe clock enabled
#pragma config IESO = ON           // Internal/External oscillator switchover
```

### UART Initialization (9600 Baud, 20MHz Clock)

```c
void UART_Init(void) {
    // 20MHz oscillator: Baud rate = FOSC/(16 * (SPBRG + 1))
    // For 9600 baud: SPBRG = (20000000/(16*9600)) - 1 = 129
    
    TXSTAbits.BRGH = 0;     // Low-speed baud rate
    BAUDCONbits.BRG16 = 0;  // 8-bit baud rate generator
    SPBRG = 129;            // Baud rate register value
    
    TXSTAbits.SYNC = 0;     // Asynchronous mode
    TXSTAbits.TX9 = 0;      // 8-bit transmission
    RCSTAbits.RX9 = 0;      // 8-bit reception
    
    TXSTAbits.TXEN = 1;     // Transmit enabled
    RCSTAbits.CREN = 1;     // Continuous receive enabled
    RCSTAbits.SPEN = 1;     // UART enabled
}
```

### Motor Control Functions

```c
// Motor forward (locking cable)
void Motor_Forward(void) {
    LATDbits.LATD0 = 1;     // IN1 = HIGH
    LATDbits.LATD1 = 0;     // IN2 = LOW
}

// Motor reverse (releasing cable)
void Motor_Reverse(void) {
    LATDbits.LATD0 = 0;     // IN1 = LOW
    LATDbits.LATD1 = 1;     // IN2 = HIGH
}

// Motor stop
void Motor_Stop(void) {
    LATDbits.LATD0 = 0;     // IN1 = LOW
    LATDbits.LATD1 = 0;     // IN2 = LOW
}
```

### State Handler Example: Charging State

```c
void Handle_Charging(void) {
    static unsigned int charge_timer = 0;
    static unsigned char soc = 0;           // State of Charge (0-100%)
    static unsigned char toggle_count = 0;
    
    // Increment charge (simulated)
    charge_timer++;
    if(charge_timer > 100) {                // Every ~100 loop cycles
        if(soc < 100) soc++;                // Increment SOC
        charge_timer = 0;
    }
    
    // Blink yellow LED during charging
    toggle_count++;
    if(toggle_count > 50) {
        LATDbits.LATD6 = !LATDbits.LATD6;   // Toggle yellow LED
        toggle_count = 0;
    }
    
    // Broadcast telemetry every 500ms
    static unsigned int uart_timer = 0;
    uart_timer++;
    if(uart_timer > 500) {
        UART_SendTelemetry(soc);            // Voltage, Current, SOC
        uart_timer = 0;
    }
    
    // Transition to complete when charged
    if(soc >= 100) {
        charge_state = STATE_CHARGE_COMPLETE;
        LATDbits.LATD6 = 0;                 // Yellow OFF
        soc = 0;                            // Reset for next charge
    }
}
```

### UART Telemetry Format

```c
void UART_SendTelemetry(unsigned char soc) {
    // Example: "V:5.0 I:32.5 SOC:87%\r\n"
    unsigned char voltage_val = 5;          // Simulated 5V
    unsigned char current_val = 32;         // Simulated 32.5A
    
    UART_SendString("V:");
    UART_SendDecimal(voltage_val);
    UART_SendString(".0 I:");
    UART_SendDecimal(current_val);
    UART_SendString(" SOC:");
    UART_SendDecimal(soc);
    UART_SendString("%\r\n");
}
```

### Button Debouncing (20ms delay)

```c
unsigned char Button_Read(unsigned char port_bit) {
    if(!port_bit) {                         // Button pressed (LOW)
        __delay_ms(20);                     // Debounce delay
        if(!port_bit) return 1;             // Confirmed press
    }
    return 0;                               // Not pressed
}
```

---

## 🎮 How to Use & Test the Project

### Method 1: Interactive Serial in Proteus

1. **Open Proteus Simulation**
   - Load the provided `.pdsprj` file
   - Ensure virtual terminal is visible (right-click device → Display Virtual Terminal)

2. **Click Virtual Terminal Window** to focus input

3. **Test Commands:**
   - Press `S` → Simulate plug insertion (triggers State 1: Locking)
   - Press `F` → Toggle between Fast/Normal charging modes
   - Press `K` → Simulate unplug request (begins unlock sequence)
   - Press `E` → Trigger emergency stop (State 4: Fault)

4. **Monitor Output:**
   - LCD shows current state and SOC%
   - UART terminal displays: `V:5.0 I:32.5 SOC:XX%`
   - LEDs change color based on state
   - Motor runs forward/reverse as expected

### Method 2: Physical Hardware Testing

**Setup:**
- Program PIC18F4580 with compiled HEX file
- Connect circuit on breadboard or PCB
- Attach USB-to-Serial converter to RC6 (TX)

**Test Steps:**
1. **Press RB0** (Plug Switch) → Motor locks forward, LCD shows "LOCKING PLUG"
2. **Observe LCD** → Changes to "CHARGING..." after 2 seconds
3. **Watch Serial Monitor** (9600 baud) → Telemetry updates every 500ms
4. **Press RB1** → Toggles between AC/DC charging modes
5. **Press RB2** → Emergency stop triggers instant motor reverse

### Expected Serial Output

```
V:5.0 I:32.5 SOC:0%
V:5.0 I:32.5 SOC:15%
V:5.0 I:32.5 SOC:30%
V:5.0 I:32.5 SOC:45%
V:5.0 I:32.5 SOC:60%
V:5.0 I:32.5 SOC:75%
V:5.0 I:32.5 SOC:90%
V:5.0 I:32.5 SOC:100%
CHARGE COMPLETE!
```

---

## ⚠️ Safety Considerations

### Critical Safety Features

This project implements several safety mechanisms found in real EV chargers:

1. **Motor-Driven Cable Lock**
   - Prevents accidental disconnection during active charging
   - 2-second lock engagement/release sequences
   - Bi-directional control (forward to lock, reverse to release)

2. **Emergency Stop (E-Stop)**
   - Highest priority interrupt
   - Instantly reverses motor to release cable
   - Locks system in fault state to prevent unintended restart
   - Red LED indication of fault condition

3. **State Machine Validation**
   - Only transitions to next state after completing current state actions
   - Prevents race conditions or erratic behavior
   - All state changes are controlled and predictable

4. **Button Debouncing**
   - 20ms software delay prevents false triggers
   - Necessary for reliable mechanical switch operation
   - Prevents accidental E-Stop activation

### Real-World Considerations Not Implemented

*These are important for production systems but beyond scope of this educational project:*

- **High-Power Safety:** Real chargers handle 350A+ DC; this uses 5V DC motor
- **Insulation Testing:** Production systems test insulation before energizing
- **Temperature Monitoring:** Real chargers monitor cable/connector temperature
- **Arc Detection:** Active systems detect and suppress arcing during disconnect
- **Ground Fault Protection:** Real chargers use GFCI circuits
- **Continuous Health Monitoring:** Firmware watchdog and self-test routines

---

## ✨ Features Summary

| Feature | Implementation | Status |
|---------|----------------|--------|
| **Automatic Cable Locking** | DC motor via L293D | ✅ Functional |
| **Real-Time Telemetry** | UART 9600 baud | ✅ Functional |
| **Dual Charging Modes** | AC/DC toggle button | ✅ Functional |
| **Emergency Stop** | E-Stop button with instant unlock | ✅ Functional |
| **Multi-State FSM** | 5-state finite state machine | ✅ Functional |
| **LED Status Indicators** | Green/Red/Yellow LEDs | ✅ Functional |
| **4-Bit LCD Display** | Real-time status updates | ✅ Functional |
| **Button Debouncing** | Software 20ms delay | ✅ Functional |
| **Motor Speed Control** | Bidirectional control | ⏳ Basic implementation |
| **Power Monitoring** | Simulated (not measured) | ⏳ Simulated |
| **Temperature Monitoring** | Not implemented | ❌ Future enhancement |
| **Arc Detection** | Not implemented | ❌ Real-world only |

---

## 📸 Circuit Diagram

The complete circuit diagram is available in `EVSchematic.png`. Key sections:

- **Power Distribution:** 5V regulated supply with bypass capacitors
- **Microcontroller Section:** PIC18F4580 with crystal oscillator and reset circuit
- **Motor Driver:** L293D H-bridge for bidirectional motor control
- **Input Conditioning:** Pull-up resistors for debouncing
- **Display Interface:** LCD 16×2 in 4-bit mode
- **Serial Communication:** UART TX/RX connections

---

## ✅ Verification & Troubleshooting

### Expected Behavior Checklist

- [ ] At power-up: Green LED ON, LCD shows "STATUS: READY"
- [ ] Press RB0: Motor rotates forward for 2 seconds (locking)
- [ ] After lock: LCD shows "CHARGING..." and yellow LED blinks
- [ ] Serial output: Telemetry displays every 500ms with increasing SOC
- [ ] At SOC=100%: Motor reverses for 2 seconds (releasing cable)
- [ ] After unlock: Green LED ON again, system returns to READY
- [ ] Press RB2 anytime: Immediate motor reverse and system halt

### Troubleshooting Guide

| Issue | Likely Cause | Solution |
|-------|--------------|----------|
| **Motor doesn't run** | L293D not powered or pins not connected | Verify L293D VCC connections; check RD0/RD1 continuity |
| **Motor runs but won't stop** | IN1/IN2 both HIGH or control pins stuck | Verify RD0/RD1 logic; test with multimeter |
| **LCD shows garbage** | Initialization timeout or contrast not adjusted | Verify LCD_RS/EN/D4-D7 connections; adjust contrast pot |
| **Serial data not received** | UART not initialized or baud rate mismatch | Check RC6/TX connection; verify SPBRG value for clock |
| **Buttons don't respond** | Pull-up resistors missing or debounce too long | Add 10kΩ pull-ups; check RB0/RB1/RB2 connections |
| **LEDs don't light** | Resistors too high or pin logic inverted | Use 220Ω resistors; verify RD4/RD5/RD6 logic levels |
| **System won't start charging** | FSM stuck in STATE_STANDBY | Press RB0 to trigger plug detection; check RB0 connection |
| **E-Stop doesn't work** | RB2 not polled or emergency handler not called | Verify emergency check executes before state logic |
| **State doesn't transition** | Timer not incrementing or condition never met | Add debug serial output; check timer logic |
| **Proteus simulation halts** | Infinite loop in state handler | Review state handler logic; add timeout safety checks |

### Debug Serial Output

Add debug print statements to track FSM transitions:

```c
void State_Transition(unsigned char new_state) {
    UART_SendString("STATE: ");
    UART_SendDecimal(charge_state);
    UART_SendString(" -> ");
    UART_SendDecimal(new_state);
    UART_SendString("\r\n");
    charge_state = new_state;
}
```

---

## 📚 Advanced Topics

### Charging Rate Simulation

Current implementation uses fixed "simulated" rates. Real implementation would:

```c
// AC Charging (7.4 kW at 240V, ~30A)
unsigned int ac_current = 30;

// DC Fast Charging (350 kW at 400V, ~875A - not physically safe to simulate)
unsigned int dc_current = 875;

// Charging time = Battery Capacity / Charging Current
// Example: 100 kWh battery at 30A (AC) = 3.3 hours
// Example: 100 kWh battery at 875A (DC) = ~7 minutes
```

### Motor Speed Control (PWM Enhancement)

To control motor speed in a real system:

```c
// Configure PWM on RC2 (CCP1)
void PWM_Init(void) {
    // PWM frequency = FOSC/(4*PR2*(TMR2 Prescale))
    PR2 = 249;                  // For 20kHz PWM
    T2CON = 0x04;              // Timer2 on, 1:1 prescale
    CCP1CON = 0x0C;            // PWM mode
}

void Motor_Speed(unsigned char duty) {
    // duty: 0-255, where 255 = 100%
    CCPR1L = duty;              // Set PWM duty cycle
}
```

### Battery Simulation

Real system would monitor actual battery through ADC:

```c
unsigned int Battery_Voltage(void) {
    unsigned char adc_result = ADC_Read();
    // Convert: 0-1023 (10-bit) = 0-48V
    // Voltage = (adc_result * 48) / 1023
    return (adc_result * 48) / 1023;
}

unsigned int Battery_Current(void) {
    unsigned char adc_result = ADC_Read_Channel(AN1);
    // Current sensor: ACS712 (±5A range)
    // Voltage = (adc_result * 5) / 1023
    // Current = (Voltage - 2.5V) / 0.185V per A
    return ((adc_result * 5) / 1023 - 2500) / 185;
}
```

### State Machine Timing Diagram

```
Event Timeline:
0ms:   User presses RB0 (plug inserted)
       ├─ STATE_STANDBY → STATE_LOCKING_CABLE
       └─ Motor_Forward() starts
       
2000ms: Motor stops after 2 seconds
        ├─ STATE_LOCKING_CABLE → STATE_CHARGING
        └─ Yellow LED starts blinking
        
500ms:  UART telemetry broadcast (repeats every 500ms)
        └─ "V:5.0 I:32.5 SOC:5%"
        
1000ms: SOC increments 5%
        ├─ UART: "V:5.0 I:32.5 SOC:10%"
        
... (repeats until SOC reaches 100%)

20000ms: SOC reaches 100%
         ├─ STATE_CHARGING → STATE_CHARGE_COMPLETE
         └─ Motor_Reverse() starts
         
22000ms: Motor stops after 2 seconds (cable released)
         ├─ STATE_CHARGE_COMPLETE → (4s delay)
         └─ Yellow LED OFF, Green LED ON
         
26000ms: After 4-second delay
         └─ STATE_* → STATE_STANDBY
         
E-Stop Event (can happen anytime):
Tx ms:  User presses RB2
        ├─ Check_Emergency_Stop() executes
        ├─ ANY_STATE → STATE_EMERGENCY_FAULT
        ├─ Motor_Reverse() (force unlock)
        └─ Red LED ON (fault indicator)
```

---

## 🔗 Related Projects in This Series

- [INTERUPT - External Interrupt Control](https://github.com/srxcr/INTERUPT) – Foundation for event-driven programming
- [ADC - Analog Input Conversion](https://github.com/srxcr/ADC) – Battery voltage/current sensing
- [USART-LCD - Serial & Display](https://github.com/srxcr/USART-LCD) – Telemetry and user interface
- [Dual-Motor-Control - Motor PWM](https://github.com/srxcr/Dual-Motor-Control-with-LCD) – Advanced motor control
- [Crane-Collision-Safety-System](https://github.com/srxcr/Crain_Collition_Saftey_Sys) – Safety-critical FSM design

---

## 🎯 Learning Outcomes

After completing this project, you will understand:

1. **Finite State Machines** – Event-driven architecture for complex systems
2. **Motor Control** – H-bridge drivers and directional control
3. **Real-Time Systems** – Timing, synchronization, and state transitions
4. **Safety Design** – Emergency stops, fail-safes, and critical sections
5. **Serial Communication** – UART telemetry and real-time data transmission
6. **Multi-Module Integration** – Combining GPIO, LCD, UART, and timing
7. **Embedded System Architecture** – Designing scalable, maintainable code
8. **Hardware Debugging** – Testing and verifying complex systems

---

## 📄 License

This project is intended for **educational and learning purposes**. Users are free to modify and enhance the project for academic and personal use. No commercial use without proper attribution.

---

## 🙏 Acknowledgements

Developed and tested using:

* **PIC18F4580 Microcontroller** – Microchip Technology
* **MPLAB X IDE** – Microchip Technology
* **XC8 Compiler** – Microchip Technology
* **L293D H-Bridge Motor Driver** – Texas Instruments
* **16×2 LCD Module** – Character display standard
* **Proteus Design Suite** – Labcenter Electronics
* **Real-World EV Charging References** – SAE J1772, IEC 62196 standards

---

## 📞 Support & Further Learning

For questions or improvements:
- Review the FSM state diagrams and timing analysis
- Consult Microchip PIC18F4580 datasheet (UART, GPIO sections)
- Study real EV charging standards (SAE J1772 for NA, IEC 62196 for EU)
- Experiment with different charging profiles and safety scenarios
- Extend the project with temperature monitoring or power calculations

**This project bridges educational learning and real-world embedded systems design. Have fun exploring! ⚡🚗**
