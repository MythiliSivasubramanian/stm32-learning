# Bitwise Operations: STM32 Fundamentals

Bitwise operations are the foundation of low-level embedded programming.
They allow you to read, write, and modify individual bits inside memory and hardware registers.
In STM32 development, every peripheral register is a group of bits, and each bit or group of bits often represents one hardware function.

---

## 1. Basic Concepts

### 1.1 Bit definition
A bit is the smallest unit of digital information.
It can have only two values:
- `0` (off)
- `1` (on)

A single bit represents one binary choice.

### 1.2 Byte definition
A byte is a group of 8 bits.
For example, the byte `0b01010110` contains 8 bits.

### 1.3 Importance of bits in embedded systems
In STM32 microcontrollers, hardware registers are typically 32 bits wide.
Each bit may mean:
- pin output state
- peripheral enable/disable
- mode selection
- interrupt flag

This means you rarely change the whole register at once.
Instead, you use bitwise operations to change only the bits you want.

### 1.4 Bit numbering
Bits are numbered from right to left, starting at `0`.
For an 8-bit binary number:

    0b10110010
     bit7...bit0

The rightmost bit is `bit0` and the leftmost bit is `bit7`.

For STM32 32-bit registers, valid bit positions are `0` through `31`.

---

