## 01_Interrupts_NVIC_practicals (STM32F407_Interrupt_Practicals)

Recording actual register-level experiments, C code, debugger observations and screenshots all without HAL. 

Before doing any practical experiments, to understand the basic terms of circuit diagrams, file [01_Basic_Electrical_Concepts](../01_Fundamentals/01_Basic_Electrical_Concepts/01_Basic_Electrical_Concepts.md) would be helpful.

### Goal:
Create an external interrupt when the User Button B1 is pressed. When B1 is pressed, the onboard LED should light up.


**Board:** STM32F407G-DISC1
**Language:** C
**Approach:** Bare-metal, register-level programming, without HAL and No LL


Before using interrupts, will make the onboard LED respond to the USER button using polling because a working baseline is required before replacing polling with interrupts.

**Hardware mapping:** STM32F407G-DISC1 User Manual, DocID022256 Rev 6.

|Component	| STM32 pin |
|-----------|-----------|
|B1 USER button	| PA0 |
|LED4 Green	| PD12 |
|LED3 Orange	| PD13 |
|LED5 Red	 | PD14 |
|LED6 Blue	| PD15 |

### Project 1: 
#### GPIO Polling:

The first experiment will use:
-   PA0 as a GPIO input to read the USER button.
-   PD12 as a GPIO output to control the green LED.

Polling will be implemented before introducing interrupts.

Starting with 1 LED, which is Green LED4 on PD12. Before configuring it, the clock for Port D needs to be enabled. 

### RCC Register Identification:

The STM32F407 has a clock-control peripheral called RCC (Reset and Clock Control). GPIO peripherals need their clocks enabled before we can use them. The register used is **RCC_AHB1ENR** (Reset and Clock Control, AHB1 bus, Enable Register).
**RCC_AHB1ENR = Reset and Clock Control – AHB1 Peripheral Clock Enable Register.** 

In simple terms:
- RCC is the peripheral that controls clocks and resets.
- AHB1 identifies the bus group.
- EN tells us this register controls clock enabling.
- R means register.

Why does AHB1 matter?
GPIOA, GPIOB, GPIOC, GPIOD, and several other peripherals are connected to the AHB1 bus. For our practical, we need to enable the GPIOD clock by setting bit 3. Bit 3 of RCC_AHB1ENR controls the GPIOD peripheral clock.

## Enable GPIOD clock
Enable the GPIOD peripheral clock in C by understaning bit shifting in c using register-level programming.. We can enable the GPIO Port D by updating the respective bit (bit 3) of RCC_AHB1EN. I.e Basically we wantb the bit 3 of this register RCC_AHB1EN to be set to 1, without changing the other bits of the register. In main.c,  eventually written as : `RCC->AHB1ENR |= (1U << 3);`. 
Detailed explanation in file [Mastering bitwise](../../../embedded_systems_foundations/02_C_for_Embedded/03_mastering_bitwise.md)
 
 Let's break down the operation: RCC->AHB1ENR |= (1U << 3);
- 1U << 3 creates a bit mask with only bit 3 set. 
- |= performs a bitwise OR and writes the result back.
- Existing bits remain unchanged; bit 3 becomes 1.
Practical result: The GPIOD peripheral clock is enabled.

We use |= instead of simply writing RCC->AHB1ENR = (1U << 3);, because 
-   |= sets the selected bit to 1 while preserving the other bits of the register RCC->AHB1ENR
-   = clears the entire register bits and only sets bit 3 as 1. This affects the other bits, which has various other functionalities. 

So, after this instruction, ```c RCC->AHB1ENR |= (1U << 3);```, bit 3 of RCC_AHB1ENR is set (GPIOD clock enabled). #

Next, need to configure the LED pin. Enabling the peripheral clock does not automatically make PD12 an output. Need to configure the GPIO pin using the Mode Register ```GPIOD_MODER```. The relevant register is MODER (GPIO Port Mode Register).

![alt text](../screenshots/1.png)

How many bits are used to configure the mode of a single GPIO pin in MODER? Each pin supports four possible modes.

|Bits|	Mode|
|-----|-----|
|00	|Input|
|01	|General-purpose output|
|10	|Alternate function|
|11 |	Analog|

Each GPIO pin uses 2 bits in MODER. STM32F407 GPIO ports have 16 pins, numbered PD0, PD1, PD2, ... PD15

Therefore, GPIOD's MODER register has 32 bits: 16 * 2 = 32 bits. Each GPIO pin uses 2 bits in MODER. Since each pin uses two bits, so for PD12, 2 * 12 = 24. So bit 24 and bit 25 of GPIOD_MODER corresponds to PD12. Therefore:
- Bit 24 = 0
- Bit 25 = 1
Together: 01 → General-purpose output mode.

Basically,
-   We want to configure PD12 as an output. PD12 is controlled by bits 24 and 25 of GPIOD_MODER.
-   And we need those two bits to become 01.

The binary representation of decimal 3 is ```0000 0011``` The two rightmost bits are both 1. Binary 11 gives us two adjacent bits set to 1. Need a mask that targets exactly two bits: bit 24 and bit 25. So, shift 3U left by 24 positions: ```c (3U << 24)```.

The result could be:
bit 24 : 1
bit 25 : 1
All other bits : 0

Here, 1U is not used, since 1U contains only one set bit. But two set bits are required in mask to target both PD12 mode bits. Next is that these both bits should become zero without affecting any other bits. Bitwise AND (&) with a negated mask (~). ```GPIOD->MODER &= ~(3U << 24);```  

~(3U << 24) creates a mask where:
- Bits 24 and 25 are 0.
- All other bits are 1.

Then we use bitwise AND (&) to clear only those two bits in the register. Desired mode for PD12 is 01. First, clear both bits:
```c GPIOD->MODER &= ~(3U << 24);``` Then set bit 24 ```c GPIOD->MODER |= (1U << 24);``` The result is:
- Bit 25 = 0
- Bit 24 = 1

Now PD12 is configured as a general-purpose output.

***Quick Notes:***
-   3U << 24 creates a two-bit mask targeting bits 24 and 25.
-   ~(3U << 24) prepares those two bits for clearing.
-   &= clears both bits without changing the others.
-   |= (1U << 24) sets bit 24 to 1.

Firest, enabled the GPIOD clock and now configured PD12 as an output. But this doesnt automatically turn ON the green LED.  need Another register to control the actual output level is reequired, which is ***GPIOD_ODR (Output Data Register)***

|Register	|Purpose |
|GPIOD_MODER	| Configures PD12 as an output |
|GPIOD_ODR	| Controls whether PD12 outputs HIGH or LOW |