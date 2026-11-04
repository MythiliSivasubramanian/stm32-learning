Cortex-M4 Core Registers — Practical Experiments

This document demonstrates the Cortex-M4 core registers through C, assembly, debugger inspection, and STM32F407 hardware experiments.

For the complete theory explanation of Cortex-M4 core registers, refer to [01_Cortex_M4_Core_Registers.md](01_Cortex_M4_Core_Registers.md).

## Creating a Bare-Metal Project and Connecting the Debugger
Date: 04 November 2026

### My setup

Development board : STM32F407G-DISC1
Microcontroller : STM32F407VG
IDE : STM32CubeIDE
Programming language : C
Programming approach : Bare-metal (no HAL / LL)
Debugger : Onboard ST-LINK
Debug interface : SWD

I am recording the board, MCU, IDE, and debug connection and the produced observations and connecting the theroy part with the praticals. The STM32F407G-DISC1 is the board; STM32F407VG is the MCU fitted to it. ST-LINK is the debug probe on the board, and SWD (Serial Wire Debug) is the two-wire debug interface used to communicate with the MCU.

---

### 1. What I wanted to achieve

I wanted to create a separate praticals for learning Cortex-M4. My intention was to connect my STM32F407 board to STM32CubeIDE, start a debugging session and inspect the actual CPU registers.

### 2. Creating my project

I opened STM32CubeIDE and followed these steps:

1. Click on the STM32 Project Create/Import icon -> Create New STM32 Project -> STM32CubeIDE Empty Project.
2. Searched for and selected my STM32F407VG microcontroller.
3. Named the project `STM32F407_BareMetal_Learning`.
4. Selected STM32CubeIDE as the toolchain.
5. Generated the project.

**Screenshot:** Project creation / MCU selection.
![STM32CubeIDE MCU selection showing STM32F407VG](images/stm32cubeide-mcu-selection-stm32f407vg.png)

### 3. Checking the generated project

I opened `Core/Src/main.c`.

**Screenshot:** Generated `main.c` in the project.
![Generated main.c file in STM32CubeIDE](images/generated-main-c-infinite-loop.png)

The generated main function contained:

```c
int main(void)
{
    /* Loop forever */
    for(;;);
}
```
The processor remains in an infinite loop, continuously executing the loop instruction(s). This loop does not request sleep mode. I did not modify the code. I built the project using Project -> Build Project.

This is a minimal starting point: it gives the debugger a valid firmware image to load and run, without application code changing registers or peripherals. A successful build means the source compiled and linked into a firmware image; it does not by itself prove that the board was programmed or that the program behaves correctly on hardware.

**My observation:** Build finished successfully with no warnings.

### 4. Connecting my STM32F407 board and starting the debugger

I used the onboard ST-LINK debugger on the STM32F407G-DISC1. The USB connection carries the debugger communication between the computer and ST-LINK; ST-LINK then communicates with the MCU using SWD. I used the board's ST-LINK USB connector, rather than a USB connector intended for a different function on the board.

#### 4.1 Connect the board

1. I connected the board to my computer using the USB cable and the onboard ST-LINK USB connector.
2. I checked that the board was powered and that the ST-LINK connection was available to the computer. The debugger needs this physical connection before CubeIDE can connect to the MCU.
3. I kept the project built before starting the debug session. This gives CubeIDE a current firmware image to load. If I change the source later, I need to build again so the image matches the source I am looking at.

#### 4.2 Check the debug configuration

1. In CubeIDE, I opened **Run -> Debug Configurations...**.
2. In the left-hand list, I expanded **STM32 Cortex-M C/C++ Application** and selected the debug configuration for `STM32F407_BareMetal_Learning`. CubeIDE may create this configuration the first time I start debugging; the important part is to select the configuration associated with this project.
3. I opened its **Debugger** tab.
4. I checked the probe and interface settings:

   - **Debug probe:** ST-LINK (OpenOCD)
   - **Interface:** SWD

   ST-LINK identifies the debug hardware. OpenOCD is the debug server used by this setup to communicate with the probe. SWD is the debug interface between ST-LINK and the MCU. These settings must match the hardware connection, otherwise CubeIDE may fail to establish the session.

5. I clicked **Apply** to save the configuration, then clicked **Debug** to launch it.
6. If CubeIDE asked whether to switch to the Debug perspective, I accepted. A perspective is an IDE layout containing the views used for debugging; changing perspective does not change the firmware.

#### 4.6 If debugging does not work

If CubeIDE cannot connect to my board, I should check:

1. Is the USB cable connected to the ST-LINK connector?
2. Is the board powered?
3. Is the ST-LINK detected by the computer?
4. Is the correct project selected?
5. Is ST-LINK (OpenOCD) selected?
6. Is SWD selected as the debug interface?
7. Did the project build successfully?

If it still fails, I should record the exact error
message before changing debugger settings.


#### 4.3 What happens when debugging starts

CubeIDE starts the debug server, connects to ST-LINK, connects to the MCU through SWD, and loads the project's firmware image if the configuration is set to download it. It then halts the processor at a startup location or breakpoint so I can inspect it. The exact first halt location can depend on the debug configuration and startup code. A halted target is expected during debugging; it is not by itself a sign that the MCU is broken.

If the debugger does not connect, I should check the simple things first: the board is connected through the ST-LINK USB connector, the board is powered, the selected configuration belongs to this project, and the probe/interface still show ST-LINK and SWD. I should also read the CubeIDE error text, because a connection failure and a firmware/build failure are different problems.

