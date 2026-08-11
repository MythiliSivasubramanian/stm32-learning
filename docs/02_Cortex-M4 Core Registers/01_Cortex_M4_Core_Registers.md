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

Example : ```asm SUB SP, SP, #4``` meaning to take the current value of SP, subtract 4, and put the result back into SP.  If SP = `SP = 0x20010000`. Then the CPU performs `0x20010000 - 4 = 0x2000FFFC` and stores that result back into the SP register. This operation happens inside the Cortex-M4 CPU, using its internal hardware (ALU). We haven't talked about what is stored in that RAM location yet. Changing MSP and storing something in the stack are two separate operations. **MSP is a CPU register → ALU calculates the new value → new value goes back into MSP.** The CPU's stack pointer is currently pointing at RAM address `0x2000FFFC` (the address of the current stack location). Incase, if we want to store something ((suppose R4 = 25) in this particular location in RAM, then we have to say to store the value 25 at the RAM address currently held by MSP. So basically MSP = `0x2000FFFC`which is an address of a Stack in RAM and in this address , we want to store the value 25. 

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

MSP → tells CPU WHERE the stack location is and R4  → tells CPU WHAT value to save. `PUSH` combines those ideas into one stack instruction.
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