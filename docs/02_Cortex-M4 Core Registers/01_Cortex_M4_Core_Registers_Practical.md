Cortex-M4 Core Registers — Practical Experiments

This document demonstrates the Cortex-M4 core registers through C, assembly, debugger inspection, and STM32F407
hardware experiments.

For complete theory explanation of M4 Cortex core registers, refer the file 01_Cortex_M4_Core_Registers.md[ here ](01_Cortex_M4_Core_Registers.md)

## Revision: Cortex-M4 Core Registers

### Topics revised
- R0–R12: General-purpose registers
- R13: Stack Pointer (SP)
- R14: Link Register (LR)
- R15: Program Counter (PC)
- MSP and PSP
- Function calls and nested function calls
- Exception stacking

### Key understanding
- BL stores the return address in LR and branches to the function.
- Nested function calls can overwrite LR, so it must be preserved.
- The stack grows towards lower memory addresses.
- Cortex-M4 automatically saves the basic exception stack frame.

### Practical work
Theory revision completed. Debugger experiments are pending.