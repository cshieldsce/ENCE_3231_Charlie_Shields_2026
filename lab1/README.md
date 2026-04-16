# ENCE 3231: Lab 1: Ultrasonic Sensor with Timers

**Author:** Charlie Shields
**Course:** ENCE 3231: Embedded Systems
**Year:** 2026

## Building / Importing

This is an STM32CubeIDE project (`Week3_Class1_Timer1_Pulse/`). Build artifacts (`Debug/`, `*.elf`, `*.o`, …) are gitignored and regenerated on import.

1. Clone the repo.
2. In STM32CubeIDE: **File → Import → Existing Projects into Workspace**, point at `lab1/Week3_Class1_Timer1_Pulse/`.
3. Right-click the project → **Build Project** (regenerates `Debug/`).
4. **Run → Debug As → STM32 C/C++ Application** to flash + open the live-expression view.

Target MCU: **STM32F407VG**. If you re-generate code from the `.ioc`, leave the timer settings below unchanged.

## Goal

Interface an HC-SR04 ultrasonic sensor to an STM32 using **only timer functionalities** (no delay loops, no blocking code). The trigger pulse, measurement cycle, and echo capture are all driven by hardware timers and their interrupts. The measured range counter and the computed distance (cm) are observed through the debug window / live expression visualizer.

## Block Diagram

![Block diagram](output-FC782C85-31C1-407C-BADF-A170DB190378.jpeg)

## Timer Configuration

### TIM1 — 60 ms Measurement-Cycle Tick

| Field         | Value      | Reason                                        |
|---------------|------------|-----------------------------------------------|
| Prescaler     | `84 - 1`   | 84 MHz / 84 = 1 MHz (1 µs tick)               |
| Counter Period| `65536 - 1`| ~65.5 ms update event → gates each ping ≥60 ms|
| Mode          | Interrupt  | `HAL_TIM_PeriodElapsedCallback` retriggers TIM2 |

### TIM2 — 10 µs TRIG Pulse (PWM, one-shot-style)

| Field         | Value        | Reason                                     |
|---------------|--------------|--------------------------------------------|
| Prescaler     | `84 - 1`     | 1 MHz counter                              |
| Counter Period| `100000 - 1` | 100 ms frame (irrelevant once one-shot)    |
| Channel       | CH1 PWM out  | Drives HC-SR04 `TRIG` pin                  |
| Pulse (CCR1)  | `10`         | 10 counts × 1 µs = **10 µs high time**     |


### Scope Capture — 10 µs TRIG Pulse

Measured on the TIM2 CH1 output pin (PA5).

![Scope trace showing 10 µs pulse](image.png)

## What Works

- TIM1 periodic interrupt fires reliably at ~60 ms.
- TIM1 callback kicks off TIM2 PWM, producing a clean 10 µs pulse on TRIG (scope-verified).

## What Doesn't Work Yet

- Echo capture is not implemented.
- Because echo capture is missing, no distance value is computed.
