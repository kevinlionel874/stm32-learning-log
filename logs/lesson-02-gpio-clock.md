# Lesson 02: GPIO internals and a 72 MHz system clock

**Date:** 2026-10-22 · **Code:** [`blink/`](../blink/) · **Video:** [SOS on the Blue Pill](../media/lesson-02-sos.mp4)

## Goal

Understand what the GPIO pin is actually doing electrically, run the chip at its full 72 MHz from the external crystal, and write my own small LED functions.

## What I did

1. **Worked out why the on-board LED is active-low.** On the Blue Pill the LED circuit is 3.3V → resistor → LED → PC13. Current only flows when there is a voltage difference, so the LED lights when PC13 outputs 0 and turns off when it outputs 1.
2. **Configured PC13 properly in CubeMX:**
   - User label `LED`, so CubeMX generates `LED_Pin` and `LED_GPIO_Port` and the code reads like what it does
   - Initial output level **High**, so the LED is off at power-up
   - Output speed Low, since an LED doesn't need fast edges
3. **Switched the system clock from the internal 8 MHz RC oscillator (HSI) to the external 8 MHz crystal (HSE) and the PLL:**

   | Clock | Setting | Frequency |
   |---|---|---|
   | HSE crystal → PLL | ×9 | 72 MHz |
   | SYSCLK / HCLK | PLLCLK, AHB /1 | 72 MHz |
   | APB1 (max 36 MHz) | /2 | 36 MHz |
   | APB2 | /1 | 72 MHz |

   I checked the regenerated `SystemClock_Config()`: it now uses `RCC_OSCILLATORTYPE_HSE` and `RCC_PLL_MUL9`. My own code in the `USER CODE` sections survived regeneration.
4. **Wrote helper functions** so the active-low detail lives in one place:
   ```c
   void LED_On(void)  { HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_RESET); } // 0 = on
   void LED_Off(void) { HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET); }   // 1 = off
   void LED_Blink(uint32_t on_ms, uint32_t off_ms) { LED_On(); HAL_Delay(on_ms); LED_Off(); HAL_Delay(off_ms); }
   ```
5. **Blinked SOS in Morse code** with a 150 ms time unit: dot = 1 unit, dash = 3 units, gap between letters = 3 units, gap between words = 7 units.
   ```c
   for (int i = 0; i < 3; i++) LED_Blink(150, 150);  // S
   HAL_Delay(300);
   for (int i = 0; i < 3; i++) LED_Blink(450, 150);  // O
   HAL_Delay(300);
   for (int i = 0; i < 3; i++) LED_Blink(150, 150);  // S
   HAL_Delay(900);
   ```
6. **Checked the timing against a video.** I filmed the board at 30 frames per second and measured how many frames the LED stayed on and off:

   | Element | Expected | Measured |
   |---|---|---|
   | Dot (on) | 150 ms | 133–150 ms |
   | Dash (on) | 450 ms | 433–500 ms |
   | Gap between letters | 450 ms | ~467 ms |

   Each frame is 33 ms, so the measurements match within one frame.
7. **Experiment: push-pull vs open-drain.** I switched PC13 to open-drain and the SOS still worked exactly the same. A push-pull output can drive the pin both high and low. An open-drain output can only pull it low, and "high" just means letting go. This LED only needs the pin to pull low (its other end is already at 3.3V), so both modes behave the same here. If the LED were wired from the pin to GND instead, open-drain could never light it without a pull-up resistor. Open-drain matters for shared buses like I2C, where several devices pull the same line low and a pull-up resistor makes it high. The project is still in open-drain mode from this experiment. I'll switch it back to push-pull, the normal choice, before the next lesson.

## Problems I ran into and how I fixed them

| Problem | Cause | Fix / lesson |
|---|---|---|
| "No ST-LINK detected" when I only meant to compile | I clicked Run (build + flash) while the ST-Link was unplugged | The hammer only builds; Run also flashes and needs the programmer connected |
| The SOS looked like "three fast blinks" every time | A 450 ms dash is hard to tell apart from a 150 ms dot by eye | The video measurement showed the timing was correct. Doubling the unit makes the dashes obvious |
| Later it looked like "fast 3, fast 3, slow 3" | The pattern repeats forever, so it depends on where you start watching | The longest pause marks the start of each SOS. That's exactly why Morse code uses a longer gap between words than between letters |

## What I learned

- How an LED's wiring decides whether 0 or 1 turns it on
- The difference between push-pull and open-drain outputs, and where open-drain is needed (I2C)
- How the STM32 clock tree works: crystal → PLL → system clock → bus prescalers, and why APB1 has to be divided down
- Writing small functions to hide hardware details and make the main loop readable
- Verifying timing with an actual measurement instead of trusting my eyes

## Next

Lesson 03: non-blocking timing with `HAL_GetTick()`, so the program can do several things at once instead of sitting inside `HAL_Delay`.
