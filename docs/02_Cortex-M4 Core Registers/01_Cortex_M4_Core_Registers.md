# Cortex-M4 Core Registers
```text
Cortex-M4 Core Registers
│
├── R0–R12  (General purpose registers)  
│    
├── R13 (SP)        
│     │__ MSP (Main Stack Pointer)
│.    │__ PSP (Process Stack Pointer)
│
├── R14 (LR)  (Link Register)
│       
├── R15 (PC)  (Program Counter)    
│
├── xPSR             Program Status Register
│   │
│   ├── APSR         Arithemetic operations (flags) ← N, Z, C, V, Q
│   ├── EPSR         Execution State ← T + ICI/IT
│   └── IPSR         exception number
│
│
├── CONTROL 
│      │___  nPRIV   Privileged / Unprivileged in Thread mode
│      │___  SPSEL   MSP / PSP selection in Thread mode
│      │___  FPCA    Floating-point context active
│
│
├── Special masking registers
│        │______   PRIMASK         Mask most configurable exceptions/interrupts
│        │______   FAULTMASK       Mask all exceptions except NMI
│        │______   BASEPRI         Mask configurable exceptions/interrupts
│                                   at and below a priority threshold
│
│__ FPSCR   Floating Point Status and Control Register
```
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
    ├-- R13 -> SP
    ├── R14 -> LR
    ├── R15 -> PC
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

R0–R12 are 32-bit general-purpose registers of the Cortex-M4 core. They are used by the CPU for things such as:

•	temporary values 
•	arithmetic/logic operations 
•	function arguments and return values 
•	addresses/pointers 
•	local variables, when the compiler chooses to keep them in registers 

For example:
```c
int a = 5;
int b = 3;
int c = a + b;
```
R0–R3 have special roles in the ARM procedure-call convention. R0–R3 → commonly used for function arguments and return values  The compiler may place a and b in registers such as R0 and R1 and perform the addition using the ALU. However, C does not guarantee which registers are used. Register allocation is determined by the compiler and the ARM calling convention.

Registers are much faster for the CPU to access than RAM, so the compiler tries to keep frequently used values in registers when practical.

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

So MSP and PSP are not two views of the same 32-bit physical register. They are two independent 32-bit stack-pointer registers. Then the processor chooses which one represents the active SP depending on the current execution mode.

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


### What happens to MSP when a function is called? 

Lets consider the below code Snippet:
```c
int main(void)
{
    function();
}

void function(void)
{
    int x = 10;
    int y = 20;
}
```

Here,
-   x and y are local variables which are commonly stored in the stack in RAM
-   MSP points to the current top of the stack

**Before function() is called:** Lets imagine
```text
RAM

0x20020000  ← MSP
     │
     │
     │   free stack space
     │
     ▼
lower addresses
```
The function() needs stack space. When function() starts executing, it may need space for things such as:
```text
function()
│
├── local variables
├── saved registers
└── other temporary information
```
The CPU/compiler reserves some space on the stack. Because the stack grows downward:

```text
Before:

0x20020000  ← MSP
     │
     │
     ▼


After reserving stack space:

0x20020000
     │
     │
0x2001FFF0  ← MSP
     │
     │
     ▼
Before: MSP = 0x20020000

After: MSP = 0x2001FFF0

MSP moves downward when stack space is allocated.
```
The area allocated for a particular function is called its **stack frame.**

Example:

```text
RAM

0x20020000
────────────────
   previous stack
────────────────
0x2001FFF0  ← MSP
│
│ function()'s
│ stack frame
│
│ x
│ y
│ saved information
│
────────────────

So, Stack
     └── function()'s stack frame
     
When function() finishes, that stack frame is released. MSP moves back upward.
```text
Before function: MSP = 0x20020000
function starts: MSP = 0x2001FFF0
function returns: MSP = 0x20020000
```

**The function's generated code may need a stack frame, and the compiler generates instructions that adjust the stack pointer to reserve that space.**. Not every function call requires a stack frame. 

For example,
```c
int add(int a, int b)
{
    return a + b;
}
```
The compiler may be able to perform the entire function using registers without allocating local stack space.

Now, lets quickly calculate, how much does the MSP move. Suppose, Before function MSP = `0x20010000`. Lets consider a function which needs 16 bytes of stack space. Since the stack grows downward, what will the MSP become? What address will it point when the function is returned.

Before function MSP = 0x20010000 (hex))
Required Stack space = 16 bytes (decimal) = 0x10  (hex)
MSP = 0x20010000 - 0x 10 = 0x2000FFF0 (Stack grows lower). MSP moved by 16 bytes, not 16 bits. 

After function :
MSP = 0x20010000 + 0x 10 = 0x20010010 (Stack shrinks)
            
Now that, we understood, how MSP moves as the Stack grows and shrinks as the function call and returns, lets see ***how the CPU actually changes the MSP.***

We already knew that MSP is one of the CPU's core register. That means the CPU currently has the number 0x20010000 stored inside its MSP register. RAM and MSP are two different things, MSP is inside the CPU and not a memory location in RAM. RAM doesnt change MSP. Only a CPU instruction can changes MSP. For example, there are instructions that explicitly modify the Stack pointer. 

Example : ```asm SUB SP, SP, #4``` meaning to take the current value of SP, subtract 4, and put the result back into SP.  If SP = `SP = 0x20010000`. Then the CPU performs `0x20010000 - 4 = 0x2000FFFC` and stores that result back into the SP register. This operation happens inside the Cortex-M4 CPU, using its internal hardware (ALU). We haven't talked about what is stored in that RAM location yet. Changing MSP and storing something in the stack are two separate operations. **MSP is a CPU register -> ALU calculates the new value -> new value goes back into MSP.** The CPU's stack pointer is currently pointing at RAM address `0x2000FFFC` (the address of the current stack location). Incase, if we want to store something ((suppose R4 = 25) in this particular location in RAM, then we have to say to store the value 25 at the RAM address currently held by MSP. So basically MSP = `0x2000FFFC`which is an address of a Stack in RAM and in this address , we want to store the value 25. 

So instead of manually doing: 
1. Move MSP downward
2. Store a register value at the new address, 
the Cortex-M4 provides an instruction that does the stack operation for us ```asm PUSH {R4}```.

We just learnt that the stack grows downward, and MSP moves toward lower addresses when we put something on the stack. The ARM Cortex-M4 provides instructions specifically for this. One of them is **PUSH**, which basically means to save one or more register values onto the stack.

Example: 

Suppose:`MSP = 0x20010000`. And we execute: `PUSH {R4}`. We're asking the CPU to save the current value of R4 on the stack and MSP updated to point to it. Since R4 is 32 bits (4 bytes). Therefore, the stack needs 4 bytes. The MSP moves downward : 

Before: `0x20010000`  ← MSP
After: `0x2000FFFC`← MSP

Thats because , `0x20010000 - 4 = 0x2000FFFC`. And the value of R4 is stored there:
```text
RAM
0x20010000
──────────────
0x2000FFFC  ← MSP
│
│ R4 value
│
──────────────
```

So conceptually:

```text
PUSH {R4}

R4
 │
 │  copy value
 ▼
Stack
 │
 ▼
MSP moves downward

Before:
Before:

CPU
┌──────────────┐
│ R4  = 25     │
│ MSP = 0x20010000
└──────────────┘

RAM
0x20010000   ?
0x2000FFFC   ?

After the stack operation:

CPU
┌──────────────┐
│ R4  = 25     │
│ MSP = 0x2000FFFC
└──────────────┘
              │
              ▼
RAM
0x2000FFFC   25

```
So two things have happened:

-   MSP moved to a new stack address.
-   The value of R4 was stored in RAM at that stack location.

MSP -> tells CPU WHERE the stack location is and R4  -> tells CPU WHAT value to save. `PUSH` combines those ideas into one stack instruction.
But ***which happens first conceptually— MSP changing or R4 being written to RAM—and why the stack uses the address `0x2000FFFC` rather than `0x20010000`.

We have: R4  = 25 and MSP = 0x20010000. PUSH {R4}

The question is Why does the value end up at 0x2000FFFC instead of 0x20010000? 

1. Thats because, Our Cortex M4 stack is a descending stack. When we need space for something, then the stack needs to move towards the lower address. R4 is a 32 bit register (4 bytes). Therefore, we need 4 bytes of stack space. Starting from `MSP = 0x20010000` we need to move down by 4 bytes, so the new stack location is `0x2000FFFC`

2. Conceptually, the CPU changes from MSP = `0x20010000` to MSP = `0x2000FFFC`. Now MSP points to the newly allocated stack space.

3. R4 gets stored there. R4 contains 25.  So CPU stores RAM[0x2000FFFC] = 25. So finially, MSP moved from `0x20010000` to MSP = `0x2000FFFC` and 25 is saved there.

```
1. Make room for R4
       ↓
2. Move MSP downward by 4 bytes
       ↓
3. Store R4 at the new stack location
```
```text
Before PUSH:

MSP
 ↓
0x20010000

After PUSH: 
PUSH {R4}
     │
     ├── stack needs 4 bytes
     ↓
MSP moves down
     ↓
0x2000FFFC -- MSP
     │
     └── R4 (25) stored here
     
MSP = 0x2000FFFC
RAM[0x2000FFFC] = 25

```
R4 is a 32-bit general-purpose register. The CPU doesn't know whether the bits represent 25 or 1000 or an address or a pointer or part of a calculation or even characters / flags / other data. R4 = 0x00000019 represent decimal 25. But the CPU simply sees as 
`00000000 00000000 00000000 00011001`


The ```asm PUSH{R4}``` and `R4 = 25`is not about storing the value 25 directly in a RAM location. The reason we PUSH {R4} is not simply because RAM needs the value 25. It's because we may need to preserve the value that was already inside R4 while another function temporarily uses R4. For example, a C variable could eventually result in a memory store: ```c int x = 25; ``` and x may live in RAM. That's normal data storage. But that's NOT the main purpose of PUSH. Here's the situation where PUSH becomes important.

Imagine we're running:

```c
main()
{
    function();
}
```
Before calling function(), suppose the CPU has R4 = 25 . And 25 is important to main(). Now function() needs to use R4 for its own work. It might do R4 = 100. Now the original R4 = 25 is gone. That's a problem. So we have to save the old value of this register somewhere in Stack, before using R4 for something else. Hence we do ```asm PUSH{R4}```. Now the function can safely change R4 R4 = 100 since the old value is preserved in Stack. Later, as the function finishes, we can restiore it using ```asm POP{R4}. The saved value comes back to R4. R4 = 25. So after the function returns, the caller gets its original R4 value back.

Here, we are using the stack as it gives functions a temporary, organized place to save CPU state. It's not primarily about storing application data like ```c counter = 25```. It's used when we neeed to temporarily give up this register, but we don't want to lose the value that was already there.


```text
R4 = important old value
       │
       │ PUSH
       ▼
Stack (Old value of R4 preserved in Stack)
       │
       │ function uses R4 
       ▼
R4 = temporary value of new function
       │
       │ POP (As new function returns)
       ▼
R4 = original value of old function

```

**What if we push multiple registers?**
Suppose: 
```text
MSP = 0x20010000
R4 = 25
R5 = 100
R6 = 0x12345678
```
when we execute ```asm PUSH {R4, R5, R6}` four 32-bit registers. And each register is 4 bytes. Therefore 3 registers × 4 bytes = 12 bytes

So:
Before: MSP = `0x20010000`
After: MSP = `0x2000FFF4`
because: 0x20010000 - 0xC = `0x2000FFF4`

Memory conceptually becomes:

```text
RAM

0x20010000
──────────────
     ...
──────────────
0x2000FFFC
│ R4
──────────────
│ R5
──────────────
│ R6
──────────────
│ LR
──────────────
     ↑
    MSP
0x2000FFF0

```
When we study the ARM instruction behavior, lets deep dive more precisely into the exact oder of the registers in the memory. 

### What does POP actually do?