#### 4.4 Confirm that the target is connected and halted

I checked the **Debug** view in the Debug perspective:

1. I looked for the `STM32F407_BareMetal_Learning` debug session and its STM32F407 target in the Debug window.
2. I checked the thread status. Mine showed `Suspended: Breakpoint`, meaning the processor was halted at a breakpoint and was not currently executing instructions.
3. I checked the editor for the execution arrow. Mine appeared at the infinite loop in `main()`.

These checks tell me the session attached to the target and execution is halted at the displayed source location. The source arrow is the debugger's best source-level mapping for the current instruction; optimized or startup code can sometimes make source mapping less direct. I should record the status and location together with any register value, because register values change while the processor runs.

**My observation:** The debugger connected successfully. The STM32F407 target appeared in the Debug window, the thread showed `Suspended: Breakpoint`, and the execution arrow appeared at the infinite loop in `main()`.

#### 4.5 Open the CPU Registers view

1. While the target was halted, I opened **Window -> Show View -> Other...**.
2. I expanded **Debug** and selected **Registers**.
3. In the Registers view, I located `SP` (also called `R13`). If the view is empty or says there is no active debug context, I first select the active target/thread in the Debug view. Register values are meaningful only for a selected target, and I should keep the target halted while recording them.

The Registers view shows the CPU state at the current halt. It is a snapshot: if I resume execution, the CPU can change registers before I halt it again. This is why I recorded the breakpoint status and source location along with SP.

### 5. CPU register inspection — SP

I opened the Registers view: Window -> Show View -> Other -> Debug -> Registers

I located the SP register.

**Actual value observed on my board:**

```text
SP (R13) = 0x2001FFFC
```

The address is within the STM32F407VG main SRAM region.

```text
SRAM start : 0x20000000
SRAM end   : 0x2001FFFF
```

The end address is inclusive. An address from `0x20000000` through `0x2001FFFF` is in this SRAM range. The observed `0x2001FFFC` is four bytes below the top address `0x2001FFFF` and is word-aligned (its lowest two address bits are zero), which is consistent with a stack address. This range check tells me the address is plausible; it does not, on its own, explain why SP has this exact value or prove which stack pointer (MSP or PSP) is currently selected.

I had learned earlier that SP is R13 and that the Cortex-M4 stack grows towards lower memory addresses. Now I could see an actual stack address in the debugger!

**My understanding:**

The SP register (R13) stores the current stack pointer address. In Cortex-M, R13 refers to the currently selected stack pointer: normally MSP in Handler mode, and MSP or PSP in Thread mode depending on CONTROL.SPSEL. My observed value lies inside the STM32F407VG's main SRAM region.

The stack is used for temporary data such as saved registers, function call data, and exception context. It grows toward lower addresses: pushing data generally moves SP down, and popping data generally moves it up. I have not yet investigated why SP currently has this exact value, which stack pointer is selected, or whether startup code or debugger setup accounts for it. I will explore those in later experiments rather than assume this one observation explains the whole startup sequence.

**Screenshot:** Registers view showing SP.

![STM32CubeIDE Registers view showing SP (R13) as 0x2001FFFC](images/debugger-registers-view-sp-r13.png)

### 6. Technical words I learned today


-   **Bare-metal :** Writing firmware directly at the hardware/register level without using HAL or LL drivers. 
-   **IDE :** The software environment where I write, build and debug my program. 
-   **Build :** Converting my source code into an executable firmware image. More precisely, compilation and linking produce the executable image.compilation and linking produce the executable image.
-   **ST-LINK :** The hardware debugger/programmer available on my Discovery board.
-   **SWD :** Serial Wire Debug. It is the interface used to communicate with and debug the Cortex-M processor. Signal names: SWDIO (Data signal used by the debugger) and SWCLK (Clock signal used by the SWD interface). I encountered these terms in	ST-LINK connection and in Debug configuration
-   **OpenOCD :** Software that communicates with the debug probe and provides debugging services. Encountered this term in Debug configuration. 
-   **Debugger :** A tool that lets me pause the processor, inspect registers and execute code step by step.
-   **Breakpoint :** Helps to pause the processor execution when the specified location is reached.
-   **SP :** Stack Pointer. It holds the current stack position. 
-   **SRAM :** RAM used by the processor while the program is running. 
-   **Main SRAM region :** The STM32F407VG SRAM address range checked above. The MCU also has other memory regions, so this range describes   the observed address rather than all memory on the chip. 
-   **Thread mode :**   The normal mode for running application code. Cortex-M also has Handler mode for exception and interrupt handlers. 
-   **MSP / PSP  Main Stack Pointer / Process Stack Pointer :**Cortex-M has two stack-pointer registers, but ordinary code sees the currently selected one through R13 (SP). 
-   **Word-aligned :**  An address aligned to a 4-byte boundary. `0x2001FFFC` is divisible by four. 


### 7. Questions I want to answer in upcoming practicals

- Why is SP currently `0x2001FFFC`?
- What exactly does PC point to while the processor is halted?
- How does PC change when I execute the next instruction?
- How does LR get updated during a function call?
- Can I observe PUSH and POP changing SP?
- Is the debugger showing MSP or PSP here, and how can I confirm it from CONTROL.SPSEL and the execution mode?
- What is the initial SP value in the vector table, and at what point does it change during startup?
- Which instructions are executing at the breakpoint, and how can I relate the disassembly to the C source?


