# Cortex-M4 Core Registers

## Why do we need registers?

Let's start with the most basic reason.
Suppose, when we write as :

```c
int a = 10;
int b = 20;
int c = a + b;
```

At runtime, a and b ultimately exist somewhere in memory. But the CPU's arithmetic hardware works with values held in its registers.

Conceptually :

```text
RAM
       │
       │ load a
       ▼
      R0
      10

RAM
       │
       │ load b
       ▼
      R1
      20

       │
       │ CPU adds
       ▼

      R2
      30
      
R0 = 10
R1 = 20

R2 = R0 + R1
```
**Registers provide very fast storage directly inside the CPU for values the CPU is actively working with.**

## Cortex-M4 Core Registers

The ARM Cortex-M4 processor inside STM32F407VG has registers divided into groups.

Cortex-M4 Core Registers

```text

|
|-- General Purpose Registers (R0 - R12)
|
|-- Special Registers
|
|-- Program Status Registers
|
|-- Control Registers
|
|-- Floating Point Registers (optional)
```



## 1. General Purpose Registers (R0-R12):

There are 13 general-purpose registers: R0 - R12. **Are R0–R12 identical?** At the hardware level, they are all 32-bit registers. But the ARM procedure-call convention gives different registers different typical roles. 

**R0-R3 (Argument / Return Registers):**

These have special roles according to ARM calling convention.

Example:
```text
R0-R3
|
|-- function arguments
|-- return values
|-- temporary values
```

**R4-R11:**

These are usually preserved across function calls Local/saved values

**R12:**

Often called as *Intra-procedure scratch register*

Mostly temporary use by compiler.

## 2. Special Registers :

-   R13 : Stack Pointer (SP)
-   R14 : Link Register (LR)
-   R15 : Program Counter (PC)

**What is the Stack?**

Before learning the Stack Pointer (SP), we must first understand what the stack actually is.
Imagine your STM32 has this RAM:

```text
RAM
0x20000000
 ---------------------- 
|                      |
|                      |
|                      |
|                      |
|                      |
|                      |
 ---------------------- 
0x2001FFFF
````
The stack is simply a region inside RAM that the CPU uses temporarily while our program is running. It is not a separate type of memory.

```text
RAM

 ---------------------- 
| Stack               |
 ---------------------- 
|                      |
| Free RAM            |
|                      |
 ---------------------- 
| Heap                |
 ---------------------- 
| .bss                |
 ---------------------- 
| .data               |
 ---------------------- 
````
Everything above is still RAM. The linker/startup code simply gives different purposes to different regions.
Whenever a function runs, the CPU needs temporary storage. For example,

```c
void add(void)
{
    int x = 10;
    int y = 20;
}
```
Here, the variables x and y are temporary. Once add() finishes, they are no longer needed. Instead of putting them in .data forever, the CPU stores them on the stack. It follows LIFO (Last In, First Out) . Here, the stack space is released, not necessarily RAM is erased. The stack grows downward.

The CPU doesn't usually clear the old values from RAM. It simply moves the Stack Pointer (SP), saying "This memory is now free to use again." So if we looked at RAM immediately afterward with a debugger, we might still see 10 and 20 there. But they are no longer valid variables—that memory can be overwritten by the next function call. Thats why returning the address of a local variable is dangerous.

**If the stack is just an area in RAM, how does the CPU know where the current top of the stack is?** 

The linker creates the symbol `_estack = 0x20020000;`. It does not write to the CPU register. The startup file uses `_estack`. So the first word of the Vector Table contains the value of _estack. When the MCU comes out of reset, before any C code or Reset_Handler runs, the Cortex-M4 hardware automatically does:

```text
Read first word of Vector Table
        │
        ▼
MSP = 0x20020000
```

**The Stack Pointer (SP) register always contains the address of the current top of the stack.** When a function is called, the Stack Pointer moves towards lower memory addresses because the Cortex-M4 stack grows downward. Because the stack starts at the highest RAM address. The Stack Pointer value decreases as stack memory is allocated.