The goal is to **take the saved value from the stack and put it back into R4 and the stack space that was being used must be released.** So MSP needs to move back upward.

Before POP: `MSP = 0x2000FFFC`. There is one 32-bit value on the stack: 0x2000FFFC → 25. The register is 4 bytes. So when we remove it, `0x2000FFFC + 4 -> 0x20010000`.

Therefore After POP:
```text
R4  = 25
MSP = 0x20010000```

Visually:

```text

             Before POP

CPU                    RAM
┌──────────────┐
│ R4 = ???     │
│              │
│ MSP ───────────────► 0x2000FFFC
└──────────────┘       ┌─────────┐
                       │   25    │
                       └─────────┘
                       
                       
After:
             After POP

CPU
┌──────────────┐
│ R4 = 25      │
│              │
│ MSP =        │
│ 0x20010000   │
└──────────────┘

RAM
0x2000FFFC → 25   ← old stack data
```
**The 25 doesn't necessarily get physically erased from RAM. The stack space is considered released because MSP moved away from it.**

### PUSH vs POP:

| Operation   | MSP       | RAM                  |
| ----------- | --------- | -------------------- |
| `PUSH {R4}` | decreases | R4 saved             |
| `POP {R4}`  | increases | value loaded into R4 |

### What about multiple registers?

Suppose: R4 = 25, R5 = 100, R6 = 200 when we do ```asm PUSH {R4, R5, R6}``` We save 3 × 4 bytes = 12 bytes. Since we do PUSH, the address grows from higher address to lower address, ie MSP moves down from `0x20010000`to `0x2000FFF4`. ```asm POP{R4,R5,R6}``` restores the registers and releases those 12 bytes, MSP goes back (up) from `0x2000FFF4`to `0x20010000`. The old bits may still physically remain in RAM until something overwrites them.

### PSP — Process Stack Pointer

We already know:
```text
R13
 │
 ├── MSP
 └── PSP
```

So MSP and PSP are not two separate physical registers like R4 and R5. They are two stack-pointer registers associated with R13.
The Cortex-M4 can use either one as the active stack pointer depending on the processor's mode. 

#### Why do we need a second stack pointer?

The main reason is that we want to separate:

1. The stack used by the operating system / privileged code from 
2. The stack used by an application/task

For example, imagine we're running FreeRTOS. There may be several tasks like Task A, Task B, Task C.  And each task needs its own stack.We don't want every task to use the same stack.

```text
RAM

Task A stack
──────────────

Task B stack
──────────────

Task C stack
──────────────
```
The **PSP** can point to the stack belonging to the currently running task. Meanwhile, the **MSP** can be reserved for the system/kernel/exception handling.

The Cortex-M4 has different processor modes, and the actual stack-pointer selection depends on the CONTROL register and exception handling.

#### Example :

Suppose RAM contains:

```text
RAM

0x20020000
──────────────
        ↑
      MSP
──────────────
System stack
──────────────


Task stack
──────────────
        ↑
      PSP
──────────────

We might have:

MSP = 0x20020000
PSP = 0x20018000

Now the processor can have:

MSP
 ↓
0x20020000

for the main/handler stack, while:

PSP
 ↓
0x20018000

points into a task's stack.

```
**MSP and PSP are both capable of pointing into RAM. They are simply two different stack-pointer values.**

For example:
```text
MSP = 0x20020000
PSP = 0x20018000
```

Some question could arise like **Which one is the CPU actually using right now?  or Does the CPU use both stacks simultaneously for normal instructions? or Does it use one active stack pointer at a time?**

For normal stack operations, the CPU uses one active stack pointer at a time.
So if `MSP = 0x20020000`and `PSP = 0x20018000`, both registers contain valid addresses, but the CPU doesn't normally do both 
`PUSH → MSP` and `PUSH → PSP`at the same time. Instead, it has to choose which stack pointer is active.

```text
              Cortex-M4
                  │
          Which stack to use?
                  │
          ┌───────┴───────┐
          ▼               ▼
        MSP             PSP
    0x20020000       0x20018000
    
```
***Only one is selected as the current stack pointer for Thread mode. The selection is controlled by the CONTROL register. There is one important exception, Handler mode always uses MSP.*** We will study more about later.

```text
Thread mode
    │
    ├── MSP  ← can use this
    └── PSP  ← or this

Handler mode
    │
    └── MSP  ← always
```

### R14 Link Register (LR)

Suppose a function main() calls another function:

```c
main()
{
     other_function();
}
```

The CPU needs some way to remember where to return in main(), once `other_function()` finishes. So, the link register (R14) holds the return address.

Let's first understand the ARM instruction that creates this behavior. The important instruction is ```asm BL other_function``` Here, `BL`means `Branch with Link`. There are two things happening in a BL instruction.

-    1. Save a return address in LR / R14
-    2. Branch to the target function (other_function)

When the processor calls other_function(), it needs to remember the address where execution should continue after other_function() returns. So that information will be stored in Link register or R14.

The ARM instruction used for a normal function call is ```asm BL other_function```.  Then, when other_function() finishes, it can use LR to get back to the caller. A common return instruction is ```asm BX LR```.


Suppose,
```c
main()
{
    other_function();
}

other_function()
{
    another_function();
}
```
So the call chain is main() -> other_function() -> another_function(). So when other_function() calls another_function(), what does happen to the value currently stored in LR? B) It gets overwritten with the return information for other_function(). So the original return information for main() is lost unless other_function() saves it somewhere first. And this is where the stack becomes important. So if other_function() knows that it is going to call another_function(), it should save its current LR before executing ```asm BL another_function```. 

The sequence is ,

```text

other_function()
        │
        │ LR = return address to main()
        │
        ↓
    PUSH {LR}
        │
        │ LR is safely saved
        ↓
BL another_function
        │
        │ LR gets overwritten
        ↓
another_function()
        │
        ↓
return
        │
        ↓
    POP {LR}
        │
        │ original LR restored
        ↓
    BX LR
        │
        ↓
main()
```

```text
LR
│
└── register → temporarily holds return information

Stack
│
└── memory → can save that LR when we need the register for another call
```

Suppose we have LR = 0x08000108 and  SP = 0x20001000. And the instruction is ```asm PUSH {LR}```.  R13 = SP, So SP tells us where the stack currently is.When we execute ```asm PUSH {LR}``` what would happen conceptually?  The value 0x08000108 is stored in RAM on the stack. After ```asm PUSH {LR}``` will the SP value change? Yes. It should move towards the lower address as we pushed something in stack. The Cortex-M stack grows downward, toward lower addresses.

```asm POP{LR}```takes the value currently at the top of the stack and put it back into LR.


So the complete chain is:
```text

```text
main()
 │
 │ BL other_function
 │
 ↓
other_function()
 │
 │ PUSH {LR}              // save return information to main()
 │
 │ BL another_function    // LR is overwritten
 │
 ↓
another_function()
 │
 │ return
 ↓
other_function()
 │
 │ POP {LR}               // restore return information to main()
 │
 │ BX LR                 // return to main()
 ↓
main()
```

The key idea is:

```text
main()
   │
   │ BL other_function
   ↓
other_function()

At this point:
LR = "return to main()"

   │
   │ PUSH {LR}
   ↓
LR is safely saved on the stack

   │
   │ BL another_function
   ↓
LR = "return to other_function()"

   │
   ↓
another_function() finishes

   │
   │ POP {LR}
   ↓
LR = "return to main()"   ← restored

   │
   │ BX LR
   ↓
main() continues
```

So the **reason for `PUSH {LR}`** is specifically:

> `other_function()` needs to call another function, and that call will overwrite its LR. Therefore, it saves its original LR on the stack first.

***LR is not always an ordinary function return address. When an exception occurs, LR receives a special value called an EXC_RETURN value. That value tells the processor important information about how to return from the exception.
****

## R15 — Program Counter

**The PC (Program Counter) tells the processor where it is executing in the program.**
Imagine our program instructions are stored in Flash:

```text
Flash

0x08000000    instruction 1
0x08000004    instruction 2
0x08000008    instruction 3
0x0800000C    instruction 4
0x08000010    instruction 5
```
If the processor is currently executing the instruction at `0x08000008`, then conceptually PC = `0x08000008`. ie, PC contains the address associated with the current instruction execution flow.

For example, Suppose if PC = `0x08000020`, and the next instruction to execute is located at `0x08000024`, what is the processor going to do next? It would execute the instruction at 0x08000024, because the instructions are stored in the program's memory, and the PC controls the instruction-fetch flow.

On Cortex-M4, the exact relationship between the visible PC value and the instruction currently being executed has pipeline/architectural details. We don't need that complication yet. For now, lets underdstand that, PC tells the CPU where the instruction execution flow is going.

### PC and BL :

Earlier, when a function call is made, we learnt about `Branch with Link (BL)` and it does two things.
-    1. Save return information  in Link Register LR / R14 (LR -> remembers where to return)
-    2. Branch to the function - PC changes with new function address (tells CPU where to execute)

Suppose, LR = 0x08000108, PC = 0x08001000 and when we execute ```asm BX LR```, what would be the basic effect?
PC changes to the address contained in LR because the processor uses the value in LR as the branch target.

```text
LR = 0x08000108
        │
        │ BX LR
        ▼
PC = 0x08000108
So:

LR -> tells the CPU WHERE TO RETURN
PC -> tells the CPU WHERE TO EXECUTE

```
```c
main()
{
    other_function();
}
```
Conceptually:
```text
main()
  │
  │ BL other_function
  │
  ├────-> LR = return information
  │
  └────-> PC = other_function()
             │
             │ execute
             │
             │ BX LR
             ↓
          PC = return address
             │
             ↓
        main() continues
        
And when there is a nested call:

main()
  │
  │ BL other_function
  ↓
other_function()
  │
  │ PUSH {LR}
  │
  │ BL another_function
  ↓
another_function()
  │
  │ return
  ↓
other_function()
  │
  │ POP {LR}
  │
  │ BX LR
  ↓
main()

R14 -> LR -> return information
R15 -> PC -> current instruction flow

```


### xPSR [Z, N, C, V]

The xPSR (Program Status Register) is a 32-bit internal register inside the ARM Cortex-M4 core that tracks the status and health of our program. 

While general-purpose registers (R0–R12) hold our variable data, the xPSR holds status flags that tells what just happened after an instruction executed (e.g., "Was the result negative?", "Did it overflow?").

The "x" stands for three combined status sub-registers:
-    APSR (Application PSR): Holds ALU condition flags (Negative, Zero, Carry, Overflow).
-    IPSR (Interrupt PSR): Holds the current exception/interrupt number being processed.
-    EPSR (Execution PSR): Tracks execution states like Thumb mode.

But when we refer to xPSR, we're looking at a combined view of these status fields. These are not 3 physically separate registers
```text
                 xPSR
┌──────────────────────────────────────────────┐
│                  32 bits                     │
├───────────────┬───────────────┬──────────────┤
│     APSR      │     IPSR      │     EPSR     │
│ condition     │ exception     │ execution    │
│ flags         │ number        │ state        │
└───────────────┴───────────────┴──────────────┘
This drawing is conceptually useful, not the actual bit layout. The three parts overlap in the 32-bit xPSR.
```
### The Complete 32-Bit `xPSR` Bit Layout

So which bits belong to which part among APSR, IPSR; EPSR?

```text
 Bit:   31  30  29  28  27  26   25    24                16  15          10   9   8                  0
      ┌───┬───┬───┬───┬───┬───┬───┬───┬──────────────────┬──────────────┬───┬──────────────────┐
      │ N │ Z │ C │ V │ Q │ ICI / IT  │ T │     Reserved     │   ICI / IT   │ 0 │   ISR_NUMBER     │
      └───┴───┴───┴───┴───┴───┴───┴───┴──────────────────┴──────────────┴───┴──────────────────┘
View: │<───── APSR ──────>│<── EPSR ─>│                      │<── EPSR ────>│   │<──── IPSR ──────>│
```

### Bit-by-Bit Mapping Table

| Bit Range | Field Name | View | Description |
| --- | --- | --- | --- |
| **Bit 31** | **N** (Negative) | **APSR** | `1` if result is negative, `0` if positive/zero |
| **Bit 30** | **Z** (Zero) | **APSR** | `1` if result is zero, `0` if non-zero |
| **Bit 29** | **C** (Carry / Borrow) | **APSR** | `1` on unsigned addition carry / no borrow on subtraction |
| **Bit 28** | **V** (Overflow) | **APSR** | `1` if signed operation exceeded valid bounds |
| **Bit 27** | **Q** (Sticky Saturation) | **APSR** | `1` if SSAT/DSAT instruction saturated (DSP extensions) |
| **Bits 26:25** | **ICI / IT** (Top bits) | **EPSR** | Holds top bits of If-Then state or multi-cycle LDM/STM state |
| **Bit 24** | **T** (Thumb State) | **EPSR** | Always `1` on Cortex-M (Cortex-M only executes Thumb-2 instructions) |
| **Bits 23:16** | *Reserved* | — | Unused / Reserved by ARM (reads as 0) |
| **Bits 15:10** | **ICI / IT** (Low bits) | **EPSR** | Holds low bits of If-Then state or multi-cycle LDM/STM state |
| **Bit 9** | *Reserved* | — | Unused / Reserved by ARM (reads as 0) |
| **Bits 8:0** | **ISR_NUMBER** | **IPSR** | Active interrupt/exception ID (`0` = Thread mode, `15` = SysTick, `16+` = IRQ) |

1. **APSR is strictly the top 5 bits `[31:27]`.** These are the only bits modified by `ADDS`, `SUBS`, `CMP`, etc.
2. **EPSR is split across two non-contiguous chunks:** Bits `[26:24]` and bits `[15:10]`. It cannot be read directly via software (reading EPSR via `MRS` yields all zeros for security/architecture reasons).
3. **IPSR is strictly the lower 9 bits `[8:0]`.** It stores the exact exception number currently being handled by the CPU hardware.

The important thing is that APSR, IPSR and EPSR aren't simply three consecutive chunks of bits.
```
Then how can ARM call them three "registers"? The processor has one underlying status register representation, but ARM provides different views of selected bits. Think of it like a 32-bit box:

```text
                  xPSR
        ┌─────────────────────┐
        │ 31              0   │
        └─────────────────────┘
             ↑      ↑      ↑
             │      │      │
           APSR    IPSR    EPSR
           view    view    view
```
Each named register exposes the bits relevant to that particular purpose.
- APSR : Showsthe application-status bits.
- IPSR : Shows the exception-status bits.
- EPSR : Shows the execution-status bits.
- xPSR : Shows the combined status.

Who actually maintains these bits? This is the CPU hardware, not our C program.
For example, suppose we execute: `ADDS R0, R1, R2`. The ALU performs the addition.
The processor's hardware determines whether N = 1 or 0, Z = 1 or 0, C = 1 or 0, V = 1 or 0 and updates the corresponding APSR bits. Our software doesn't manually calculate and write those flags.

Similarly, when an exception occurs:
```text
Normal code
     │
     │ interrupt
     ▼
Exception entry
     │
     ▼
IPSR gets exception number

The Cortex-M hardware updates the exception state. And EPSR contains execution-state information maintained by the processor.
```

Don't think APSR, IPSR and EPSR are three registers sitting somewhere in memory. They aren't normal memory-mapped registers like `GPIOA_ODR`,
`GPIOA_IDR`. We can't do `*(uint32_t *)some_address = ...` to access xPSR. These are special CPU registers. The processor provides instructions and mechanisms to access certain parts of them.

For example, in assembly we can use `MRS R0, APSR` to read the APSR view into R0 or `MRS R0, IPSR` to read the IPSR view, or `MRS R0, xPSR`
to obtain the combined view.

We need to be careful here because some bits are shared/overlapping between the views.
For example:
```text
                 32-bit xPSR
31                         0
┌───────────────────────────┐
│ N Z C V Q      ...        │
└───────────────────────────┘
 ↑
 APSR
```
while IPSR is concerned with the exception number field in the lower bits. EPSR has execution-state fields in the middle/lower portions.
So the three views are not three independent physical 32-bit values. That's why ARM documentation often shows something like:
```text
              xPSR
        ┌──────────────┐
        │ APSR │ EPSR │
        │      │      │
        │      │ IPSR │
        └──────────────┘
with fields positioned according to their actual bit locations.
```
```text
                 CPU status
                     │
                    xPSR
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     APSR           IPSR          EPSR
   "flags"       "exception"    "execution"
And the CPU hardware is responsible for maintaining these fields.
```

```text
APSR
 │
 ├── N -> Negative  (sign/negative result indicator)   
 ├── Z -> Zero       (result is zero)  
 ├── C -> Carry   (unsigned arithmetic carry) 
 └── V -> Overflow  
```
```text
CMP
 │
 ├── calculates comparison
 │
 └── updates xPSR flags
          │
          ▼
       Z, N, C, V
          │
          ▼
Conditional branch
```

Think of xPSR as a status register. While the CPU executes instructions, certain operations produce information about their result. 

#### Z (Zero Flag)

For example, suppose: ```text R0 = 10, R1 = 10```. The CPU compares them ```asm CMP R0, R1```. The result is ```text R0 - R1 = 0```. The CPU can record the result as zero. That information is represented by a flag in the processor's status register. (Z = Zero flag). 

So conceptually,
```text
R0 = 10
R1 = 10

CMP R0, R1 
     │
     ▼
10 - 10 = 0
     │
     ▼
Z = 1
     │
     ▼
BEQ equal
     │
     ▼
Branch to "equal"

```
**The Zero flag becomes 1 when the result of the comparison/subtraction is zero.** And CMP does not change R0 or R1. It performs the comparison and updates the condition flags.
Then a conditional branch can use that information ```asm BEQ somewhere``` ***BEQ means Branch if Equal*** and it can determine whether to branch based on the Zero flag.

CMP produces the status -> xPSR holds the flags -> BEQ checks the relevant flag -> branch happens.

#### N (Negative Flag)

Suppose:
```tex
R0 = 10
R1 = 20
and CMP R0, R1
10 - 20 = -10

Now, 
Z = 0   ← result is NOT zero
N = 1   ← result is negative, hence 1

```

Suppose:

R0 = 20 R1 = 10
CMP R0, R1 
20 - 10 = +10

Therefore:
N = 0     result is not negative
Z = 0     result is not zero

Notice that CMP is essentially doing a subtraction for the purpose of setting flags:

```text
CMP R0, R1
     ↓
   R0 - R1
     ↓
  ┌─────────────┐
  │ N Z C V ... │
  └─────────────┘
       xPSR
```
It does not put 20 - 10 into another register. So after CMP R0, R1, we still have R0 = 20, R1 = 10 but the status information has been updated.

#### C — Carry Flag

The Carry flag is mainly about whether an unsigned arithmetic operation produces a carry out of the most significant bit.
Let's start with a very simple unsigned addition.

Suppose the CPU has R0 = 5, R1 = 3
and executes ADD R0, R1
Conceptually: 5 + 3 = 8

There is no carry out from the most significant bit. So, for an addition that updates the flags `C = 0`

Think about an 8-bit example first:

```text

1111 1111
+      1
────────
10000 0000       
```
**But an 8-bit register can only hold 0000 0000 and the extra 1 goes beyond the 8 bits. That extra bit is the carry out**.

So conceptually:
```text
11111111
+       1
─────────
1 00000000
↑
carry out

Therefore: C = 1

```
Lets consider an another example:

```text

     1111 1111
+    0000 0001
     ─────────
     1 0000 0000
     ↑
     carry out
     
```

Because we're considering an 8-bit result, only the lower 8 bits (0000 0000) fit in the register. Result = 00000000
Carry  = 1. So `C = 1`

Now let's connect this back to Cortex-M4:
The Cortex-M4 registers are 32 bits, not 8 bits.

So the same idea applies at bit 31:

```text

  11111111111111111111111111111111
+                                1
-----------------------------------
1 00000000000000000000000000000000
↑
carry out of bit 31

```
The 32-bit register receives 00000000000000000000000000000000 while the extra carry is reflected in C = 1.

Example :

In the 32-bit case: 0xFFFFFFFF + 1. 
0xFFFFFFFF is a 32-bit value because there are 8 hexadecimal digits F F F F F F F F. Each hex digit = 4 bits(F = 1111)
 
Therefore : 0xFFFFFFFF = 1111 1111 1111 1111 1111 1111 1111 1111. That's all 32 bits set to 1.

```text

  1111 1111 1111 1111 1111 1111 1111 1111
+ 0000 0000 0000 0000 0000 0000 0000 0001
-------------------------------------------
1 0000 0000 0000 0000 0000 0000 0000 0000   (33 bits) 
↑
extra carry

```

The extra leftmost 1 (33rd bit) doesn't fit inside R0. The processor records it in the Carry flag (C).So Carry C = 1.
R0 can fit 0000 0000 0000 0000 0000 0000 0000 0000 which is 0x00000000.  So Z = 1.

```text

C = 1
└── There was an extra carry beyond bit 31

Z = 1
└── The 32-bit result is zero

```
**Two flags can be 1 at the same time.**

**C behaves differently for addition and subtraction on ARM/Cortex-M4. The key is to understand what the CPU is actually telling us**

### Carry Flag C — Addition vs Subtraction

The simplest statement to remember is:

-    Addition: 
     1. C = 1 means a carry came out of the most significant bit MSB.

-    Subtraction: 

     1.    C = 1 means NO borrow was needed.
     2.   C = 0 -> borrow occurred.
               
**1. Addition — C = 1 means carry out**

Take 8-bit arithmetic:

```c
  1111 1111
+ 0000 0001
-----------
1 0000 0000
↑
carry out           We have only 8 bits available. The result that fits in the 8-bit register is 0000 0000
But there is an extra 1 outside bit 7. Therefore C = 1

```
So for addition:

```text
                 ADDITION
                    │
                    ↓
        Did a carry leave bit 7?
              /             \
            YES              NO
             ↓                ↓
           C = 1            C = 0

For a 32-bit Cortex-M4, replace bit 7 with bit 31.

```
**2. Subtraction — C = 1 means NO borrow**

Now consider `5 - 3`.  In normal decimal arithmetic `5 - 3 = 2`. Did we need to borrow? No. Therefore, on ARM `C = 1`.
If instead we calculate `3 - 5`, we cannot subtract 5 from 3 without borrowing. So `3 - 5` requires a borrow. Therefore `C = 0`

So for subtraction:

```text
                 SUBTRACTION
                      │
                      ↓
             Was a borrow needed?
                /          \
              NO            YES
               ↓             ↓
             C = 1         C = 0

```

**Why does ARM define it this way?**

This becomes much clearer when we look at unsigned numbers. Suppose we're doing `5 - 3`, Since `5 ≥ 3` the subtraction is valid without going below zero. So ARM says `C = 1`

Now `3 - 5`, since `3 < 5`, the unsigned subtraction needs a borrow. ARM says `C = 0`.

Therefore, for unsigned subtraction, we can think:
-    C = 1 -> the subtraction did not need to borrow.
-    C = 0 -> the subtraction needed a borrow.

Lets see how it does work with binary:

```text

Take `5 - 3`: 
  0000 0101
- 0000 0011
------------
  0000 0010
  
```
No borrowing was necessary. Therefore `C = 1`. Now, lets check for `3 -5`,

```text

  0000 0011
- 0000 0101

We can't subtract the lower bits without borrowing from a higher position. So C = 0

```
This is especially important for unsigned comparisons. For example `CMP R0, R1` is essentially a subtraction `R0 - R1`

If `R0 >= R1` then there is no unsigned borrow `C = 1`
If `R0 < R1` then a borrow occurs `C = 0`

That's why ARM conditional branches include:
BCS / BHS → C = 1
BCC / BLO → C = 0

Lets learn about it later. 

| Operation       | `C = 1` means      | `C = 0` means       |
| --------------- | ------------------ | ------------------- |
| **Addition**    | Carry out occurred | No carry out        |
| **Subtraction** | **No borrow**      | **Borrow occurred** |

**Flag C is primarily about unsigned arithmetic. Flag V is about signed overflow.**

#### V - (Overflow Flag)

While `C (Carry Flag) is about unsigned arithmetic, Overflow Flag (V) is about signed arithmetic.

Before understanding what an Overflow V flag represents, lets quickly understand about signed vs unsigned.

Lets consider a 8 bit, 10000000. It is 128 in decimal. If we interpret it as unsigned then `10000000₂ = 128``. Now here's the key, the bits themselves have not changed. `10000000` it's still exactly the same 8 bits. But a computer can interpret those bits in different ways depending on whether we're treating the value as unsigned or signed.

```text

Same bits
   │
   ├── interpreted as unsigned -> 128
   │
   └── interpreted as signed   ->something else
   
```
When we say an 8-bit number is signed, we want it to represent both: positive numbers + negative numbers. So instead of the unsigned range 0  to 255, an 8-bit signed system uses -128 to +127. And here is the interesting part 10000000 is interpreted as:   128     ← unsigned, -128    ← signed. The bits did not change. Only the interpretation changed.

So now we have an important boundary:
8-bit unsigned:
```text
00000000 ->  0
        ...
01111111 -> 127
10000000 -> 128
        ...
11111111 -> 255
```
And for 8-bit signed numbers, the range is -128 -> +127

Now what happens if we add 1?
```text
  0111 1111
+ 0000 0001
-----------
  1000 0000
```
The bits changed from `01111111`to `10000000`. As unsigned, that's `128`. But as signed, `10000000` represents `-128`. So if we were doing signed arithmetic `+127 + 1` we expected `+128` but the 8-bit signed system cannot represent `+128`.
Instead, the bit pattern becomes `10000000 = -128`

That is the first concrete example of signed overflow. And that is what the V flag is designed to tell the CPU about.

For an 8-bit signed number, what is the largest positive value it can represent? +127

For 8-bit signed values:

```text
smallest                         largest
   ↓                                ↓
 -128 ─────────────────────────── +127

So: +127 is the largest positive signed value we can represent.

```
Matamatically, if we add 127 + 1 = 128, but 128 is outside the signed range -128 -> +127. So something has gone wrong from the signed-number perspective. And that's precisely what Overflow (V) is telling us. ***V = 1 means the signed arithmetic result could not be represented correctly in the available number of bits.***

#### C (Carry Flag) vs V (Overflow Flag) :

This is where the distinction becomes clearer:

C -> "Did an unsigned carry go out?"
V -> "Did signed arithmetic go outside its valid range?"

They're asking different questions about the same bit operation.

Let's do your first actual V calculation. We have 8 bits:

```text
  0111 1111         (+127 in signed,  127 in unsigned)
+ 0000 0001         (+1 in signed,    1 in unsigned)
-----------
  1000 0000          128 (when interepted as unsigned)
                    -128 (when interepted as signed) but signed range is (-128 to 127)
  
```
**Question A: If this was Unsigned, did it break?**

Unsigned math expects 127 + 1 = 128. In unsigned bits, 1000 0000 is 128.
Did anything fall off the left side? No. (It takes 9 bits to exceed 255).
Result: `C = 0` (No unsigned carry happened. Everything is fine).

**Question B: If this was Signed, did it break?**

Signed math expects +127 + 1 = +128. But in signed 8-bit, +127 is the maximum positive limit!
The bit pattern 1000 0000 actually represents -128.
We added two positive numbers (+127$ and +1$) and got a negative result (-128).
Result: `V = 1` (Signed math overflowed and gave a garbage sign).

**Summary Table : Carry vs Overflow:**

| Flag         | Focus   |  Question    |
|--------------|---------|--------------------------------------------------|                                            
| C (Carry)    | Unsigned| Did the number get bigger than the physical hardware register can hold?   |           
| V (Overflow) | Signed  | Did the answer cross the sign boundary and flip from positive to negative (or vice versa)?|

Example: 

**Carry but NO Overflow**

Let's use 8-bit numbers to keep the calculation simple.

```text

  1111 1111
+ 0000 0001
------------
1 0000 0000

```

The 8-bit register can keep only `0000 0000` and the extra 1 is outside the register. Hence, `C = 1`and `Z = 0` and 
`N = 0`. To check Overflow V Flag, let's look at signed interpretation.

```text

1111 1111 = -1
0000 0001 = +1

```
So `-1 + 1 = 0`.  0 fits perfectly inside the signed range `-128 -> +127`. Therefore `V = 0`.

In above example, We had a carry, but there was no signed overflow. That's because `Unsigned 255 + 1 = 256` doesn't fit in 8 bit, hence `C = 1`. But `Signed `-1 + 1 = 0` fits, hence `V = 0`. So **Carry does NOT automatically mean overflow.**

C = 1, Z = 0, N = 0, V = 0.

**Example 2 — Overflow but NO Carry**

```text

  0111 1111
+ 0000 0001
------------
  1000 0000
  ↑
MSB

```
There is no extra 9th bit, so `C = 0`. Result is not zero, so `Z = 0`. The MSB is 1, so `N = 1`. Now lets calculate Overflow :

```

Before the operation:
0111 1111 = +127
0000 0001 = +1

So mathematically : +127 + +1 = +128

```
But 8-bit signed numbers can only represent `-128 → +127`. +128 doesn't fit. Hence, `V = 1`. The hardware gives us 
`1000 0000` which, when interpreted as signed, is `-128`. So `+127 + 1` has wrapped around to `-128`. 

C = 0, Z = 0, N = 1, V = 1.

**Example 3 — Neither Carry nor Overflow**

```text

  0000 0101             +5
+ 0000 0011             +3
------------
  0000 1000             +8
  
```
There is no extra 9th bit, so `C = 0`. Result is not zero, so `Z = 0`. The MSB is 0, so `N = 0`. +8 fits in `-128 -> +127`, so `V = 0`. 

C = 0, Z = 0, N = 0, V = 0.

**Example 4 — Subtraction with NO borrow**

```text

  0000 0101         5
- 0000 0011         3
------------
  0000 0010         2
  
```
`5 - 3`, no borrow was required, hence `C = 1`.  Result is not zero, so `Z = 0`. The MSB is 0, so `N = 0`. +2 fits in `-128 -> +127`, so `V = 0`. 

C = 1, Z = 0, N = 0, V = 0.

**Example 5 — Subtraction WITH borrow**

```text

  0000 0011         3
- 0000 0101         5
------------
  1111 1110         -2 signed 
  
```
`3 - 5`, requires borrow, hence `C = 0`.  Result is not zero, so `Z = 0`.  +2 fits in `-128 -> +127`, so `V = 0`. The resulting 8-bit pattern is `1111 1110`is `-2`as signed. And mathematically `+3 - +5 = -2`, So the signed result is perfectly valid. Therefore `V = 0`. The result has MSB 1, so `N = 1`. 

C = 0, Z = 0, N = 1, V = 0.

**Summary :**

| Flag          | Desc  |
| ------------- | ------------------------------------ |
| N             | If MSB = 1, then N = 1, else N = 0  |
| Z             | If final result all zeros? then Z = 1, else Z = 0  |
| C             | Addition: If a carry come out of the MSB? then C = 1, else C = 0
|               | For subtraction: no borrow -> C = 1, borrow -> C = 0 |
| V             | If mathematical signed result go outside the signed range? V = 0, else V = 1 |

Now lets connect these flags to the instructions that actually use them. 

#### CMP instruction

Based on few examples above, `CMP instruction` conceptually performs subtraction . Eg. R0 - R1 is clear. Suppose, `R0 = 5`, `R1 = 5`. When CPU executes, `CMP R0, R1`, its `5 - 5 = 0`. Lets rewrite it in biniary.

```text

0000 0101           +5
0000 0101   -       +5
---------      
0000 0000           0

```
The Result (8 bits) are 0, hence `Z = 1`. To calculate Carry Flag : Its Subtraction without borrow, hence `C = 1`. The MSB is 0, hence `N = 0`. The Subtraction of Signed (+5) from signed (+5) is 0, which is very well with the signed range `-128 to +127). Hence `V = 0`.

-    Z = 1   -> result is zero
-    C = 1   ->  subtraction happened without borrow
-    N = 0   ->  MSB of result is 0
-    V = 0   ->  signed result (+5 - +5 = 0) fits

The instruction `CMP` does not update the result 0 into R0 or R1. ***The subtraction result is used to update the flags, but the result itself is discarded.*** It is conceptually:

```
text
R0 - R1
   ↓
update N Z C V and result is discarded
```
So after ```asm CMP R0, R1```, we still have `R0 = 5 and R1 = 5` but `N = 0 ,Z = 1, C = 1, V = 0`. 

That's why this works:

```asm 
CMP R0, R1
BEQ equal
```
Because CMP made:

```text
R0 - R1 = 0
       ↓
     Z = 1
       ↓
     BEQ
       ↓
   branch to equal
```

Now, lets do the opposite. CMP R0, R1. And R0 is 3 and R1 is 7. (3 - 7 ). Lets write it in binary.

```text
     0000 0011  (3)
-    0000 0111  (7)
     -----------
     1111 1100  (-4)
 
 In ARM assembly, the CMP R0, R1 instruction calculates R0 - R1 to update the Application Program Status Register (APSR) flags without saving the numerical result to a destination register.
 
 Step-by-Step Binary Subtraction (via 2's Complement Addition):
 
 ```text
  0000 0011  (3)
- 0000 0111  (7)
-----------
  1111 1100  (-4)
```

In ARM assembly, the `CMP R0, R1` instruction calculates `R0 - R1` to update the Application Program Status Register (APSR) flags without saving the numerical result to a destination register.

**Step-by-Step Binary Subtraction (via 2's Complement Addition):**

1. **Find 2's complement of R1 (+7):**
* Invert bits: `1111 1000`
* Add 1: `1111 1001` (-7)


2. **Add to R0 (+3):**
```text
  0000 0011  (+3)
