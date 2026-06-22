# GPIO and Pin Control

## Introduction

GPIO is the most basic way STM32 interacts with the outside world.
A GPIO pin is a physical connection on the microcontroller that can be configured as an input or output.
Understanding GPIO is essential before working with buttons, LEDs, sensors, or any digital peripheral.

---

## What GPIO does

GPIO stands for General Purpose Input Output.
It is the interface between the microcontroller core and the external hardware.
A single GPIO pin can do one of two things:

- behave as an input and read an external signal
- behave as an output and drive an external signal

In STM32, GPIO pins are grouped into ports such as GPIOA, GPIOB, GPIOC, and so on.
Each port contains a number of pins, typically 16 pins per port.

---

## Pin modes

A GPIO pin in STM32 can be configured in several modes.
The most common modes are:

- Input mode
- Output mode
- Alternate function mode
- Analog mode

For basic digital control, input and output modes are the ones used most often.

### Output mode

When a pin is configured as output, software can drive it high or low.
A high output usually means 3.3V and a low output means 0V.
This is how LED control, relay signals, and digital actuators are implemented.

### Input mode

When a pin is configured as input, software reads the pin state.
This is how buttons, switches, and digital sensors are sampled.

---

## GPIO registers

GPIO uses memory-mapped registers to control pin behavior.
The key registers for a port are:

- MODER — selects mode for each pin
- OTYPER — selects output type (push-pull or open-drain)
- OSPEEDR — selects output speed
- PUPDR — selects pull-up or pull-down resistors
- IDR — reads input data
- ODR — writes output data
- BSRR — sets or resets output bits atomically

Each register is 32 bits wide.
The bit fields are arranged so that each pin occupies a fixed range of bits inside the register.

---

## Setting a pin as output

To make a pin into a digital output, the MODER register is used.
Each pin uses two bits in MODER:

- 00 = input
- 01 = output
- 10 = alternate function
- 11 = analog

For example, to configure pin 5 of port A as output:


GPIOA->MODER &= ~(0x3U << (5 * 2)); // clear mode bits for PA5
GPIOA->MODER |=  (0x1U << (5 * 2)); // set PA5 as output


This sequence keeps the other pin settings unchanged while modifying only PA5.

---

## Driving a pin high or low

Once a pin is configured as output, the value can be written through ODR or BSRR.
Using ODR directly is simple and clear:


GPIOA->ODR |=  (1U << 5);  // set PA5 high
GPIOA->ODR &= ~(1U << 5);  // set PA5 low


Using BSRR is safer in interrupt-driven code because it changes only one bit without affecting others:


GPIOA->BSRR = (1U << 5);       // set PA5
GPIOA->BSRR = (1U << (5 + 16)); // reset PA5


---

## Reading a pin state

To read a GPIO pin configured as input, use the IDR register.
Each bit in IDR corresponds to the current logic level on a pin.

Example: read the state of pin 13 on port C:


uint32_t pin_state = (GPIOC->IDR >> 13) & 1U;

If the value is 1, the pin is high.
If the value is 0, the pin is low.

---

## Pull-up and pull-down resistors

Many STM32 pins can be configured with internal pull-up or pull-down resistors.
This is useful when the pin is used as an input and the external signal may float.

The PUPDR register controls this behavior:

- 00 = no pull
- 01 = pull-up
- 10 = pull-down
- 11 = reserved

A button input often uses pull-up so the pin is stable when the button is open.

---

## Practical notes

GPIO configuration is one of the first things done in the initialization phase.
Before using a pin, the peripheral clock for the corresponding GPIO port must be enabled.

For example, enabling clock for port A is usually done through the RCC register:

c
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;


After the clock is enabled, the port registers become writable.

Another practical point is that output speed and type matter for real signals.
Push-pull is the default output type, and low or medium speed is often enough for simple LEDs.
High-speed outputs are reserved for fast communication or PWM.

---

## Summary

GPIO is the bridge between STM32 and the physical world.
A pin can be configured as input or output with MODER, driven with ODR or BSRR, and read through IDR.
Keeping register updates precise and using the correct pull resistor makes GPIO behavior reliable.
This foundation is the starting point for all embedded firmware that controls sensors, lights, buttons, and motors.
