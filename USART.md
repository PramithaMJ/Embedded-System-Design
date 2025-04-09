# USART in ATmega328P/2560 - Explained

### What is USART?
USART stands for **Universal Synchronous and Asynchronous Receiver and Transmitter**. It is a hardware peripheral used to communicate serially with other devices.

- In **Asynchronous mode**, the devices do **not share a clock**. Instead, the communication protocol is based on agreed timing (baud rate).
- USART enables **bi-directional** serial communication.

---

### Why is USART said to be asynchronous?
USART is called asynchronous because it does **not require a common clock signal** between the transmitter and receiver. Instead:
- The transmitter and receiver agree on a **baud rate**.
- Each character is framed by **start and stop bits**, making timing recovery possible.
- There's **no synchronization signal** like in synchronous protocols.

---

### What is Baud Rate?
**Baud rate** refers to the **number of signal changes per second**. For USART:

> Baud Rate = Bits Per Second (bps)

Example:
- Baud Rate = 9600
- This means the line transmits **9600 bits per second**.

---

### USART Settings to Match Between Transmitter and Receiver
To ensure proper communication, both devices must have the same configuration:

| Setting                   | Description                                        |
|--------------------------|----------------------------------------------------|
| **Baud rate**            | Must be exactly the same                          |
| **Data bits per packet** | Usually 8 bits; options: 5, 6, 7, 8, or 9          |
| **Stop bits**            | 1 or 2                                            |
| **Parity bit**           | None, Even, or Odd                                |
| **Flow control**         | Optional hardware control (e.g., RTS/CTS)         |

Any mismatch can result in data corruption or failed transmission.

---

### USART Transmission Steps in ATmega328P/2560

#### Step-by-step Explanation:

#### Step 1: Configure USART
```c
#define F_CPU 16000000UL           // CPU Frequency
#define BAUD 9600                  // Desired Baud Rate
#define UBRR_VALUE ((F_CPU/16/BAUD)-1)  // Baud rate register value

#include <avr/io.h>
#include <util/delay.h>

void initUSART(void) {
    UBRR0H = (unsigned char)(UBRR_VALUE >> 8); // Set UBRR upper byte
    UBRR0L = (unsigned char)UBRR_VALUE;        // Set UBRR lower byte

    UCSR0B = (1 << TXEN0);                     // Enable Transmitter only
    UCSR0C = (3 << UCSZ00);                    // Set 8-bit data
}
```

Explanation:
- **UBRR0H and UBRR0L**: Used to set the baud rate.
- **TXEN0**: Enables the USART transmitter.
- **UCSZ00 and UCSZ01**: Set the character size to 8-bit.

#### Step 2: Wait for the Transmit Buffer to be Ready
```c
while (!(UCSR0A & (1 << UDRE0))) {
    // Wait until the transmit buffer is empty
}
```

#### Step 3: Send Data
```c
UDR0 = data; // Load data into buffer, sends the data
```

---

### USART Reception and Transmission
- The USART can also receive data in the same asynchronous mode.
- When a byte is received, it is stored in a register and a flag is set.
- The program can then read the byte and optionally echo it back via the transmitter.

---

### USART Reception with Interrupts
Instead of continuously checking if data has arrived (polling), USART can be configured to **generate an interrupt** when data is received.

This lets the CPU handle other tasks and only respond when an interrupt occurs.

#### Steps:
1. Enable USART Receive Complete Interrupt.
2. Write an ISR (Interrupt Service Routine).
3. In the ISR, read the received byte and process it.

---

### Summary
USART is a powerful tool for serial communication. Key takeaways:
- Asynchronous = No shared clock.
- Baud rate must match.
- Configure data bits, stop bits, and parity correctly.
- Transmission is simple: set registers, wait for buffer, send data.
- Reception can be handled via polling or interrupts.

---