+ 1111 1001  (-7)
-----------
  1111 1100  (-4 in 2's complement)
```

**Condition Code Flags Set:**

* **Negative (N = 1):** The most significant bit (MSB) is `1`, indicating a negative result.
* **Zero (Z = 0):** The result is non-zero.
* **Carry (C = 0):** ARM uses the "addition with inverted carry" convention for subtraction. No carry-out occurred during the addition, which indicates a borrow was required (`R0 < R1`).
* **Overflow (V = 0):** Overflow (V = 0): No signed overflow occurred because the mathematical result, +3 - +7 = -4, is within the signed 8-bit range of -128 to +127.

#### Conditional Branches

**BEQ ** BEQ label means `Branch if Equal` Condition `Z = 1`. Because `R0 - R1 = 0` means `R0 == R1`. 
**BNE** ```asm BNE label```meams `Branch if Not Equal` Condition `Z = 0` Because `R0 - R1 ≠ 0` means `R0 != R1`
so 
```text
CMP R0, R1
     ↓
   N Z C V
     ↓
BEQ/BNE
```

#### unsigned comparisons using C
**BCS Branch if Carry Set** 
That means:
```text
C = 1 → branch
C = 0 → don't branch
```
Remember our subtraction rule:
```text
After subtraction:

C = 1 → NO borrow
C = 0 → borrow
```

Suppose: R0 = 7,  R1 = 3 and ```asm CMP R0, R1  BCS higher_or_equal``` We already know `7 - 3 = 4`. Therefore `C = 1`
```text
CMP R0, R1
      ↓
7 - 3 = 4
      ↓
No borrow
      ↓
C = 1
      ↓
BCS
      ↓
Branch taken

BCS = Branch if Carry Set = branch if C = 1.
```
And because after subtraction C = 1 means no borrow, BCS can be useful for unsigned ≥ comparisons.

**BCS / BHS — Unsigned ≥ Unsigned ≥**
Condition: C = 1. Meaning:   R0 >= R1

Example:
```text
R0 = 7
R1 = 3

7 - 3 = 4
C = 1
```
Branch :
```
BCS = Branch if Carry Set
BHS = Branch if Higher or Same

They're different names for the same condition.
```

**BCC Branch if Carry Clear:**

```text
C = 0 → branch
C = 1 → don't branch
````

Suppose: `R0 = 3, R1 = 7`
```asm
CMP R0, R1
BCC lower
``` And we know 3 - 7 = -4 and C = 0

```text
R0 = 3
R1 = 7

CMP R0, R1
      ↓
3 - 7 = -4
      ↓
Borrow required
      ↓
C = 0
      ↓
BCC
      ↓
Branch taken
```

```text
| Instruction | Meaning               | Condition |
| ----------- | --------------------- | --------- |
| `BCS`       | Branch if Carry Set   | `C = 1`   |
| `BCC`       | Branch if Carry Clear | `C = 0`   |

```
And because we're using CMP (subtraction):
```text
C = 1 → no borrow → R0 ≥ R1  (unsigned)
C = 0 → borrow    → R0 < R1  (unsigned)
```

**BCC / BLO — Unsigned <:**
Condition: C = 0
Meaning:   R0 < R1
Example: R0 = 3 and R1 = 7. 3 - 7 = -4
borrow → C = 0

Branch
```text
BCC = Branch if Carry Clear
BLO = Branch if Lower

Again, same condition.
```

**BCS does not literally mean "R0 is higher or equal.** Its actual definition is simply: `BCS → branch when C = 1.`
The unsigned comparison interpretation comes from the fact that CMP performed a subtraction. So:
```text
CMP R0, R1
BCS label
```
can be understood as:
```text
R0 - R1
   ↓
no borrow?
   ↓
C = 1?
   ↓
yes → branch
```
**BHI  Branch if Higher (unsigned):**

Its condition is `C = 1 AND Z = 0`. Because:
```text
C = 1 → no borrow → R0 ≥ R1 unsigned
Z = 0 → R0 ≠ R1
Z = 1 → result is zero → R0 = R1
```
Together `R0 > R1`. So if we want R0 > R1, we need:
```text
R0 ≥ R1
    AND
R0 ≠ R1 which means `R0 > R1`. 
```
***Example**
R0 = 7, R1 = 3. ```asm CMP R0, R1```. We already calculated `7 - 3 = 4`. So , C = 1 and Z = 0.

Therefore:
```text
C = 1 AND Z = 0
       ↓
     TRUE
       ↓
BHI → branch
```
So: 7 is higher than 3 → branch.

**Example 2**
Suppose R0 = 7 and R1 = 7.  We know `7 - 7 = 0` Therefore C = 1 and Z = 1. Now: BHI higher. But BHI requires: C = 1 AND Z = 0
But we have: C = 1 ← no borrow, Z = 1  ← result is zero. Hence BHI is not taken. 
C = 1 alone tells us: R0 ≥ R1 (unsigned)
But we want: R0 > R1
So we additionally require: Z = 0

Therefore:
```text
BHI = C = 1 AND Z = 0
    = unsigned R0 > R1
```

**BLS = Branch if Lower or Same (Unsigned)**
It is essentially the opposite of BHI.  BHI `C = 1 AND Z = 0` means `R0 > R1   (unsigned)`. 
BLS `C = 0 OR Z = 1 ` means `R0 <= R1   (unsigned)`

Because: 
```text
C = 0 → borrow → R0 < R1
Z = 1 → equal → R0 = R1

Together:

R0 < R1  OR  R0 = R1
        ↓
     R0 <= R1
```
**Example:**
R0 = 3 and R1 = 7 ```asm CMP R0, R1```.

Conceptually: `3 - 7 = -4`. 

Therefore:
```text
C = 0  → borrow occurred
Z = 0  → result is not zero
```

Now check BLS. ```asm BLS lower_or_same```. Its condition is `C = 0 OR Z = 1`. 

We have: 
```text
C = 0 
Z = 0
```
Because OR only needs one condition to be true:
```text
C = 0 OR Z = 1
     ↓
   TRUE

Therefore: BLS → branch is taken. 
```

Now we have :
```text
BHI → C = 1 AND Z = 0 → R0 > R1
BLS → C = 0 OR  Z = 1 → R0 ≤ R1

These are opposites:

R0 > R1
   ↕
R0 ≤ R1
```
#### Signed comparisons :
Here N and V become important.

### `BGT` — Signed Greater Than (`>`)
* **Condition:** `Z = 0` AND `N = V`
* **Meaning:** `R0 > R1`

```text
R0 = +7
R1 = +3

7 - 3 = +4

Flags: N = 0, V = 0, Z = 0

Evaluation:
  N = V (0 = 0) → Yes
  Z = 0        → Yes

Result: Branch Taken
```
### `BLT` — Signed Less Than (`<`)

* **Condition:** `N ≠ V`
* **Meaning:** `R0 < R1`

```text
R0 = +3
R1 = +7

3 - 7 = -4

Flags: N = 1, V = 0

Evaluation:
  N ≠ V (1 ≠ 0) → Yes

Result: Branch Taken
```

### `BGE` — Signed Greater Than or Equal (`>=`)

* **Condition:** `N = V`
* **Meaning:** `R0 >= R1`

```text
R0 = +7
R1 = +3

7 - 3 = +4

Flags: N = 0, V = 0

Evaluation:
  N = V (0 = 0) → Yes

Result: Branch Taken
```

### `BLE` — Signed Less Than or Equal (`<=`)

* **Condition:** `Z = 1` OR `N ≠ V`
* **Meaning:** `R0 <= R1`

```text
R0 = 3
R1 = 7

3 - 7 = -4

Flags: Z = 0, N = 1, V = 0

Evaluation:
  N ≠ V (1 ≠ 0) → Yes

Result: Branch Taken
```

---

## Complete Condition Code Map

| Instruction | Condition | Type | Comparison |
| --- | --- | --- | --- |
| **`BEQ`** | `Z = 1` | Signed / Unsigned | `==` |
| **`BNE`** | `Z = 0` | Signed / Unsigned | `!=` |
| **`BCS`** / **`BHS`** | `C = 1` | Unsigned | `>=` |
| **`BCC`** / **`BLO`** | `C = 0` | Unsigned | `<` |
| **`BHI`** | `C = 1` AND `Z = 0` | Unsigned | `>` |
| **`BLS`** | `C = 0` OR `Z = 1` | Unsigned | `<=` |
| **`BGE`** | `N = V` | Signed | `>=` |
| **`BLT`** | `N ≠ V` | Signed | `<` |
| **`BGT`** | `Z = 0` AND `N = V` | Signed | `>` |
| **`BLE`** | `Z = 1` OR `N ≠ V` | Signed | `<=` |

---

## The Key Split

```text
                     CMP R0, R1
                         │
          ┌──────────────┴──────────────┐
          ↓                             ↓
      Unsigned                        Signed
     (C and Z)                     (N, V, and Z)
          │                             │
    BHI / BLS                     BGT / BLT
    BCS / BCC                     BGE / BLE

```

#### Q flag — Saturation
The Q flag is bit 27 of APSR/xPSR . We already know that N, Z, C, V are condition flags produced by normal arithmetic operations. But Q is different. So lets understand what saturation means. Normally, when a number becomes too large for the available number of bits, it wraps around.

For example, imagine an 8-bit signed number. Its range is -128 to +127. Suppose we want to calculate `120 + 20 = 140`, but 140 cannot fit into a signed 8-bit number. With normal arithmetic, we get overflow/wraparound.

With saturation arithmetic, instead we say `120 + 20 = 127` We clamp the result to the maximum representable value.

Likewise `-120 + (-20) = -140` cannot fit, so saturation gives `-128`
```text
                 Saturation
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
too large                    too small
        ↓                         ↓
maximum value               minimum value
```
Saturation arithmetic is particularly useful in things like Digital signal processing, Audio processing, Motor/control algorithms, Image processing, Fixed-point mathematics. Imagine an audio sample that must stay between `-32768` and `+32767`. If an operation produces 40000, we don't necessarily want it to wrap around to some negative number. We may instead want `40000` to the maximum possible limit `32767`. That's saturation.

So what does the Q flag in APSR actually tell us? Q = 1 indicates that a saturation condition has occurred. ie, result exceeds maximum -> result is saturated -> Q = 1. The Q flag acts somewhat like a sticky indication that saturation has occurred. "Sticky" is important. **Once Q becomes 1, it generally stays 1 until explicitly cleared.** It isn't simply recalculated to 0 after the next ordinary arithmetic instruction in the way Z/N/C/V can be.

For Cortex-M4, the Q flag is not part of the normal ADD/ADDS overflow example. To demonstrate all five APSR flags together, we'll use the same arithmetic result to understand N/Z/C/V, and then separately show how Q behaves with a saturating instruction.

### APSR — Complete Example

The APSR contains these important condition flags:
```text
Bit:    31    30    29    28    27
        ┌─────┬─────┬─────┬─────┬─────┐
        │  N  │  Z  │  C  │  V  │  Q  │
        └─────┴─────┴─────┴─────┴─────┘
N → Negative
Z → Zero
C → Carry
V → Signed overflow
Q → Saturation occurred
```

**Example 1 — N, Z, C, V Flags**

Let's use 8-bit arithmetic just to make the bits easy to see.
```text
  0110 0100    (+100)
+ 0011 0010    (+50)
------------
  1001 0110
```
Mathematically `100 + 50 = 150`.  But an 8-bit signed number can represent only between -128 to +127.
Now let's examine each flag individually.
**N — Negative flag :**
The result's MSB is 1, so the result is interpreted as negative in signed two's-complement representation. Hence N = 1. N simply copies the result's most significant bit.

N does not tell us whether signed overflow happened. It only tells us the sign bit of the result.

**Z — Zero flag :**
Our result is `1001 0110`. It is not zero. Therefore `Z = 0`. Z = 1 only when the complete result is 0. If Result is 0000 0000 then Z = 1, if 
anything else otherthan 0, then Z = 0.

**C — Carry flag :**
Look at the addition:
```text
  0110 0100
+ 0011 0010
------------
  1001 0110
```
There is no carry out of bit 7. No carry was produced beyond the most significant bit. Therefore C = 0. And remember C is about unsigned carry-out. If we interpret the operands as unsigned 100 + 50 = 150 and 150 fits inside an unsigned 8-bit number 0 ... 255 so there is no carry-out.

**V — Overflow flag :**
Here both operands are positive: 0110 0100 → (100) positive) and 0011 0010 → (50) positive. But the result has MSB 1, 1001 0110 → negative
So positive + positive → negative. That cannot happen in valid signed arithmetic without overflow. Therefore, V = 1, Signed overflow occurred.

The mathematical result is +150 but the maximum signed 8-bit value is +127
So after the ordinary addition:
```text
  0110 0100
+ 0011 0010
------------
  1001 0110
```
Q	—	Ordinary addition isn't a saturation operation

So, N = 1, Z = 0, C = 0, V = 1.

**Example 2 — Q flag / Saturation :**

Now let's specifically demonstrate Q. Suppose a saturating signed 8-bit addition performs 100 + 50. Mathematical result is 150
But signed 8-bit maximum = +127.
Therefore, instead of wrapping around:
```text
150
   ↓
0111 1111
   ↓
+127

The result is saturated to +127. And Q = 1 because saturation occurred.
```
So:
- Mathematical result = +150
- Representable result = +127
- Saturation occurred = YES
- Q = 1
What makes Q different? N/Z/C/V describe properties of the arithmetic result.
Q tells us: **A saturating arithmetic operation encountered a value that could not be represented and therefore had to be saturated. And Q is sticky:**
```text
Q = 0
   ↓
saturation occurs
   ↓
Q = 1
   ↓
stays 1 until explicitly cleared
```
#### Overview APSR:
APSR — Application Program Status Register (Cortex-M4) 

