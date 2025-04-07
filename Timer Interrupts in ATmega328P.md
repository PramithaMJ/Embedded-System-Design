# Timer Interrupts in ATmega328P (CTC Mode with Timer1)

## Table of Contents
- [Overview](#overview)
- [Clock and Prescaler](#clock-and-prescaler)
- [Timer Modes](#timer-modes)
- [CTC Mode Configuration](#ctc-mode-configuration)
- [Register Breakdown](#register-breakdown)
- [Complete Code Explanation](#complete-code-explanation)
- [Expected Behavior](#expected-behavior)
- [Visual Diagram](#visual-diagram)
- [Useful Notes](#useful-notes)

---

## Overview
This guide explains how to use **Timer1** on the **ATmega328P** microcontroller to generate an **interrupt every 1 second** using **CTC (Clear Timer on Compare Match) mode**.

We use:
- Timer1 (16-bit)
- Clock frequency = 16 MHz
- Prescaler = 256
- Compare Match Register A (OCR1A)

---

## Clock and Prescaler

The clock of the ATmega328P runs at **16 MHz**.
- 1 clock tick = \( \frac{1}{16000000} \) seconds = 62.5 ns
- Timer1 is a 16-bit timer: max value = \(2^{16} = 65536\)

To slow down the timer count, we use a **prescaler**. Common values:

| Prescaler | Formula                                | Resulting Max Time         |
|-----------|-----------------------------------------|----------------------------|
| 1         | \(\frac{1}{16000000} \times 65536\)     | 4.096 ms                  |
| 8         | \(\frac{8}{16000000} \times 65536\)     | 32.768 ms                 |
| 64        | \(\frac{64}{16000000} \times 65536\)    | 262.144 ms                |
| 256       | \(\frac{256}{16000000} \times 65536\)   | **2.097 s**               |
| 1024      | \(\frac{1024}{16000000} \times 65536\)  | 16.777 s                  |


To generate a 1-second delay:
- Tick duration with prescaler 256: \(\frac{256}{16000000} = 16 \mu s\)
- Needed ticks: \(\frac{1}{16 \mu s} = 62500\)

So, we set OCR1A = 62500 (0xF424)

---

## Timer Modes

We use **CTC (Clear Timer on Compare Match)** mode:
- Timer counts from 0 to OCR1A
- When `TCNT1 == OCR1A`, it triggers an interrupt and resets to 0

---

## CTC Mode Configuration

To set CTC mode:
- WGM13 = 0
- WGM12 = 1
- WGM11 = 0
- WGM10 = 0

Registers involved:
- `TCCR1A &= 0b11111100;` → WGM11, WGM10 = 0
- `TCCR1B |= 0b00001000;` → WGM12 = 1
- `TCCR1B &= 0b11101111;` → WGM13 = 0

Prescaler (CS12:CS10): 256
- `TCCR1B |= 0b00000100;`
- `TCCR1B &= 0b11111100;`

---

## Register Breakdown

```c
TIMSK1 |= 0b00000010; // Enable Timer1 Compare Match A interrupt
TCCR1A &= 0b11111100; // Clear WGM11, WGM10
TCCR1B |= 0b00001000; // Set WGM12
TCCR1B &= 0b11101111; // Clear WGM13
TCCR1B |= 0b00000100; // Set CS12 (Prescaler = 256)
TCCR1B &= 0b11111100; // Clear CS11 and CS10
OCR1AH = 0xF4;         // High byte of 62500
OCR1AL = 0x24;         // Low byte of 62500
TCNT1H = 0x00;         // Reset timer counter
TCNT1L = 0x00;
sei();                 // Enable global interrupts
```

---

## Complete Code Explanation

```c
#define F_CPU 16000000UL
#include <avr/io.h>
#include <avr/interrupt.h>

volatile int toggle = 0;

int main(void)
{
    TIMSK1 |= 0b00000010; // Enable Timer1 Compare Match A Interrupt

    // Set CTC Mode (WGM12 = 1)
    TCCR1A &= 0b11111100;
    TCCR1B |= 0b00001000;
    TCCR1B &= 0b11101111;

    // Prescaler = 256
    TCCR1B |= 0b00000100;
    TCCR1B &= 0b11111100;

    // Set OCR1A for 1 second interval
    OCR1AH = 0xF4;
    OCR1AL = 0x24;

    // Reset counter
    TCNT1H = 0x00;
    TCNT1L = 0x00;

    sei(); // Enable global interrupts

    // Set PB5 as output (LED), PB0 as input (button or unused)
    DDRB = 0b11111110;

    while (1)
    {
        // Main loop does nothing, waiting for interrupt
    }
}

ISR(TIMER1_COMPA_vect)
{
    if (toggle)
    {
        PORTB |= 0b00100000; // Set PB5 High (LED ON)
    }
    else
    {
        PORTB &= 0b11011111; // Set PB5 Low (LED OFF)
    }
    toggle = 1 - toggle; // Flip between 0 and 1
}
```

---

## Expected Behavior

- LED connected to **Pin PB5** will **toggle ON/OFF every 1 second**.
- Timer1 runs in the background and fires an interrupt every 62500 ticks.
- The ISR (interrupt service routine) handles toggling the LED.

---

## Visual Diagram

```
| Timer1 Count (TCNT1) |
|----------------------|
|     0 → 62500        |
         ^
         |
    OCR1A Match → ISR (Toggle LED)
         ↓
       Reset to 0
```

---

## Useful Notes

- `volatile` keyword is **very important** for shared variables between ISR and `main()`.
- Using hardware timers is **much more efficient** than software delays.
- This works even if the `main()` is busy or sleeping — hardware interrupts are reliable.
- Be sure to connect the LED properly with a **current-limiting resistor**.

---

## Common Mistakes

| Mistake                         | Fix                                                   |
|----------------------------------|--------------------------------------------------------|
| Forgetting `sei()`              | Global interrupts won’t enable                        |
| Wrong OCR1A value               | Recalculate based on desired time & prescaler         |
| Wrong DDRB setting              | Pin won’t output if not set to output mode            |
| Incorrect register bit masking  | Use `&=` and `|=` correctly to set/clear bits safely  |

---

## Conclusion

Timer interrupts are a powerful tool in embedded systems. They allow precise, timed actions without blocking code. In this tutorial, we used **Timer1 CTC mode** with a **prescaler of 256** and a **compare match value of 62500** to toggle an LED every 1 second.

## Datasheet
https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf
