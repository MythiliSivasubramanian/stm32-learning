# Cortex-M4 Core Registers

The ARM Cortex-M4 processor inside STM32F407VG has registers divided into groups.

Cortex-M4 Core Registers

```text

|
|-- General Purpose Registers
|
|-- Special Registers
|
|-- Program Status Registers
|
|-- Control Registers
|
|-- Floating Point Registers (optional)
```


## 1. General Purpose Registers (R0-R12)

There are 13 general-purpose registers: R0 - R12 and each register is 32 bit wide as its Cortex M4 is a 32 bit processor. They are temporary storage locations inside the CPU. Instead of: 
```c 
int a = 10; 
int b = 20; 
int c;
c = a + b; ```

The CPU does not directly calculate from RAM. It does something like:

```text
RAM
 |
 | Load a
 ↓
R0 = 10


RAM
 |
 | Load b
 ↓
R1 = 20


CPU calculation : R2 = R0 + R1

Registers are much faster than RAM.