Bit 31 → N : Negative
    Set when the result's MSB is 1.
    Indicates the sign bit of the result.

Bit 30 → Z : Zero
    Set when the result is exactly 0.

Bit 29 → C : Carry
    Set when an addition produces a carry out of the most significant bit.
    For subtraction, it has a related borrow/no-borrow interpretation.
    C is associated with unsigned arithmetic.

Bit 28 → V : Overflow
    Indicates signed arithmetic overflow.
    Example:
        positive + positive → negative
        negative + negative → positive

Bit 27 → Q : Saturation
    Set when a saturating arithmetic operation produces a value outside the representable range and the result is saturated to the maximum/minimum.
    Q is sticky and remains set until explicitly cleared.
    
The four flags we are able to distinguish instantly:
- N → Is the result's MSB bit 1?
- Z → Is the result zero?
- C → Did unsigned carry/borrow condition occur?
- V → Did signed overflow occur?
- Q → Did a saturation operation have to clamp the result?

Q is not automatically set just because the mathematical result overflows. Q is set by saturating instructions when saturation actually occurs. Normal ADD/SUB sets N, Z, C, V, but does not set Q.

### EPSR — Execution Program Status Register :

```text
                    xPSR
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
        APSR         EPSR        IPSR
          │           │           │
      Arithmetic    Execution    Exception
        status        state        status
 ```
 So EPSR is not about arithmetic. APSR answered questions like if the result was zero or if the signed overflow happened etc. EPSR answers questions more like what execution state is the CPU currently in? EPSR is entirely about processor control state rather than arithmetic output.
 
 ```text
 EPSR
 │
 ├── T bit
 │
 └── ICI / IT bits
 ```
 
 **The important EPSR bits on Cortex-M4 :**
 ```text
EPSR Bit Layout:

 31                     26 25 24                    15         10  9                 0
┌──────────────────────┬─────┬──┬──────────────────┬─────────────┬──┬──────────────────┐
│       Reserved       │ICI/ │T │     Reserved     │   ICI / IT  │  │     Reserved     │
│       (APSR)         │ IT  │  │                  │             │  │      (IPSR)      │
└──────────────────────┴─────┴──┴──────────────────┴─────────────┴──┴──────────────────┘
                        └─┬─┘ │                     └──────┬────┘
                          │   └── Bit 24 (Thumb)           │
                          └─────────── ICI / IT ───────────┘
                      
EPSR bit 24 is called as T bit (T = Thumb state) which holds the execution state. It tells us how the processor is executing instructions.
On a Cortex-M4, the processor executes Thumb instructions. Therefore, during normal operation `T = 1`.

```
Lets understand what does Thumb state means? Thumb state means the Cortex-M4 is interpreting the instructions using the **Thumb instruction set.** It is about how the CPU interprets the instruction bits, not about whether our C program is in a particular mode. Lets think of the CPU receiving a stream of bits from memory:

```text
Flash
  ↓
instruction bits
  ↓
Cortex-M4
  ↓
"How should I interpret these bits?"
  ↓
Thumb instruction set
```
The T bit (24th bit of EPSR) tells the processor about this execution state. This name (Thumb) comes from ARM's history. Originally, ARM processors had a full 32-bit ARM instruction set. ARM then introduced a more compact instruction set called Thumb.
 
**ARM state vs Thumb state :**
Historically, classic ARM processors could operate in two instruction states:
```text
              ARM processor
                   │
          ┌────────┴────────┐
          ↓                 ↓
      ARM state         Thumb state
          │                 │
     ARM instructions   Thumb instructions
       generally          generally
       32-bit encoded     more compact 16-bit or 32-bit encoding
```
Modern Thumb, specifically Thumb-2, contains both 16-bit instructions and 32-bit instructions.  And the Cortex-M4 uses Thumb-2. The Cortex-M4 does not support the classic ARM instruction state. It executes Thumb instructions. Thats why we usually have EPSR.T = 1 during normal operations.

**Then why do we even have a T bit?**
The T bit exists because the ARM architecture has an execution-state concept, and the processor architecture needs to maintain that state. But on Cortex-M, the architecture is much more constrained. We arent allowed to change the 24th bit EPSR.T because Cortex-M4 doesn't support ARM state. Attempting to create an invalid execution state can result in an exception such as a UsageFault.

***T records the instruction-set execution state, and Cortex-M4 operates in Thumb state.***

###### state Vs mode :
These are two different concepts.

**Execution state :**
- ARM state
- Thumb state

This answers, which instruction set is the processor using?

**Processor mode :**
- Thread mode
- Handler mode

This answers, is the processor executing normal application code or handling an exception?

```text
             Cortex-M4
                 │
        ┌────────┴────────┐
        │                 │
   Execution state    Processor mode
        │                 │
     Thumb            Thread / Handler    
                         │       │
                      normal    interrupt
                  application
```
**Example :**
Imagine our STM32F407 is executing:
```c
int main(void)
{
    while (1)
    {
        GPIOA->ODR ^= (1 << 5);
    }
}
```
The compiler generates Thumb-2 machine instructions.
The processor is therefore:
```text
Execution state: Thumb

T bit: 1

Processor mode: Thread mode
```
Now suppose a timer interrupt occurs. The processor enters the timer ISR:
```c
void TIM2_IRQHandler(void)
{
    // handle interrupt
}
```
```text
Execution state: Thumb

T bit: 1

Processor mode: Handler mode
```
**T doesn't tell us whether we're in Thread or Handler mode. It states that it is in Thumb execution state (T = 1) and Thumb-2 instructions can be 16-bit or 32-bit encoded.**

Example:

Imagine the CPU is executing:
```c
int main(void)
{
    while (1)
    {
        // normal code
    }
}
```
No interrupt is active.
For normal application execution in Thread mode:
```text
T = 1
Processor mode = Thread mode
IPSR[8:0] = 0
```
The IPSR (Interrupt Program Status Register) contains the exception number.
- 0 → Thread mode / no active exception
- Non-zero → an exception is currently active

So when our main() is running:
```text
main()
  ↓
Thread mode
  ↓
No active exception
  ↓
IPSR = 0
```

| What are we asking?        | Where do we get the answer?                    |
| -------------------------- | ---------------------------------------------- |
| Which instruction state?   | EPSR.T = 1                                     |
| Thread or Handler mode?    | IPSR exception number   helps determine this   |
| Which exception is active? | IPSR[8:0]                                      |

```text
                    xPSR
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      APSR          EPSR         IPSR
     flags       execution     exception
                   state        number
                                  │
                                  ↓
                              0 = Thread
                           non-zero = Handler
```
The processor's mode isn't stored as a separate "Thread/Handler" bit in xPSR. We infer the mode from the active exception state: IPSR = 0 means Thread mode; a nonzero exception number means Handler mode.

Suppose a Timer interrupt occurs and the processor enters its interrupt handler. Then,
T = 1. Entering an interrupt does not change the instruction set state. The Cortex-M4 continues executing Thumb/Thumb-2 instructions.
```text
Interrupt occurs
      ↓
Enter Handler mode
      ↓
Still Thumb state
      ↓
T = 1
```

Processor mode = Handler mode . The processor is now executing an exception handler, so it is in Handler mode.
IPSR[8:0] contains the exception number. The number 1 specifically means Reset. Exception number 1 = Reset. For example, the Cortex-M exception numbers begin roughly like this:
```text
Exception Number    Meaning
──────────────────────────────
0                   Thread mode / no active exception
1                   Reset
2                   NMI
3                   HardFault
4                   MemManage
5                   BusFault
6                   UsageFault
etc
```
Then the external interrupts start at 16. 16 + IRQ number = exception number. So if our timer peripheral generates, for example, IRQ number 30,
```text
Exception number = 16 + 30  = 46 
Then: IPSR[8:0] = 46
```
**IPSR tells me WHICH exception is currently active.**
```text
IPSR = 0
    ↓
No active exception
    ↓
Thread mode

Whereas:
IPSR = 46
    ↓
Exception number 46 is active
    ↓
Handler mode

T = 1 in both cases (Thread mode and in Handler mode)   
```
**What are IT bits?**
IT means If-Then. It allows certain Thumb instructions to execute conditionally without using a branch every time.
For example, conceptually:
```asm
CMP   R0, #0
IT    EQ
MOVEQ R1, #1
```
Meaning roughly, if the comparison was equal, execute the following instruction.
The processor needs to remember the conditional-execution state, and that's where the IT bits in EPSR come in.
**What are ICI ?**
ICI stands for Interruptible-Continuable Instruction. This is more specialized. It allows the processor to interrupt certain multi-part instructions and later continue them correctly.

Lets learn more in detail about IT bits and ICI later.

#### IPSR - Interrupt Program Status register
The IPSR is a 32-bit register. But on Cortex-M4, only the lowest 9 bits are used for the Exception Number.
```text
Bit:  31                              9  8                    0
      ┌────────────────────────────────┬───────────────────────┐
      │            Reserved            │   Exception Number    │
      │                                │       [8:0]            │
      └────────────────────────────────┴───────────────────────┘
                                      ↑
                                  9 bits
```
- IPSR[8:0] = Exception Number
- IPSR[31:9] = Reserved

Because the processor needs to store an exception number. 9 bits can represent: `2⁹ = 512`, different values from 0 to 511. That's more than enough for the exception numbers that Cortex-M4 needs. The important thing is not that every value from 0–511 corresponds to an exception. Many values are unused/reserved.

Imagine the Cortex-M4 is executing:
```c
int main(void)
{
    while (1)
    {
        // normal application
    }
}
```
No interrupt and no exception is currently being handled. Then IPSR[8:0] = 0, Processor mode = Thread mode. IPSR[8:0] contains the number of the currently active exception. If it is 0, no exception is active, and the processor is in Thread mode. 

And when an exception/interrupt is active, IPSR[8:0] = exception number → Handler mode
```text 
                                    xPSR
                                     │
       ┌─────────────────────────────┼──────────────────────┐
       ↓                             ↓                      ↓
     APSR                          EPSR                   IPSR
                                     │                      │
    Arithmetic flags           Execution state      Current exception 
          │                     information               number                
  ________│__________         _____│______________           │
 │ N │ Z │ C │ V │ Q │        │ T  │ ICI/IT       │     _____│______________
 │ 31│ 30│29 │28 │ 27│        │ 24 │ 10:15, 24:26 │     │ Exception Number │
 +-------------------+        +-------------------+     │      0:8         │
                                                        +------------------+
```

**xPSR is one 32-bit register, but ARM presents different portions of those 32 bits as APSR, EPSR, and IPSR. So we're not dealing with three physically separate 32-bit registers here. They are different views/fields of the processor's combined Program Status Register(PSR).**

#### CONTROL register

The Cortex-M4 has two important execution modes: Thread mode (normal application ) and handler mode(exception / interrupt). Now CONTROL is primarily relevant to Thread Mode. It controls two important things:
```text
CONTROL
   │
   ├── nPRIV  → privilege level -------> Privileged / Unprivileged
   │
   └── SPSEL  → which stack pointer is used ------> MSP (Main Stack pointer) / PSP (Process Stack Pointer)
```
CONTROL does NOT decide Thread Mode vs Handler Mode. That distinction comes from the processor's exception mechanism.

Incase if Cortex-M4 is running ordinary application code in Thread Mode, and an interrupt occurs, what happens to the processor's mode? It changes from Thread Mode to Handler Mode. When an exception/interrupt is taken:
```text
Before interrupt
────────────────
Thread Mode
    │
    │ exception occurs
    ▼
Handler Mode
    │
    │ ISR executes
    ▼
exception return
    │
    ▼
Thread Mode
```
So the interrupt changes the processor's mode. CONTROL did not cause this mode change. The Cortex-M4's exception mechanism caused it.
So what does this CONTROL register do? CONTROL answers a different question. The CONTROL register controls which stack pointer is used in Thread Mode, and it controls the privilege level of Thread Mode.

