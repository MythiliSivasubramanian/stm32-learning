# STM32 Foundations: Notes

## What STM32 is

STM32 is a family of microcontrollers made by STMicroelectronics.
This is not just one chip, it is a whole ecosystem of ARM-based controllers.
The key idea is: a small, affordable CPU with built-in peripherals for real-world control.

STM32 is used when I need a compact controller that can handle sensors, motors, communication, and timing without a lot of external chips.

---

## What makes STM32 special

STM32 is special because it is both flexible and widely supported.
It gives:
- a real ARM Cortex core instead of a tiny 8-bit CPU
- lots of peripherals ready to use
- good support from tools, libraries, and community
- a consistent model across many families

It feels like a good middle ground:
- more power than 8-bit microcontrollers
- less complexity than a full application processor

---

## The main STM32 families

STM32 is grouped by family names. Each family has a different focus.

### F0 / L0 / G0
These are entry-level parts.
- smaller flash and RAM
- lower cost
- enough for simple control tasks
- good for learning the basics

### F1
A classic STM32 family.
- popular in many hobby and industrial boards
- balanced features
- good first STM32 to understand

### F3 / G4
Focused on analog and motor control.
- better ADCs
- comparators, DACs, timers
- useful when sensors and analog signals matter

### F4 / L4 / H7
Higher performance.
- faster CPUs
- bigger memory
- more advanced peripherals
- good for complex, real-time tasks

### F7 / H7 and beyond
These are the top-tier parts.
- very fast cores
- extra features like caches and dual-core options
- use them when the project needs the most horsepower

---

## Which documents matter and why

When working with STM32, there are three main documents I always look at:

### 1. Datasheet
This tells me the package, pinout, electrical limits, and features of the exact part.
I use it to answer questions like:
- how many pins are available?
- what is the operating voltage?
- which peripherals are present?

### 2. Reference Manual
This is the real manual for the microcontroller family.
It describes every peripheral and register in detail.
I use it to learn:
- how GPIO works
- how timers are configured
- how UART sends and receives data
- what each register bit does

### 3. Programming Manual / Application Notes
These are extra guides and examples.
They help when I need a deeper explanation or a recommended pattern.
If the datasheet is the "what" and the reference manual is the "how," then application notes are the "why and when."

---

## How I choose the right document

If I am picking a part or checking pin count, I open the datasheet.
If I am writing code for a peripheral, I read the reference manual.
If I need a real example or a common recipe, I use application notes.

That simple rule keeps the work from becoming confusing.

---

## Why understanding the family is useful

Each STM32 family has a different set of peripherals and strengths.
That means if I assume all STM32 chips are the same, I will make mistakes.

For example:
- F0 might not support a certain timer mode
- L4 may have a better ADC but lower CPU speed
- H7 is powerful, but the clock tree is more complex

So the first step is always: know the family before writing code.

---

## A few practical notes

- The part number encodes the family, package, flash size, and speed.
- STM32 boards often use ST-LINK for programming and debugging.
- The reset vector and startup code are important when I work at register level.
- The power and clock system is the foundation for everything else.

---

## How this fits with the rest of STM32 learning

I want the basics to feel natural:
- STM32 is an ARM microcontroller family
- families are chosen based on performance and peripherals
- datasheets, reference manuals, and application notes are the documents I trust
- the controller is special because it is powerful, well-supported, and flexible

Once these ideas are clear, the next step is to look at a real part and its exact registers.