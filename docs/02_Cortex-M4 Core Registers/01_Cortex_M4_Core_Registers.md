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
    ├-- R13 → SP
    ├── R14 → LR
    ├── R15 → PC
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


## R13 — Stack Pointer Register :

R13 is the Stack Pointer register of the Cortex-M4. Cortex-M4 actually has two stack pointers.
1.   MSP : Main Stack Pointer
2.   PSP : Process Stack Pointer

The SP contains a RAM address. More specifically, it points to the current location of the stack.

For example:

```text
RAM

0x20020000  ← top of RAM
     ↑
    SP

At some moment: SP = 0x20020000
```

Then the CPU uses the stack. The SP might move to: SP = 0x2001FFF0
```text
SP
 ↓
current stack location
```

**Why do we need a Stack Pointer?**

Think about a function:
```c
void calculate(void)
{
    int x = 10;
    int y = 20;
    // calculations
}
```

x and y are local variables. They usually need temporary storage while calculate() is executing. The stack provides that temporary storage. When the function finishes, that stack space can be released, meaning the SP moves back where it was before. RAM is not erased. 

Stack is in RAM because stack contents change constantly. Whenever a function is called, the stack grows and whenever the function returns, the Stack shrinks. We need fast read / write memory and hence it is in RAM and not in Flash.

The Stack grows downwards, suppose RAM ends at `0x20020000`. The initial stack pointer can be `SP = 0x20020000`. Then when stack space is needed `0x20020000 -> 0x2001FFFC -> 0x2001FFF8 -> 0x2001FFF4` and so on. So the address gets smaller, It moves downwards from Higher address to lower address. Now connecting this to our linker script, the symbol we were just learning ```ld _estack = ORIGIN(RAM) + LENGTH(RAM);```

For our simplified STM32F407VG memory:

```text
ORIGIN(RAM) = 0x20000000
LENGTH(RAM) = 128 KB 
128 KB = 128 × 1024 = 0x20000
       
_estack = ORIGIN(RAM) + LENGTH(RAM)

_estack = 0x20000000 + 0x20000 = 0x20020000

```

Therefore `_end of RAM = 0x20020000`

So `_estack = 0x20020000`. That value becomes the initial Main Stack Pointer value MSP.

Then the Vector Table uses it : At the beginning of Flash: `0x08000000` we have the Vector Table. Its first word contains `0x20020000` which is our initial stack pointer value. Therefore MSP = 0x20020000. 

```text
Linker Script
     |
     | calculates
     ↓
_estack = 0x20020000
     |
     ↓
Vector Table
     |
     | first word
     ↓
0x20020000
     |
     ↓
Cortex-M4
     |
     ↓
MSP = 0x20020000
And only after this does the CPU move on to the second Vector Table entry which is Reset_Handler address which goes into the PC.
```

**The first Vector Table entry contains the initial value that the CPU loads into the MSP. The Vector Table contains a value. The MSP is the CPU register receiving that value**

**_estack is not itself the MSP.** `_estack` is a linker-defined symbol whose value is the initial/top address chosen for the stack, typically the end of the RAM region (Origin + Length). Whereas MSP is the Main Stack Pointer Cortex-M4 core register, which holds the current address of the top of the active Stack. The address in MSP changes as the stack grows and shrinks, however `_estack` normally stays fixed. 
