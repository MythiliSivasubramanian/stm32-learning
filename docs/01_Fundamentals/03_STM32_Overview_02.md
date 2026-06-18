# STM32 Overview

## Why STM32?

In embedded systems, the microcontroller is the central component responsible for executing the program and controlling peripherals. STM32 is one of the most widely used microcontroller families because it provides a good balance between performance, peripherals, power consumption, and cost.

Many embedded applications are built around it, ranging from simple sensor-based systems to industrial automation and automotive electronics.

---

## STM32 Family

STM32 is a family of 32-bit microcontrollers developed by STMicroelectronics and built around ARM Cortex processor cores.

Different STM32 series target different application requirements.

| Series  | Core      | Focus                             |
| ------- | --------- | --------------------------------- |
| STM32F0 | Cortex-M0 | Entry-level applications          |
| STM32F1 | Cortex-M3 | General-purpose embedded systems  |
| STM32F4 | Cortex-M4 | High-performance embedded systems |
| STM32F7 | Cortex-M7 | Advanced real-time applications   |

Although the processor core changes between series, the overall architecture remains similar. This allows developers to move between STM32 families without starting from scratch.

---

## Main Building Blocks of an STM32 Microcontroller

An STM32 microcontroller integrates multiple hardware components into a single chip.

### CPU

The CPU executes instructions and controls the overall operation of the system.

Example:

* Cortex-M4 in STM32F4
* Cortex-M7 in STM32F7

The CPU is responsible for processing data and coordinating peripheral operations.

### Flash Memory

Flash memory stores the application program permanently.

The program remains stored even after power is removed.

### SRAM

SRAM is used during program execution.

Variables, stack memory, and temporary data are stored here.

### Peripherals

Peripherals allow the microcontroller to communicate with the outside world.

Common peripherals include:

* GPIO
* Timers
* UART
* SPI
* I2C
* ADC
* DAC
* CAN
* DMA
* RTC

Without peripherals, the CPU would have no practical way to interact with external devices.

---

## STM32F4 Series

The STM32F4 family is based on the ARM Cortex-M4F core and is widely used in embedded applications.

Two important features introduced in this series are:

### Floating Point Unit (FPU)

The FPU performs floating-point calculations in hardware.

Applications involving sensor processing, control algorithms, and mathematical computations benefit from faster execution when an FPU is available.

### Digital Signal Processing (DSP)

DSP instructions accelerate mathematical operations commonly used in:

* Audio processing
* Signal filtering
* Motor control
* Communication systems

Because of these capabilities, STM32F4 is often selected for applications requiring more computational power than basic microcontrollers can provide.

---

## STM32F407VG

The STM32F407VG is one of the most popular STM32F4 devices.

Important characteristics include:

* Cortex-M4F core
* Clock speed up to 168 MHz
* Flash memory for program storage
* SRAM for runtime data
* Multiple communication interfaces
* ADC and DAC peripherals
* Timers and DMA support

The device combines processing capability and a rich set of peripherals, making it suitable for learning and real-world projects.

---

## STM32F7 Series

The STM32F7 family is based on the Cortex-M7 core.

Compared with STM32F4, it provides:

* Higher clock frequencies
* Improved architecture
* Better performance for computation-intensive applications

The STM32F7 series is typically chosen when additional processing power is required while maintaining real-time operation.

---

## Key Understanding

While STM32 devices contain many peripherals and features, the most important concept is understanding the relationship between the CPU, memory, and peripherals.

A microcontroller can be viewed as:

Input Devices - Peripherals - CPU . Peripherals . Output Devices.