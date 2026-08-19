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
    
```text
xPSR
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

