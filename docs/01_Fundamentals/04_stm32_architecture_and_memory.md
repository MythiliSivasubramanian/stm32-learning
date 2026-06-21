# STM32 Architecture and Memory

## Introduction

Every STM32 microcontroller is built around three fundamental components:

1. CPU
2. Memory
3. Peripherals

The CPU executes instructions, memory stores data and program code, and peripherals provide the interface between the microcontroller and external hardware.

---

## CPU

The CPU is the processing unit responsible for executing instructions stored in memory.

STM32 microcontrollers use ARM Cortex processor cores.

Examples:

* Cortex-M0
* Cortex-M3
* Cortex-M4
* Cortex-M7

The CPU continuously performs the following cycle:

1. Fetch instruction from memory
2. Decode instruction
3. Execute instruction

This process repeats as long as the microcontroller is powered.

---

## Memory Organization

Memory inside an STM32 microcontroller can be divided into two major categories:

### Flash Memory

Flash memory stores the application program.

Characteristics:

* Non-volatile
* Retains data after power loss
* Contains executable code

Example:

When a program is flashed into the microcontroller, it is stored in Flash memory.

---

### SRAM

Static RAM stores data required while the program is running.

Characteristics:

* Volatile memory
* Data is lost when power is removed
* Faster than Flash memory

Examples of data stored in SRAM:

* Variables
* Stack
* Function parameters
* Temporary calculations

---

## Program Execution Flow

A program is stored in Flash memory.

When the microcontroller starts:

1. CPU begins execution from a predefined memory location.
2. Instructions are fetched from Flash memory.
3. Variables are allocated in SRAM.
4. The program continues executing until reset or power removal.

---

## Stack and Heap

Memory in SRAM is typically divided into different regions.

### Stack

The stack is used for:

* Function calls
* Local variables
* Return addresses

Stack memory is automatically managed by the processor.

Example:


void display(void)
{
    int count = 10;
}


The variable `count` is stored on the stack.

### Heap

The heap is used for dynamic memory allocation.

Example:


int *ptr = malloc(sizeof(int));


In embedded systems, heap usage is often minimized because memory resources are limited.

---

## Peripherals

Peripherals are hardware modules integrated into the microcontroller.

Common STM32 peripherals include:

* GPIO
* Timers
* UART
* SPI
* I2C
* ADC
* DAC
* DMA

The CPU controls peripherals through memory-mapped registers.

Instead of directly controlling hardware signals, software configures registers to determine peripheral behavior.


---

## Memory-Mapped I/O Concept

STM32 uses memory-mapped I/O.

This means peripheral registers occupy specific memory addresses.

Example:

GPIOA->ODR = 0x01;


Although this looks like a variable assignment, the CPU is actually writing to a hardware register.

The register then changes the state of a physical pin.


---

## Key Understanding

The CPU alone cannot interact with the outside world.

Peripherals alone cannot make decisions.

Memory alone cannot execute instructions.

A microcontroller functions correctly only when all three work together:

CPU ↔ Memory ↔ Peripherals