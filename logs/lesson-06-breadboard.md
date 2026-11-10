# Lesson 06: Header pins and the breadboard

**Status:** 🔄 In progress · **Video:** [soldering the header pins](../media/lesson-06-soldering.mp4)

## Part 1: Soldering the header pins (November 2026)

Until now the Blue Pill only had its 4-pin SWD header, so the only thing I could use was the on-board LED on PC13. To put the board on a breadboard and connect my own LEDs, buttons and sensors, I needed header pins on both long edges (2 × 20 pins).

I soldered them at the **ECE OpenLab**, with help from someone at the lab. The [video](../media/lesson-06-soldering.mp4) shows the setup:

- The Blue Pill held in a "helping hands" stand with alligator clips, so both hands are free
- A temperature-controlled soldering iron
- Flux from a syringe, which helps the solder flow onto the pad and the pin
- A fume extractor behind the board

Each joint has to join the pin to the copper ring around the hole. A good joint is shiny and cone-shaped. A ball of solder that doesn't wet the pad, or solder bridging two neighboring pins, has to be reheated and fixed. After soldering, the pins point down so the board plugs straight into a breadboard, with the chip and the pin labels facing up.

## Part 2: Breadboard (next)

External LED with a current-limiting resistor, and a button with a pull-up resistor and debouncing.
