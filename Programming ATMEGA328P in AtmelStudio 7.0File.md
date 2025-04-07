# Programming ATMEGA328P in Atmel Studio 7.0

Welcome to the ultimate beginner-friendly guide to programming the **ATmega328P** microcontroller using **Atmel Studio 7.0**. This guide walks you through all the key concepts, with real code examples, explanations, and simulations. Perfect for anyone starting out in **embedded systems development**.

---

## Table of Contents

1. [ATmega328P Pinout](#atmega328p-pinout)
2. [Setting a Pin as Input / Output](#setting-a-pin-as-input--output)
3. [LED Blink Program](#led-blink-program)
4. [Make the LED Respond to a Push Button](#make-the-led-respond-to-a-push-button)
5. [Using External Interrupt to Toggle LED](#using-external-interrupt-to-toggle-led)
6. [Understanding the Status Register (SREG)](#understanding-the-status-register-sreg)
7. [Datasheet Navigation Guide](#datasheet-navigation-guide)

---

## ATmega328P Pinout

The ATmega328P has 28 pins. Understanding the **pinout** helps in configuring I/O operations properly.

- **PORTB (Digital Pins 8–13)**
  - PB0 to PB5: General purpose I/O pins
  - Example: **PB5 corresponds to Digital Pin 13** on Arduino (used for blinking LED)

- **PORTC (Analog Pins)**
- **PORTD (Digital Pins 0–7)**

Refer to the ATmega328P datasheet for the full pin diagram.

---

## Setting a Pin as Input / Output

To configure a pin:

- `DDRx` (Data Direction Register):
  - `1` = Output
  - `0` = Input
- `PORTx` (Output Register)
- `PINx` (Input Register)

### Examples
```c
// Set PORTB pin 0 as input
DDRB &= 0b11111110;

// Set PORTB pin 5 as output
DDRB |= 0b00100000;

// Read input
if (PINB & 0b00000001) { /* do something */ }
```

---

## LED Blink Program

### **File**: `LED_Blink.c`
```c
#define F_CPU 16000000UL
#include <avr/io.h>
#include <util/delay.h>

int main(void) {
    DDRB |= 0b00100000; // Set PB5 as output (Digital Pin 13)

    while (1) {
        PORTB |= 0b00100000; // Turn LED ON
        _delay_ms(500);

        PORTB &= 0b11011111; // Turn LED OFF
        _delay_ms(500);
    }
}
```

### Explanation
- PB5 (Digital Pin 13) is toggled ON/OFF every 500ms.
- `DDRB` controls the direction of pins.
- `PORTB` sends the signal to the pin.

---

## Make the LED Respond to a Push Button

### **Circuit**:
- LED on PB5
- Button on PB0 (with pull-down resistor)

### **Code**:
```c
#define F_CPU 16000000UL
#include <avr/io.h>

int main(void) {
    DDRB |= 0b00100000; // Set PB5 as output
    DDRB &= 0b11111110; // Set PB0 as input

    while(1) {
        if (PINB & 0b00000001) {
            PORTB |= 0b00100000; // LED ON
        } else {
            PORTB &= 0b11011111; // LED OFF
        }
    }
}
```

---

## Using External Interrupt to Toggle LED

### **Objective**
Toggle LED on PB5 when a button (connected to INT1 - PD3) is pressed.

### **Concepts Used**
- External Interrupts: INT0 and INT1
- `EIMSK` – Enable external interrupts
- `EICRA` – Control interrupt sense (rising/falling)
- `ISR()` – Interrupt Service Routine

### **Code**:
```c
#define F_CPU 16000000UL
#include <avr/io.h>
#include <avr/interrupt.h>

volatile int toggle = 0;

int main(void) {
    EICRA |= 0b00001100; // Rising edge on INT1 (PD3)
    EIMSK |= 0b00000010; // Enable INT1
    sei(); // Enable global interrupts

    DDRB |= 0b00100000; // Set PB5 as output

    while (1) {
        // Do nothing. Wait for interrupt.
    }
}

ISR(INT1_vect) {
    if (toggle) {
        PORTB |= 0b00100000; // LED ON
    } else {
        PORTB &= 0b11011111; // LED OFF
    }
    toggle = 1 - toggle; // Toggle the value
}
```

---

## Understanding the Status Register (SREG)

SREG is an 8-bit register that holds the result of operations and interrupt status.

| Bit | Name | Function |
|-----|------|----------|
| 7 | I | Global Interrupt Enable |
| 6 | T | Bit Copy Storage |
| 5 | H | Half Carry Flag |
| 4 | S | Sign Flag |
| 3 | V | Overflow Flag |
| 2 | N | Negative Flag |
| 1 | Z | Zero Flag |
| 0 | C | Carry Flag |

### Important Usage
- `sei();` sets the `I` bit to enable global interrupts
- Zero (`Z`) and Carry (`C`) flags are automatically set/reset by arithmetic operations

---

## Datasheet Navigation Guide

To become an expert, you **must learn how to read the datasheet**:

### How to Start
1. Download the **ATmega328P datasheet** (search "ATmega328P datasheet PDF")
2. Use **Table of Contents** to navigate:
   - **I/O Ports** → for PORTB, DDRB, PINB
   - **Interrupts** → for EIMSK, EICRA
   - **Memory and Registers** → for SREG
3. Use CTRL+F to search exact registers (e.g. "EICRA")

### Example: Want to know how to configure INT1?
- Search for `INT1` in datasheet
- Read about `EICRA`, `EIMSK`, and `INT1_vect`

---

## Conclusion

This markdown file gives you everything from setting up simple I/O, writing your first LED blink program, to working with external interrupts. Embedded systems may seem tricky at first, but when broken down clearly, they're fun and powerful.

Share this file on GitHub to help others starting their embedded journey with ATmega328P and Atmel Studio!

---
