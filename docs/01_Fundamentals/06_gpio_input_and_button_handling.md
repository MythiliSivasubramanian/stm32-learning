# GPIO Input and Button Handling

## Introduction

Reading a button is one of the first real inputs in embedded firmware.
This note explains how GPIO input works in STM32 and how to handle a button cleanly in firmware.

---

## Input pin configuration

A GPIO pin must be configured as input before it can read an external signal.
On STM32, this is done through the MODER register.
Each pin has two bits in MODER:

- 00 = input mode
- 01 = output mode
- 10 = alternate function mode
- 11 = analog mode

To set pin 13 of port C as input:


GPIOC->MODER &= ~(0x3U << (13 * 2));


This clears the two mode bits for PC13 and leaves the other pins unchanged.

---

## Pull-up and pull-down resistors

Most input pins should not be left floating.
A floating pin can randomly read high or low depending on noise and the surrounding circuitry.
STM32 provides internal pull-up and pull-down resistors that keep the pin in a stable state when nothing is driving it.

The PUPDR register controls the internal pull behavior:

- 00 = no pull
- 01 = pull-up
- 10 = pull-down

For a button that connects the pin to ground when pressed, use pull-up:


GPIOC->PUPDR &= ~(0x3U << (13 * 2));
GPIOC->PUPDR |=  (0x1U << (13 * 2));


This makes the pin read high when the button is released and low when pressed.

---

## Reading pin state

The IDR register reports the current level on the pin.
To read a single pin, shift and mask the bit from IDR:


uint32_t button_state = (GPIOC->IDR >> 13) & 1U;


If button_state is 1, the pin is high.
If it is 0, the pin is low.

---

## Debounce awareness

Mechanical buttons do not change state perfectly cleanly.
When pressed or released, the contacts may bounce and produce a series of transitions.
A simple firmware strategy is to sample the button state with a short delay and require the state to remain stable.

Example concept:


if ((GPIOC->IDR & (1U << 13)) == 0)
{
    delay_ms(10);
    if ((GPIOC->IDR & (1U << 13)) == 0)
    {
        // button is confirmed pressed
    }
}


This reduces false triggers from contact bounce.

---

## Edge detection and events

A button press is often an event rather than a continuous state.
To detect edges cleanly, compare the current pin state with the previous state.

This pattern works well in a polling loop:


uint32_t previous_state = 1;

while (1)
{
    uint32_t current_state = (GPIOC->IDR >> 13) & 1U;

    if (previous_state == 1 && current_state == 0)
    {
        // button went from released to pressed
    }

    previous_state = current_state;
}


This captures the moment the button is pressed instead of repeating while it remains down.

---

## Practical button wiring

A stable button circuit often uses one side of the switch tied to ground and the other side to the GPIO pin.
The internal pull-up resistor pulls the pin high when the button is open.
When the button closes, the pin is pulled low.

This arrangement is simple and robust for most STM32 boards.

---

## Using external pull resistors

If the board design already includes external pull-up or pull-down resistors, internal pulls can be disabled.
External resistors are useful when the button is part of a more complex signal chain or when higher reliability is required.

When using external pull resistors, set PUPDR to 00 for no internal pull.

---

## Summary

GPIO input and button handling depend on a few key steps:

- configure the pin as input using MODER
- select a stable default state with PUPDR
- read the pin through IDR
- handle button bounce with a brief delay or state validation
- detect edges by comparing current and previous pin state
