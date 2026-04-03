AI-Supervised Adaptive Thermal Control in a Forced-Convection Enclosure

An embedded closed-loop thermal control system built on an **ESP32** for regulating enclosure temperature using a **12 V DC fan**, **dual SHT31 temperature/humidity sensors**, a **discrete PI controller**, and an **adaptive fuzzy supervisory layer**. The system also includes **Nextion HMI display support**, **RTC timekeeping**, **tachometer-based fan monitoring**, **CSV serial logging**, and **Blynk IoT remote supervision/control**.

This project was developed as a final-year Electrical and Electronic Engineering instrumentation and control project, with emphasis on practical embedded implementation, measurable verification, and structured engineering evaluation, consistent with the project planning guidance and assessment expectations. :contentReference[oaicite:0]{index=0} :contentReference[oaicite:1]{index=1}

---

## Project Overview

Thermal systems with forced convection are slow, nonlinear, and affected by changing operating conditions such as airflow, ambient temperature, and enclosure disturbances. A fixed-gain controller can regulate temperature, but its performance is often limited by the trade-off between responsiveness and robustness.

To address this, the project implements:

- a **baseline SIMC/IMC-inspired PI controller**
- multiple fixed operating modes
- an **adaptive Sugeno fuzzy supervisor** that schedules controller aggressiveness online
- embedded fault handling and safety logic
- local and remote monitoring interfaces

The objective is to maintain stable temperature regulation while improving transient response, disturbance rejection, and practical deployability on a low-cost microcontroller platform.

---

## Key Features

- **ESP32-based real-time thermal control**
- **Dual SHT31 sensors** with averaged temperature measurement
- **Humidity measurement** integrated into the supervisory logic
- **Discrete PI controller** with bumpless mode switching
- **Four operating modes**
  - Conservative
  - Normal
  - Aggressive
  - Adaptive
- **Sugeno fuzzy supervisor** for online lambda scheduling
- **RLS-based online plant estimation**
- **Anti-windup protection**
- **Slew-rate limiting**
- **Fan minimum-duty and kick-start logic**
- **Tachometer-based fan fault detection**
- **Sensor fault fallback mode**
- **Authority exceeded detection**
- **Nextion HMI integration**
- **Blynk IoT remote monitoring and control**
- **RTC support (DS3231)**
- **CSV serial logging for analysis and validation**

---

## Control Strategy

### 1. Baseline PI Control
The core controller is a discrete PI regulator designed for thermal control of a forced-convection enclosure.

### 2. Fixed Control Modes
The firmware supports three fixed gain sets:

| Mode | Lambda | Kp | Ki |
|------|--------|----|----|
| Conservative | 40 | 11.63 | 0.2907 |
| Normal | 30 | 15.15 | 0.3788 |
| Aggressive | 20 | 21.74 | 0.5435 |

### 3. Adaptive Mode
In adaptive mode, a **Sugeno fuzzy supervisor** adjusts the effective tuning aggressiveness by modifying the closed-loop parameter **lambda** within:

- **Lambda range:** 20 s to 40 s

Inputs used by the supervisor include:

- control error
- filtered error derivative
- humidity

This allows the controller to become more aggressive during large disturbances and more conservative near steady state.

### 4. Online Identification
The system also includes a lightweight **Recursive Least Squares (RLS)** estimator to track plant behaviour online and support adaptive gain correction.

---

## Hardware Used

- **ESP32 development board**
- **2 × Sensirion SHT31 sensors**
  - I2C addresses: `0x44` and `0x45`
- **12 V DC fan**
- **Fan tachometer output**
- **Fan driver / power stage**
- **Nextion display**
- **DS3231 RTC module**
- **Wi-Fi connection for Blynk IoT**

---

## Pin Configuration

| Function | Pin |
|---------|-----|
| I2C SDA | GPIO 21 |
| I2C SCL | GPIO 22 |
| Fan PWM | GPIO 33 |
| Fan Tachometer | GPIO 34 |
| Nextion RX/TX | GPIO 16 / 17 |

---

## PWM and Timing Settings

| Parameter | Value |
|----------|-------|
| PWM frequency | 25 kHz |
| PWM resolution | 10-bit |
| Sample time | 1 s |
| Control filter time constant | 3 s |
| Logging filter time constant | 10 s |

---

## Safety and Reliability Features

This system was designed with practical embedded safety mechanisms to improve robustness during real operation.

### Implemented safeguards
- sensor failure detection
- safe fallback duty cycle on sensor fault
- fan stall / tachometer fault detection
- minimum duty enforcement
- kick-start support for fan startup
- anti-windup logic
- slew-rate limiting
- saturation / authority monitoring
- bumpless transfer during mode switching

### Fault fallback behaviour
- If sensor readings fail repeatedly, the controller enters a safe mode with a predefined fallback duty.
- If the fan is commanded on but tachometer feedback remains too low, the firmware attempts recovery and eventually declares a fan fault.

---

## Human-Machine Interface

### Nextion HMI
The local display provides:
- measured temperature
- setpoint
- fan duty
- humidity
- active mode
- controller gains
- alarm/status indication
- RTC time display

### Blynk IoT Dashboard
Remote supervision includes:
- temperature
- setpoint
- humidity
- fan duty
- active mode
- error
- anti-windup factor
- disturbance flag
- cooling gain

---

## Blynk Virtual Pin Map

| Virtual Pin | Function |
|------------|----------|
| V0 | Temperature |
| V1 | Setpoint |
| V2 | Humidity |
| V3 | Fan duty |
| V4 | Active mode |
| V5 | Requested mode |
| V6 | Error |
| V7 | Anti-windup factor |
| V8 | Thermal disturbance flag |
| V9 | Cooling gain |

---

## Serial Commands

The firmware supports simple serial commands for debugging and manual operation.

### Set setpoint
```text
SP=19.0