CONTROL has two important bits. 
```text
CONTROL
   │
   │
   │── Bit 0: nPRIV
   │        │
   │        ├── 0 → Privileged
   │        └── 1 → Unprivileged
   │        
   │── Bit 1: SPSEL
          │
          ├── 0 → MSP
          └── 1 → PSP
           
```
CONTROL controls two things in Thread Mode:
1.  Which stack pointer is used: MSP or PSP.
2.  Whether Thread Mode is privileged or unprivileged.

**In Handler Mode, the processor always uses MSP, regardless of CONTROL.SPSEL.**

```text
CONTROL
31                    2    1      0
┌──────────────────────┬──────┬──────┐
│      Reserved        │SPSEL │nPRIV │
└──────────────────────┴──────┴──────┘
```
```text
nPRIV
nPRIV = 0 → Privileged
nPRIV = 1 → Unprivileged

SPSEL
SPSEL = 0 → MSP Thread mode / handler mode (MSP always in Handler mode)
SPSEL = 1 → PSP Thread mode
```
```text
                 CONTROL
                    │
          ┌─────────┴─────────┐
          │                   │
       nPRIV                 SPSEL
          │                   │
   Privileged?            Which SP in Thread mode?
          │                   │
     ┌────┴────┐          ┌───┴───┐
     │         │          │       │
     0         1          0       1
     │         │          │       │
   Priv.   Unpriv.       MSP     PSP
```
**SPSEL is relevant to Thread Mode. In Handler Mode, the processor uses MSP always. nPRIV matters for Thread Mode. Handler Mode is always privileged.**

So:
```text
                 Cortex-M4
                    │
          ┌─────────┴─────────┐
          │                   │
     Thread Mode          Handler Mode
          │                   │
     CONTROL matters       Privileged , MSP
          │
     ┌────┴────┐
     │         │
 nPRIV       SPSEL
     │         │
Privilege     Stack

```
So,
```text
THREAD MODE
    │
    ├── nPRIV → Privileged / Unprivileged
    │
    └── SPSEL → MSP / PSP


HANDLER MODE
    │
    ├── Privileged
    └── MSP
```

We learning about Stack Pointer R13, we came accross that on Cortex M4, R13 Stack Pointer (SP) can refer to 2 different physical Stack pointers. Main Stack Pointer (MSP) and Process Stack pointer (MSP). 
```text
              R13 / SP
                 │
        ┌────────┴────────┐
        │                 │
       MSP               PSP
```
1. MSP — Main Stack Pointer

MSP is primarily used for the main/system stack. A very important use is:
```text
Reset
  ↓
MSP
  ↓
Exception / Interrupt
  ↓
Handler Mode
  ↓
MSP
```
When an interrupt occurs, the processor uses MSP for Handler Mode.

2. PSP — Process Stack Pointer

PSP is mainly useful for application/thread execution. For example, an RTOS such as FreeRTOS can give each task its own stack:
```text
Task A → PSP → Task A stack
Task B → PSP → Task B stack
Task C → PSP → Task C stack
```
This helps separate the application's/task stacks from the system/exception stack.

```text
                 Cortex-M4
                    │
             ┌──────┴──────┐
             │             │
            MSP           PSP
             │             │
       System/ISR       Task/Application
          stack             stack
```        
This gives the operating system or application a way to keep task/application stack usage separate from the main exception-handling stack.

CONTROL.SPSEL selects which stack pointer Thread Mode uses. SPSEL = 0  →  MSP, SPSEL = 1  →  PSP
While Handler Mode always uses MSP regardless of SPSEL.

It doesnt simply means that MSP = interrupt stack and PSP = normal stack.  MSP is the main stack pointer, and PSP provides an alternative stack pointer that can be used in Thread Mode. The exact choice depends on the software design—bare-metal code often uses MSP, while an RTOS commonly uses PSP for tasks.

Lets understand how it changes from privileged to unprivilaged and vice viceversa. The key point is that, Software changes the CONTROL register, but the processor enforces the privilege rules.

1. At reset, Thread Mode starts privileged

After reset, the Cortex-M4 starts executing Reset Handler in Thread Mode, and CONTROL is initially privileged `CONTROL.nPRIV = 0` and CONTROL.SPSEL = 0 (MSP). Reset is a special exception, but the processor does not execute the Reset Handler in Handler Mode. After reset, execution begins in Thread Mode, and initially it is privileged. That's the architecture-defined reset state. Conceptually,
```text
MCU Reset
   │
   ▼
Reset_Handler
   │
   │ Thread Mode
   │ Privileged
   │ MSP
   ▼
startup code
   │
   ├── .data initialization
   ├── .bss initialization
   ├── clock/system initialization
   └── call main()
             │
             ▼
       Application code
```
CONTROL controls which stack pointer is used in Thread Mode. So why does startup use MSP instead of PSP? Because SPSEL is initially 0. After reset, the processor starts with CONTROL.SPSEL = 0. So the startup code uses MSP. 

Where does the initial value of MSP come from? This connects directly to the startup code, Vector table. The very first word is MSP.
```text
Vector Table
────────────────────────
Address 0x08000000

+0x00 → Initial MSP value
+0x04 → Reset_Handler address
+0x08 → NMI_Handler
+0x0C → HardFault_Handler
...
```
For example:
```text
0x08000000 → 0x20020000
0x08000004 → Reset_Handler
```
Then execution begins at Reset_Handler. So before our startup code even executes, the hardware has already loaded the initial MSP.


2. Who changes Privileged → Unprivileged?

Our software executes an instruction that writes to the CONTROL register. For example, suppose we are writing bare metal Cortex-M4 code. We could write something like 
```asm 
MOV  R0, #1
MSR  CONTROL, R0
``` 
Here the important instruction is the MSR instruction ```asm MSR CONTROL, R0``` which means write the value in R0 into the CONTROL register.  If R0 contains 1 then CONTROL.nPRIV = 1 and the processor becomes Thread Mode -> unprivileged. So,
```text
Software
   ↓
MSR instruction
   ↓
CONTROL.nPRIV = 1
   ↓
Thread Mode becomes Unprivileged
```
C cannot normally just do this, because CONTROL isn't a normal memory-mapped peripheral register. It's a Cortex-M4 core special register. So we need a special instruction like ```asm MSR CONTROL, R0```or  some options provided by the compiler. For example, with GCC,
```c
__asm volatile (
    "mov r0, #1\n"
    "msr control, r0\n"
);
```
There are also CMSIS functions/intrinsics that wrap these special-register operations.

3. Unprivileged → privileged?

Once we are in unprivileged, we cannot simply change nPRIV back to 0 (privilaged). An unprivileged program cannot directly promote itself back to privileged. That's intentional. Otherwise privilege protection would be useless. So how does Unprivileged to Privileged happen? Its through an exception mechanism, typically a system service such as SVC (Supervisor Call). Conceptually,
```text
Unprivileged Thread Mode
        │
        │ SVC
        ▼
   Handler Mode
        │
        │ Privileged
        ▼
SVC Handler
        │
        │ performs privileged operation
        ▼
return to Thread Mode
```
**An unprivileged application asks privileged system software to perform an operation on its behalf.** This is one of the fundamental ideas behind operating systems and RTOSes. SVC doesnt change from CONTROL.nPRIV back to 0 automatically. Rather,
```text
Unprivileged Thread
        │
        │ SVC
        ▼
Privileged Handler
        │
        │ system decides what is allowed
        ▼
return
```
The privileged handler/OS controls what happens next. This is why Cortex-M4 has the privileged/unprivileged architecture:
```text
             Privileged code
                    │
          ┌─────────┴─────────┐
          │                   │
     Can access          Can deliberately
     privileged           enter unprivileged
     resources                mode
          │
          ▼
    Unprivileged code
          │
          │ needs privileged operation
          ▼
         SVC
          │
          ▼
   Privileged Handler
   ```
This becomes very important when we later study FreeRTOS, because tasks can run using PSP while the kernel operates with privileged access.

**Privileged → Unprivileged:**
- A privileged Thread Mode program can set CONTROL.nPRIV = 1. Software can deliberately set CONTROL.nPRIV = 1 using the MSR CONTROL instruction (typically through an assembly instruction or compiler/CMSIS intrinsic). The processor then executes Thread Mode as unprivileged.

**Unprivileged → Privileged:**
- An unprivileged program cannot directly clear nPRIV. It needs an exception/privileged mechanism such as SVC, whose handler executes in privileged Handler Mode.

Privilege is about what the CPU is allowed to do. It is not about whether the code is startup code, application code, or interrupt code.

Think of the Cortex-M4 CPU as having two permission levels:
| Mode         | Privilege                      |
| ------------ | ------------------------------ |
| Thread Mode  | Privileged **or** Unprivileged |
| Handler Mode | **Always Privileged**          |

When the CPU is executing our normal application in Thread mode, it can be either Privileged or Unprivileged depending on the CONTROL register. So what can the privileged do and what happens when in Unprivileged?

Privileged code has full access to the processor's resources, subject to the memory system/security configuration. For our Cortex-M4 learning, think of it as Privileged as trusted code. It can perform operations that unprivileged code cannot.

For example, privileged code can access system-control resources, including things associated with:
- changing processor control configuration
- configuring interrupts/exceptions
- accessing protected system registers
- changing certain special-purpose registers
- using privileged instructions

This is important because we don't want arbitrary application code to be able to change the entire CPU configuration.

Unprivileged means that the CPU deliberately restricts what this code is allowed to do. The code can still do normal things like the below, assuming the memory region is accessible to it.
```c
int a = 10;
a++;
GPIOA->ODR = ...;
function();
```
But certain operations are restricted. For example, an unprivileged thread cannot simply execute privileged operations and change itself to privileged or do things that an privilaged code does. The processor prevents that. That's the whole point of having the privilege mechanism.

**Thread Mode can run either privileged or unprivileged, while Handler Mode is always privileged. The CONTROL register determines Thread Mode's privilege (nPRIV) and, when in Thread Mode, whether MSP or PSP is used (SPSEL). Privileged software can transition Thread Mode to unprivileged; an unprivileged thread cannot promote itself back to privileged directly.**

#### FPCA - Floating-Point Context Active :

The Cortex-M4 can have a Floating Point Unit (FPU), which allows it to execute floating-point instructions such as calculations involving float. The processor has additional floating-point registers, 
```text
S0 – S31
FPSCR
```
These are part of the floating-point context. So now imagine the CPU is running an application:
```text
Normal CPU context
    ↓
R0–R12
SP
LR
PC
xPSR
```
If floating-point state is involved, there is additional context:
```text
Floating-point context
    ↓
S0–S31
FPSCR
```
**CONTROL[2] is the FPCA bit.**
```text
CONTROL

Bit 2       Bit 1       Bit 0
 FPCA       SPSEL       nPRIV
   │           │           │
   │           │           └── Privileged / Unprivileged
   │           └────────────── MSP / PSP
   └───────────────────────── Floating-point context active
```
When FPCA or 2nd bit of CONTROL is 0, then the current context does not have an active floating-point context. When it is 1, then the current context has an active floating-point context.

During an exception/interrupt, the processor may need to save the current context. Normally, it saves the core registers. If floating-point state is active, the processor may also need to preserve the floating-point registers. That is important because an interrupt must eventually return to the interrupted code without destroying its state. So conceptually,
```text
Application
   │
   │ using CPU registers
   ↓
Exception occurs
   │
   ↓
Processor saves context
   │
   ├── Core context
   │
   └── Floating-point context
         ↑
       FPCA helps
       identify this
```
It doesnt simply FPCA = 1 means the CPU is currently doing a floating-point calculation. FPCA indicates that the floating-point context is active for the current context.

#### PRIMASK

Before understanding what an PRIMASK is, lets see why do we need it. Imagine our Cortex-M4 is running our application,
```text
main()
  ↓
doing some important operation
  ↓
Interrupt arrives
  ↓
CPU stops current code
  ↓
runs ISR
  ↓
returns to application
```
Normally, the processor can respond to interrupts/exceptions. But sometimes software needs a short section where it says, normal maskable interrupts not to disturb it right now. That's where PRIMASK comes in.

