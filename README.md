# STM32 Learning Log

I'm a first-year student at the University of Illinois Urbana-Champaign. I plan to study Electrical Engineering, and I'm most interested in semiconductors. This repository records how I'm teaching myself embedded systems on an STM32 microcontroller: every lesson, the problems I ran into, and how I solved them.

## Goal

Start from blinking one LED and build up to a project that connects embedded systems with semiconductor physics: an **LED / diode I-V curve tracer**. It will sweep a voltage, measure current with the STM32's ADC, and plot the curves, so I can compare the turn-on voltages of different LED colors with their band gaps (planned for Spring 2027).

## Hardware

| Part | What I use it for |
|---|---|
| STM32F103C8T6 "Blue Pill" board (×2) | The microcontroller I program |
| ST-Link V2 programmer | Flashing and debugging over SWD |
| CP2102 USB-to-serial adapter | UART communication with my laptop |
| ELEGOO electronics kit (breadboard, LEDs, resistors, buttons, capacitors, diodes, transistors) | Building circuits |
| ELEGOO 37-sensor kit | Rotary encoder, buzzer, relay, line tracker, thermistor, ultrasonic sensor, etc. |
| AHT20 sensor, 1.3" I2C OLED (SH1106) | Temperature/humidity and a display |

## Toolchain

- **STM32CubeMX**: pin and clock configuration, code generation
- **STM32CubeIDE 2.2.0**: editing, building (internal builder, see [lesson 3](logs/lesson-03-gettick.md)), flashing, debugging
- **STM32 HAL library** (STM32Cube FW_F1 V1.8.7)

## Progress

| # | Date | Topic | Status | Notes | Code |
|---|---|---|---|---|---|
| 1 | 2026-10-20 | Toolchain setup, SWD wiring, first blinking LED | ✅ Done | [log](logs/lesson-01-blink.md) | [`blink/`](blink/) |
| 2 | 2026-10-22 | GPIO internals (active-low LED, push-pull vs open-drain), 72 MHz clock, SOS in Morse code | ✅ Done | [log](logs/lesson-02-gpio-clock.md) · [video](media/lesson-02-sos.mp4) | [`blink/`](blink/) |
| 3 | 2026-10-23 | Non-blocking timing with `HAL_GetTick`, two tasks at once (~910 000 loop passes/s), SOS as a state machine | ✅ Done | [log](logs/lesson-03-gettick.md) · [video](media/lesson-03-sos-nonblocking.mp4) · [photo](media/lesson-03-board.jpg) | [`blink/`](blink/) |
| 4 | | Timer interrupts | 🔄 Next | | |
| 5 | | Debugging with breakpoints and watch variables | Planned | | |
| 6 | | Breadboard: external LEDs and buttons | Planned | | |
| 7–9 | | External interrupts, UART | Planned | | |
| 10–13 | | PWM: breathing LED, buzzer music, servo, rotary encoder | Planned | | |
| 14 | | ADC | Planned | | |
| 15–20 | | I2C OLED, AHT20 sensor, ultrasonic ranging, DMA data logging | Planned | | |
| Project | Winter 2026 | Desk environment station | Planned | | |
| Project | Spring 2027 | LED / diode I-V curve tracer | Planned | | |

## How I learn

Each lesson I do hands-on on my own hardware: wiring, configuring the chip in CubeMX, writing and flashing the code, and testing it. I use an AI tutor (Claude) for explanations and troubleshooting. The wiring, configuration, debugging and experiments recorded here are my own work. Every lesson gets a log in [`logs/`](logs/) with what I did, what went wrong, and what I learned.
