# SAP-1-Architecture-Logisim
## Table of Contents
Click on the Table of Contents below to directly go to the sections:

- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Key Features](#key-features)

- [Architecture and Functional Block Analysis](#architecture-and-functional-block-analysis)
  - [System Architecture Overview](#system-architecture-overview)
  - [Register Implementation (A, B)](#register-implementation-a-b)
  - [Program Counter (PC) Implementation](#program-counter-pc-implementation)
  - [Memory System and Address Register](#memory-system-and-address-register)
  - [Instruction Register and Opcode Decoder](#instruction-register-and-opcode-decoder)
  - [Arithmetic Logic Unit (ALU) Implementation](#arithmetic-logic-unit-alu-implementation)
  - [Boot/Loader Counter and Phase Generation](#bootloader-counter-and-phase-generation)

- [Control System Design](#control-system-design)
  - [Timing Control Generator](#timing-control-generator)
  - [Automatic Operation Control Logic](#automatic-operation-control-logic)
  - [Manual/Loader Operation Control](#manualloader-operation-control)

- [Instruction Set Architecture](#instruction-set-architecture)
  - [Instruction Encoding Scheme](#instruction-encoding-scheme)
  - [Assembler](#assembler)

- [Operation](#operation)
  - [Fetch–Decode–Execute Cycle](#fetchdecodeexecute-cycle)
  - [Running the CPU in Manual Mode](#running-the-cpu-in-manual-mode)
  - [Running the CPU in Automatic Mode (JMP + ADD Program)](#running-the-cpu-in-automatic-mode-jmp--add-program)

- [Future Improvement](#future-improvement)
- [Conclusion](#conclusion)

## Project Overview

This project implements an enhanced 8-bit **SAP-1** computer in **Logisim Evolution** with hardwired control and an extended instruction set (`LDA`, `LDB`, `ADD`, `SUB`, `STA`, `JMP`, `HLT`).  
It supports **Automatic Mode** (fetch-decode-execute cycle) and **Manual/Loader Mode** for program transfer.  
A **control sequencer** manages the bus and timing, while a web-based assembler converts assembly code into Logisim-compatible images.  
Test programs verified correct instruction execution and memory operations, making it a reliable framework for undergraduate learning in processor architecture.

## Objectives

- Develop an improved **SAP-1 (8-bit)** computer in **Logisim Evolution** for educational demonstration and system-level analysis.  
- Implement a classical **single-bus architecture** with 8-bit data path, 4-bit address space (16 bytes), and a **hardwired control sequencer** managing the fetch–decode–execute cycle.  
- Support **Automatic Mode** (six-stage ring counter T1–T6 with opcode decoder) and **Manual/Loader Mode** for safe program loading into RAM.  
- Design a datapath with **dual 8-bit registers** (accumulator and B register), a **ripple-carry ALU** (ADD/SUB), **4-bit program counter** with increment/load, **memory address register**, **16×8 SRAM**, and an instruction register (opcode + operand) while enforcing strict single-driver bus operation.

## Key Features

- Enhanced 8-bit **SAP-1** with a single-bus architecture (16-byte memory) and hardwired control.  
- Supports core instructions: `LDA`, `LDB`, `ADD`, `SUB`, `STA`, `JMP`, `HLT`.  
- Dual modes: **Automatic** (fetch-decode-execute) and **Manual/Loader** for safe program transfer.  
- Datapath: Accumulator & B registers, ripple-carry ALU, 4-bit PC, MAR, 16×8 SRAM, Instruction Register.  
- **Opcode decoder** and **control sequencer** manage timing and bus operation.  
- Lightweight web-based **assembler** converts assembly to Logisim HEX.  
- Step-by-step simulation ready with probes for registers, bus, ALU, and memory.  
- Easily **extensible** for new instructions or features.

## Control System Design

The control unit translates instructions into precisely timed control pulses to:  
1. Ensure exclusive bus driver activation.  
2. Enable appropriate latch operations during each T-state.  

Key components include:  
- **Ring Counter (RC)**: Generates timing states T1–T6.  
- **Opcode Decoder**: Produces activation lines for instruction-specific micro-operations.  
- **Mode Control Inputs**: `debug` (manual/loader selection), `i1/i2` (loader handshake), defining CPU mode (`~debug` with loader masking via `~i2`).  

The control sequencer operates in two paradigms:  
- **Automatic Execution Mode**: Optimized for fetch–decode–execute cycles.  
- **Manual/Loader Mode**: Optimized for safe program loading and debugging.  

Both modes maintain strict signal integrity and timing synchronization.

## Future Improvement

Potential directions for extending the current SAP-1 implementation include:  

- **Status Flags**: Introduce Zero (Z) and Carry (C) flags to support conditional branch instructions (e.g., `JZ`, `JC`) for advanced control flow.  
- **Enhanced Memory & Instructions**: Support multi-byte memory addressing, immediate data loading, and richer instruction formats.  
- **Microcoded Control Unit**: Enable systematic ISA expansion for new operations like shift and rotate instructions.  
- **Expanded Assembler**: Develop an assembler with symbolic labels, expressions, and enhanced directive support to improve programmability, usability, and instructional validation.

## Conclusion

The enhanced SAP-1 successfully bridges classical processor design with modern simulation-based education.  
With dual operational modes, expanded instruction support, and a structured control sequencer, the system demonstrates technical correctness and pedagogical clarity.  
Validation through test programs confirmed proper execution, control logic, and timing coordination.  
This project provides a practical, extensible platform for undergraduate learning in computer architecture and lays the foundation for future enhancements in processor design and research.