**PRIMASK is a 1-bit interrupt-mask register.** Its important bit is bit 0.
```text
PRIMASK
31                         1   0
┌──────────────────────────┬───┐
│        Reserved          │ I │
└──────────────────────────┴───┘
                              ↑
                           PRIMASK[0]
Only bit 0 matters.
```                          
When bit 0 of PRIMASK is 0, then interrupt masking is not enabled. The processor can respond normally to configurable exceptions/interrupts. When bit 0 of PRIMASK is 1, then the processor masks all exceptions with configurable priority. So conceptually,
```text
PRIMASK = 0
       ↓
Normal interrupt handling

PRIMASK = 1
       ↓
Most (NOT ALL) interrupts are blocked (NMI and HardFault are not masked by PRIMASK)
```
**NMI = Non-Maskable Interrupt — cannot be blocked by PRIMASK, BASEPRI, or FAULTMASK. HardFault = a serious processor fault/exception.**

NMI and HardFault are not masked by PRIMASK. We'll understand exactly why when we study exceptions. PRIMASK doesnt disable NVIC (Nested Vectored Interrupt Controller*). The NVIC still exists and can still have interrupts pending. **PRIMASK is a CPU-level mask that affects whether certain exceptions are allowed to be taken.**

*NVIC : It is a hardware block inside the Cortex-M4 processor/core system that manages interrupts from peripherals and other external sources. The NVIC's job is basically to help the processor manage interrupt requests. NVIC is the hardware interrupt controller that manages interrupts. For example,
```text
Timer reaches its value
        ↓
Timer generates interrupt request
        ↓
NVIC receives it
        ↓
NVIC considers priority/masking
        ↓
CPU takes the interrupt
        ↓
Handler/ISR executes
```
Lets learn more about NVIC in Exceptions lessons. 

Example:
Suppose `PRIMASK bit 0 = 0`, and a normal configurable interrupt arrives. Will the CPU be able to respond to that interrupt? Yes, because PRIMASK = 0 → configurable exceptions are not masked by PRIMASK, so the processor is allowed to take them (assuming the other conditions for exception handling are satisfied).

If PRIMASK bit 0 is 1, what happens to a normal configurable interrupt that arrives? PRIMASK[0] = 1 enables interrupt masking. Therefore, exceptions with configurable priority are prevented from being taken. The interrupt request isn't necessarily destroyed. It can remain pending and be handled later when the masking condition is removed. We'll understand pending vs active vs handled when we study NVIC/exceptions.

So basically, suppose our program is doing something and that must not be interrupted for a very short time, then we can control it using PRIMNASK. 
```text
Important operation
      ↓
Disable configurable interrupts
      ↓
Do the operation
      ↓
Enable configurable interrupts

We can conceptually do:

PRIMASK = 1
    ↓
critical section
    ↓
PRIMASK = 0
```
This is one reason PRIMASK exists. We can change the bit 0 of PRIMASK using Software with special Cortex-M instructions/ ARM Thumb instructions for cortex M such as,
```asm
CPSID i    → set PRIMASK = 1
CPSIE i    → clear PRIMASK = 0
```
CPS- Change Processor State. The i tells the processor that we're changing the interrupt mask represented by PRIMASK. 

Or in embedded C, we can use inline assembly something like ```c __asm volatile ("CPSID i");``` or ```c __disable_irq();```

##### Configurable vs non-configurable exceptions :

The Cortex-M has different types of exceptions.  Some exceptions have a programmable/configurable priority.Others have fixed priority defined by the architecture.

**Configurable exceptions :**
Their priority can be configured by software.

Examples include:
- External interrupts from peripherals
- Many system exceptions such as SysTick etc

For example, an STM32 timer might generate:
```text
Timer interrupt
     ↓
NVIC
     ↓
Priority can be configured
```
So it belongs to the group affected by interrupt masking mechanisms such as PRIMASK.

**Non-configurable / fixed-priority exceptions :**
Some exceptions have priorities that cannot simply be configured like normal interrupts.
The important ones for us right now are:
- NMI
- HardFault
PRIMASK does NOT mask NMI or HardFault. Because these are considered high-priority system-level exceptions. For example, if the processor encounters a serious fault, HardFault must still be able to happen.

Exception is the broad Cortex-M concept. An interrupt is one kind of exception. 
```text
                 EXCEPTIONS
                     │
          ┌──────────┴──────────┐
          │                     │
       Interrupts          System exceptions
          │                     │
     GPIO / Timer /       SysTick / SVC /
     UART etc.             HardFault / NMI
  ```
  
#### BASEPRI :

The key difference is:
- PRIMASK = 1 blocks all configurable-priority interrupts/exceptions.
- BASEPRI = block configurable interrupts below a certain priority.
- NMI and HardFault are not masked neither by BASEPRI nor by PRIMASK.

BASEPRI is another interrupt masking register, but unlike PRIMASK, it gives us priority-based control.
Cortex-M priority numbering:
- Smaller number = higher priority
- Larger number = lower priority

Example,
| Interrupt           | Priority |
| ------------------- | -------: |
| Emergency interrupt |        1 |
| UART                |        5 |
| Timer               |       10 |
| ADC                 |       15 |

Suppose BASEPRI = 10, then interrupts with priority 10 or numerically greater are prevented from being serviced, while higher-priority interrupts such as priority 1 or 5 can still run.

Blocks the lower-priority interrupts, but allow important high-priority interrupts. BASEPRI blocks interrupts whose priority number is greater than or equal to BASEPRI.

**NMI and HardFault are not masked neither by BASEPRI nor by PRIMASK.**

#### FAULTMASK
- PRIMASK → masks most configurable interrupts
- BASEPRI → masks configurable interrupts below a certain priority threshold
- FAULTMASK → masks almost everything, including HardFault

FAULTMASK is a 1-bit special-purpose register. Only bit 0 matters:
```text
FAULTMASK

31                         1   0
┌──────────────────────────┬───┐
│        Reserved          │ F │
└──────────────────────────┴───┘
                              ↑
                           bit 0
```
When the bit 0 of FAULTMASK is 0, then normal exception behavior. When the bit 0 of FAULTMASK is 1, then the processor masks all exceptions except NMI. So,
```text
FAULTMASK = 1
        │
        ├── NMI       →  allowed
        ├── HardFault →  masked
        ├── MemManage →  masked
        ├── BusFault  →  masked
        ├── UsageFault → masked
        └── IRQs      →  masked
```
**Comparisons :**
| Register      | What does `1` do?                                | HardFault | NMI       |
| ------------- | ------------------------------------------------ | --------- | --------- |
| **PRIMASK**   | Masks configurable interrupts                    | Allowed | Allowed |
| **BASEPRI**   | Masks configurable interrupts at/below threshold | Allowed | Allowed |
| **FAULTMASK** | Masks almost all exceptions                      | **Masked** | Allowed |

Just like PRIMASK, we can normally manipulate it using Cortex-M special instructions like ```asm CPSID f``` meaning `Set FAULTMASK = 1` and ```asm CPSIE f```, meaning `Clear FAULTMASK = 0`. These are ARM Thumb assembly instructions, not C statements. There are also CMSIS C functions that wrap these instructions, but we'll come to that later.

Imagine the processor is doing something extremely sensitive, and we temporarily don't want any fault handler to interrupt the operation. We could temporarily do,
```text
FAULTMASK = 1
        ↓
perform critical operation
        ↓
FAULTMASK = 0
```
But this is very powerful and dangerous. We generally don't casually use FAULTMASK in normal application code.

Even with FAULTMASK = 1, NMI still executes. That's because NMI is the highest-priority exception that can be masked neither by PRIMASK, BASEPRI, nor FAULTMASK.

#### FPSCR : Floating-Point Status and Control Register
It belongs to the FPU (Floating-Point Unit) of the Cortex-M4. FPU is Floating-Point Unit. It is hardware inside the Cortex-M4 that can perform floating-point operations efficiently, for example:
```c
float a = 3.5f;
float b = 2.0f;

float c = a * b;
```
Without an FPU, floating-point operations can be implemented in software, which is generally slower. With the Cortex-M4 FPU enabled, floating-point instructions can use the FPU's floating-point registers (S0 - S31).

FPCA = Floating-point context active. It is a bit in the CONTROL register:
```text
CONTROL
 ├── nPRIV
 ├── SPSEL
 └── FPCA       ← Floating-point context active
```
FPCA tells the processor whether the current Thread-mode context is considered to have an active floating-point context. It is mainly important when we talk about exception entry/return and FPU context preservation. And FPCA in CONTROL connects the normal core context with the FPU context.

**FPU registers vs FPSCR vs FPCA :**
Think of the FPU as a small floating-point subsystem inside the Cortex-M4:
| Thing                      | What is it?      | Purpose                                                         |
| -------------------------- | ---------------- | --------------------------------------------------------------- |
| **FPU registers (S0–S31)** | Registers        | Hold floating-point data/results                                |
| **FPSCR**                  | Register         | Holds **status flags + control information** for the FPU        |
| **FPCA**                   | Bit in `CONTROL` | Tells the core whether the **floating-point context is active** |

##### 1. FPU registers — S0–S31:

These are the actual registers used to store floating-point values. For example, `S0 = 3.5`, `S1 = 2.0`.
An instruction could perform `S2 = S0 × S1`.
So:
```text
S0 ── 3.5 ──┐
            ├── FPU ──→ S2 = 7.0
S1 ── 2.0 ──┘
```
These are data registers.

##### 2. FPSCR Floating-Point Status and Control Register:

This is a special register associated with the FPU. It does not hold our normal float values. Instead, it contains things such as floating-point status flags and control information.
Conceptually,
```text
FPU
│
├── S0–S31       → floating-point data
│
└── FPSCR        → FPU status/control
```

##### 3. FPCA Floating Point Context Active :

FPCA is not a separate register. It is bit 2 of the CONTROL register.
```text
CONTROL
 ┌────────┬────────┬────────┐
 │ nPRIV  │ SPSEL  │ FPCA   │
 │ bit 0  │ bit 1  │ bit 2  │
 └────────┴────────┴────────┘
```
So, 
```text
CONTROL.FPCA
       ↑
       └── tells the processor about the active
           floating-point context
```
S0–S31 → Where is the floating-point data?
FPSCR → What is the FPU's status/control state?
FPCA → Does this Thread-mode context have an active FP context?

Now, more on FPSCR — Floating-Point Status and Control Register. FPSCR contains several groups of bits:
```text
FPSCR
┌──────────────────────────────────────────────┐
│ NZCV │ ... │ QC │ ... │ RMode │ ...         │
└──────────────────────────────────────────────┘
```
The important ones for us are: NZCV, QC, RMode.

**NZCV :** These are floating-point condition flags.
- N → Negative
- Z → Zero
- C → Carry
- V → Overflow

**QC :** QC = Cumulative Saturation. This is related to saturation operations. And remember our earlier APSR discussion where we leant about Q flag there. For the FPU, there is a corresponding saturation-related status indication in FPSCR.

**RMode :** This controls the rounding mode used by floating-point operations. For example, floating-point calculations sometimes produce a result that cannot be represented exactly. The FPU needs to decide how to round that result.

Example:
Suppose the FPU performs: `S0 = 1.5`, `S1 = 2.0` and the operation `S2 = S0 + S1`. The result is S2 = 3.5
The actual number 3.5 goes into S2

The FPU's status information is reflected in FPSCR.
```text
             FPU
        ┌─────────────┐
S0 ────→│             │
S1 ────→│ calculation │────→ S2 = 3.5
        │             │
        └──────┬──────┘
               │
               ↓
             FPSCR
       status/control info
```
S registers hold the answer. FPSCR describes the condition/status of the floating-point operation and controls how the FPU behaves.It contains floating-point status flags and control fields.