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
## 2. Common Terms and Definitions

### Register
A register is a fixed-width storage location inside the microcontroller.
STM32 registers are memory-mapped, which means the CPU reads and writes them through addresses.

Example:
- `GPIOA->ODR` is the output data register for GPIO port A.
- `RCC->AHB1ENR` controls clock enables for peripherals.

### Mask
A mask is a binary value used to select or deselect bits.
A mask has `1`s where we want to keep or change bits and `0`s where we want to ignore bits.

Example mask:

    0b00000100

This mask selects only bit position 2.

### Field
A field is a group of bits within a register that represent one setting.
For example, a 2-bit field might select a GPIO pin mode:
- `00` = input
- `01` = output
- `10` = alternate function
- `11` = analog

---

## 3. Bitwise Operators

### 3.1 Bitwise AND (`&`)
The AND operator compares two binary values bit by bit.
The result is `1` only when both input bits are `1`.

Truth table:
- `1 & 1 = 1`
- `1 & 0 = 0`
- `0 & 1 = 0`
- `0 & 0 = 0`

Example:

    value = 0b11001100;
    mask  = 0b10101010;
    result = value & mask; // 0b10001000

Use case:
- Clear bits that correspond to `0` in the mask.
- Keep only bits that correspond to `1` in the mask.

### 3.2 Bitwise OR (`|`)
The OR operator compares two binary values bit by bit.
The result is `1` if at least one input bit is `1`.

Truth table:
- `1 | 1 = 1`
- `1 | 0 = 1`
- `0 | 1 = 1`
- `0 | 0 = 0`

Example:

    value = 0b11001100;
    mask  = 0b10101010;
    result = value | mask; // 0b11101110

Use case:
- Set bits to `1` without changing other bits.

### 3.3 Bitwise XOR (`^`)
The XOR operator compares two binary values bit by bit.
The result is `1` when exactly one input bit is `1`.

Truth table:
- `1 ^ 1 = 0`
- `1 ^ 0 = 1`
- `0 ^ 1 = 1`
- `0 ^ 0 = 0`

Example:

    value = 0b11001100;
    mask  = 0b10101010;
    result = value ^ mask; // 0b01100110

Use case:
- Toggle bits that correspond to `1` in the mask.

### 3.4 Bitwise NOT (`~`)
The NOT operator inverts every bit.
All `1`s become `0`s, and all `0`s become `1`s.

Example:

    value = 0b11001100;
    result = ~value; // 0b00110011 on an 8-bit level

Use case:
- Build a mask for clearing bits by inverting a selection mask.

### 3.5 Left shift (`<<`)
The left shift operator moves bits to the left and fills the right side with zeros.
Each left shift multiplies the value by 2.

Example:

    1 << 0 = 0b00000001
    1 << 1 = 0b00000010
    1 << 2 = 0b00000100

Common use:
- Create a bit mask for a single position.

Example:

    uint32_t bit = 1U << 5; // selects bit 5

### 3.6 Right shift (`>>`)
The right shift operator moves bits to the right.
Each right shift divides the value by 2 (for unsigned values).

Example:

    0b00010000 >> 2 = 0b00000100

Common use:
- Extract a field value from a register.

Example:

    uint32_t value = 0b11010000;
    uint32_t field = value >> 4; // field = 0b00001101

---

