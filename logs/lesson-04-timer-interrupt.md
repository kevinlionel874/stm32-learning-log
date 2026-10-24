# Lesson 04: Timer interrupts with TIM2

**Date:** 2026-10-24 · **Code:** [`blink/`](../blink/) · **Screenshot:** [counters read in the debugger](../media/lesson-04-debugger.webp)

## Goal

Let a hardware timer keep time and interrupt the CPU when time is up, so the timing no longer depends on how fast the main loop runs. In lesson 3 the main loop had to keep checking the clock itself. If the loop got stuck, every task got late.

## What I did

1. **Configured TIM2 in CubeMX:** internal clock, prescaler `7200-1`, counter period `5000-1`, and the TIM2 global interrupt enabled in the NVIC settings.

   | Stage | Value | Why |
   |---|---|---|
   | Timer clock | 72 MHz | TIM2 is on APB1, which runs at 36 MHz. When the APB1 prescaler isn't 1, the timer clock is doubled, so it's 72 MHz again |
   | ÷ prescaler (7200) | 10 kHz | One count every 0.1 ms |
   | × period (5000 counts) | 0.5 s | The counter goes 0 → 4999, then wraps to 0 and raises an update event |

   Both values are written as "minus 1" because the hardware counts from 0: 0 to 4999 is 5000 counts.

   After regenerating the code I checked that CubeMX hadn't reset the internal-builder setting from lesson 3. It hadn't.
2. **Started the timer and wrote the interrupt callback.** CubeMX only configures the timer. `HAL_TIM_Base_Start_IT(&htim2)` actually starts it with its interrupt enabled. Then I overrode HAL's `__weak` callback with my own function of the same name:
   ```c
   void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
   {
     if (htim->Instance == TIM2)
     {
       HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
       tim2_count++;
     }
   }
   ```
   What happens on every update event:
   ```
   TIM2 wraps from 4999 to 0 → update flag set → NVIC interrupts the CPU
   → CPU saves where it was in main() and jumps to TIM2_IRQHandler()   (stm32f1xx_it.c, generated)
   → HAL_TIM_IRQHandler(&htim2) checks the flag and clears it          (otherwise the interrupt would fire again forever)
   → HAL_TIM_PeriodElapsedCallback()                                    (my code)
   → CPU returns to exactly where main() was interrupted
   ```
   I commented out the SOS task from lesson 3 so that only the interrupt controls the LED. The main loop no longer contains any LED code.
3. **Experiment: blocked the main loop on purpose.** I added `HAL_Delay(3000)` to the main loop, so each pass takes 3 seconds. The LED still toggled exactly every 0.5 s, because the timing is done by the timer hardware, not by the loop.
4. **Measured it with the debugger.** I counted interrupts in a `volatile` variable (`volatile` because it's changed inside an interrupt, so the compiler must always read it from memory). I ran the program for about half a minute, paused it, and read four variables:

   | Variable | Value | Meaning |
   |---|---|---|
   | `uwTick` | 33 261 | HAL's millisecond counter: 33.3 s really passed |
   | `tim2_count` | 67 | TIM2 interrupts. 33.3 s ÷ 0.5 s ≈ 66.5, so none were missed |
   | `loop_count` | 12 | Main loop passes. At 3 s per pass, 33 s only allows 12 passes |
   | `seconds` | 11 | The lesson 3 polling task thought only 11 s had passed |

   The polling task can only be as accurate as the loop is fast. Each loop pass can add at most one second, so it fell behind by two thirds. The two interrupt-driven counters (`uwTick` and `tim2_count`) agree with each other.

   `tim2_count` is one higher than 66.5. My guess is that it comes from pausing in the debugger. When the core is halted, SysTick (which drives `uwTick`) stops too, but on the STM32F1 a timer like TIM2 keeps counting unless its freeze bit in the DBGMCU register is set. I haven't confirmed this yet.
5. **Realized I had been using interrupts since lesson 1.** `HAL_Delay()` and `HAL_GetTick()` work because the SysTick timer interrupts the CPU every millisecond and increments `uwTick`. `HAL_Init()` sets this up.

## Problems I ran into and how I fixed them

| Problem | Cause | Fix / lesson |
|---|---|---|
| The TIM2 parameter table wasn't visible in CubeMX, only the help text for the prescaler | The CubeMX window was too small, and the Mode panel took most of the space | Maximized the window and dragged the Configuration divider up |
| Debug As showed "(none applicable)" | `main.c` was selected in the Project Explorer instead of the project | Select the project first, or use Run → Debug History |
| The Run menu had no Suspend item | No debug session was running yet. Resume and Suspend only appear during debugging | Start the debug session first |

## What I learned

- What an interrupt is: hardware stops the CPU, runs a short function, and returns to exactly where it was
- The chain from the timer hardware through the NVIC, the IRQ handler and HAL to my callback
- Prescaler and auto-reload math, and why the APB1 timer clock is doubled
- How HAL's `__weak` callbacks let me plug in my own code
- Why variables shared with an interrupt need `volatile`
- Polling vs. interrupts: a polled task fails when the loop is blocked, an interrupt doesn't
- Interrupt callbacks must be short and must never call `HAL_Delay()`

## Next

Lesson 05: debugging properly with breakpoints, stepping and watching variables.
