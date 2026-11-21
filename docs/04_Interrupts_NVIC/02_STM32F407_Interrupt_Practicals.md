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

Starting with 1 LED, which is Green LED4 on PD12. Before configuring it, the clock for Port D needs to be enabled. The STM32F407 has a clock-control peripheral called RCC (Reset and Clock Control). GPIO peripherals need their clocks enabled before we can use them. The register used is **RCC_AHB1ENR** (Reset and Clock Control, AHB1 bus, Enable Register).
**RCC_AHB1ENR = Reset and Clock Control – AHB1 Peripheral Clock Enable Register.**

In simple terms:
- RCC is the peripheral that controls clocks and resets.
- AHB1 identifies the bus group.
- EN tells us this register controls clock enabling.
- R means register.

Why does AHB1 matter?
GPIOA, GPIOB, GPIOC, GPIOD, and several other peripherals are connected to the AHB1 bus. For our practical, we need to enable the GPIOD clock by setting bit 3.

## Enable GPIOD clock
Enable the GPIOD clock in C. In main.c,  eventually written as : `RCC->AHB1ENR |= (1U << 3);`.
