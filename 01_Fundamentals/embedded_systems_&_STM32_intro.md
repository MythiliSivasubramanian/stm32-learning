# Embedded Systems and STM32 foundations: Notes

## What is Embedded Systems?

Embedded systems are computer systems designed to perform a specific task.
Unlike a laptop or phone (general-purpose systems), embedded systems are built to do one or a few dedicated jobs.

### Examples:
- Washing machine controller
- Microwave oven timer system
- Car engine control unit
- Smartwatch
- Traffic light controller

### Key idea:
Embedded system = Software + Hardware working together to control real-world devices.

---

## What is Embedded Programming?

Embedded programming is writing code that directly interacts with hardware.

Instead of:
- printing output on screen

We do:
- turning ON/OFF LEDs
- reading sensors
- controlling motors
- sending data over communication protocols

Goal: Making physical devices behave intelligently using code.

---

## What is a Microcontroller?

A microcontroller is a small computer on a single chip.

It contains:

### 1. CPU (Central Processing Unit)
    - Brain of the system
    - Executes instructions written in C

### 2. Memory
    - Flash → stores program code
    - RAM → temporary data storage

### 3. Peripherals
Hardware modules to interact with real world:

    - GPIO (General Purpose Input Output)
    - UART (communication)
    - SPI / I2C (data transfer)
    - Timers
    - ADC (Analog to Digital Converter)

---

## What is STM32?

STM32 is a family of microcontrollers made by STMicroelectronics.

They are widely used in:

    - industrial systems
    - automotive electronics
    - IoT devices
    - robotics

### Why STM32?

    - Powerful and fast
    - Industry standard
    - Many built-in peripherals
    - Used in real embedded systems development

---

## How STM32 Works (Simple Model)

C Program -> Compiled into machine code -> Loaded into STM32 memory (Flash)
-> CPU executes instructions -> Hardware reacts (LED ON/OFF, sensors, etc.)


---

## Important Concepts: 

## GPIO

GPIO = General Purpose Input Output. It controls pins of STM32.

Each pin can be:
- Input (read button/sensor)
- Output (control LED/motor)

---

## Basic Idea of Pin Control

A microcontroller pin behaves like a switch:

- HIGH (1) → ON (3.3V)
- LOW (0) → OFF (0V)

So:
- LED ON = send HIGH signal
- LED OFF = send LOW signal

---