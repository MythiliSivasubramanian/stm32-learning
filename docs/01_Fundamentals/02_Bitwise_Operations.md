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

## 4. Detailed Examples with Binary

### Example 1: Set bit 3

    value = 0b00001010; // 10 decimal
    mask  = 1 << 3;     // 0b00001000
    result = value | mask; // 0b00001010 | 0b00001000 = 0b00001010

If bit 3 was already `1`, the value stays the same.

### Example 2: Clear bit 3

    value = 0b00001110; // 14 decimal
    mask  = ~(1 << 3);  // 0b11110111
    result = value & mask; // 0b00001110 & 0b11110111 = 0b00000110

This clears bit 3 and keeps the other bits unchanged.

### Example 3: Toggle bit 1

    value = 0b00000101; // 5 decimal
    mask  = 1 << 1;     // 0b00000010
    result = value ^ mask; // 0b00000111 (7 decimal)

If bit 1 were `1`, this would clear it instead.

### Example 4: Check bit 2

    value = 0b00000100;
    mask  = 1 << 2;
    if (value & mask) {
        // bit 2 is set
    }

The expression is non-zero only when the bit is `1`.

---

## 5. STM32 Register Examples

### 5.1 Set a GPIO pin high

STM32 output data registers use one bit per pin.
To set pin 5 of GPIOA high:

    GPIOA->ODR |= (1U << 5);

Explanation:
- `GPIOA->ODR` is the output data register.
- `1U << 5` creates a mask with bit 5 set.
- `|=` sets just that bit.

### 5.2 Set a GPIO pin low

To clear pin 5 of GPIOA:

    GPIOA->ODR &= ~(1U << 5);

Explanation:
- `~(1U << 5)` creates a mask with all bits `1` except bit 5.
- `&=` clears bit 5 while preserving all other pins.

### 5.3 Configure a GPIO pin mode

Suppose `GPIOA->MODER` uses two bits per pin:
- `00` = input
- `01` = output
- `10` = alternate function
- `11` = analog

To make PA5 a general-purpose output:

    GPIOA->MODER &= ~(3U << (5 * 2)); // clear mode bits for PA5
    GPIOA->MODER |=  (1U << (5 * 2)); // set mode to output

Explanation:
- `3U << (5 * 2)` selects the two bits for pin 5.
- `&= ~mask` clears those bits.
- `|= 1U << (5 * 2)` writes `01` into the field.

### 5.4 Enable a peripheral clock

To enable the clock for GPIOA in RCC AHB1ENR:

    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;

To disable it:

    RCC->AHB1ENR &= ~RCC_AHB1ENR_GPIOAEN;

These bitwise operations control hardware power and clock gating.

---

## 6. Important Notes and Best Practices

### 6.1 Use parentheses
Always use parentheses around shifts and masks when combining operations.

Good:

    register &= ~(1U << pin);

Bad:

    register &= ~1U << pin; // wrong: operator precedence breaks the mask

### 6.2 Prefer `1U` for bit masks
Using unsigned constants avoids unexpected sign extension.

Good:

    1U << 5

Bad:

    1 << 31 // may be signed and undefined on some compilers

### 6.3 Work with named masks
Using named constants makes code easier to read.

Example:

    #define GPIOA_PIN5 (1U << 5)

    GPIOA->ODR |= GPIOA_PIN5;

### 6.4 Read before write when necessary

Some registers are write-only, while others are read-modify-write.
Always check the STM32 reference manual for the register behavior.

---

## 7. Summary

Bitwise operations are the tools that let embedded programmers control hardware one bit at a time.
In STM32 programming, registers are collections of bits, and understanding how to use masks, shifts, and logical operators is essential.

Key points:


- `&` selects bits
- `|` sets bits
- `^` toggles bits
- `~` inverts bits
- `<<` and `>>` move bits

Use these operations to modify only the bits you need and keep the rest of the register unchanged.


