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

## Architecture and Functional Block Analysis

### System Architecture Overview
The processor architecture uses a *unified single-bus design* with an *8-bit datapath* controlled by tri-state sources. Bus arbitration ensures that only *one driver* is active during each T-state, with possible drivers including pc_out, sram_rd, ins_reg_out_en, a_out, b_out, alu_out, and sh_out. Bus listener components such as mar_in_en, ins_reg_in_en, a_in, b_in, and sram_wr allow selective data capture when required.

![Automatic Mode Control Sequencer](images/fig1.png)
|:--:|
| *Figure 1: Automatic mode operation of the control sequencer showing fetch–decode–execute sequencing.* | 

![Manual/Loader Mode Control Sequencer](images/fig2.png)
|:--:|
|*Figure 2: Manual/Loader mode operation of the control sequencer showing secure program loading with debug and handshake signals.*|  

### Register Implementation (A, B)

The *A and B registers* use reg_gp modules to store *8-bit data*. They have three interfaces:

1. *Input:* Connected to the system bus; controlled by a_in and b_in signals.  
2. *Output:* Drives the bus via tri-state logic using a_out and b_out.  
3. *Internal:* Provides direct access to the *ALU* through reg_int_out without using the bus.

![A/B Register Subsystem](images/fig3.png)
|:--:|
|*Figure 3: A/B register subsystem with input, output, and internal interfaces.*|

### Program Counter (PC) Implementation

The *Program Counter (PC)* supports *dual modes* for sequential execution and program flow control:

- *Increment Mode:* At timing state T3, when pc_en = 1, the PC updates as PC ← PC + 1, enabling sequential instruction progression.  
- *Jump Mode:* During a JMP instruction at timing state T4, when jump_en = 1, the lower nibble of the Instruction Register (IR) is placed on the bus and loaded into the PC for direct program control transfer.  

*Bus Interface:* At timing state T1, when pc_out = 1, the current PC value is driven onto the system bus and loaded into the *Memory Address Register (MAR)* to start the instruction fetch cycle.
 
![Program Counter Increment/Jump](images/fig4.1.png)
|:--:|
|*Figure 4: Program Counter showing *increment and jump modes.*|

![Program Counter Direct Load](images/fig4.2.png)
|:--:|
|*Figure 5: Program Counter direct load and sequential functionality.*|

### Memory System and Address Register

The memory subsystem includes a *4-bit Memory Address Register (MAR)* which captures addresses from the system bus under the control of mar_in_en.

- *Instruction Fetching:* During the T1 phase, pc_out and mar_in_en load the Program Counter value into the MAR for instruction fetch.  
- *Operand Addressing:* During T4 of LDA, LDB, STA, or JMP instructions, ins_reg_out_en together with mar_in_en transfers the operand address (IR[3:0]) into the MAR.

The *SRAM* operates in two modes:  
- *Read Mode:* When sram_rd = 1, the value at RAM[MAR] is placed on the bus during T2 (instruction fetch) and T5 (LDA/LDB).  
- *Write Mode:* When sram_wr = 1, the data on the bus is written into RAM[MAR] during T5 of the STA instruction.

![Memory Element](images/fig5.png)
|:--:|
|*Figure 6: Register-based memory element showing data_in, wr_en, rd_en, clock, and chip select (cs) signals. Output to the bus is via data_out.*|

![Memory Subsystem](images/fig6.png)
|:--:|
|*Figure 7: Memory subsystem showing MAR operation and SRAM read/write timing.*|


### Instruction Register and Opcode Decoder

The *Instruction Register (IR)* has a dual role for *instruction storage* and *operand handling*:

1. *Instruction Loading:* During T2, signals sram_rd = 1 and ins_reg_in_en = 1 transfer IR ← M[MAR], capturing the instruction from memory.  
2. *Opcode Handling:* The upper nibble IR[7:4] goes to the *opcode decoder* (ins_tab), generating one-hot control signals for the decoded instruction.  
3. *Operand Handling:* The lower nibble IR[3:0] can be placed onto the system bus when ins_reg_out_en = 1 (typically during T4) to provide operand addresses or target values for memory and jump instructions.

The *opcode decoder (ins_tab)* implements a *4-to-16 decoding scheme*, producing one-hot signals such as insLDA, insLDB, insADD, insSUB, insSTA, insJMP, insHLT, with unused outputs reserved for future instructions.

![Instruction Register and Opcode Decoder](images/ir.png)
|:--:|
|*Figure 8: Architecture of the Instruction Register and opcode decoder, showing instruction loading, opcode routing, and operand forwarding.*|

### Arithmetic Logic Unit (ALU) Implementation

The *ALU subsystem* performs *8-bit arithmetic* with the following features:

- *Input Sources:* Operands come directly from A.reg_int_out and B.reg_int_out, bypassing the bus.  
- *Operation Control:* alu_sub = 1 selects subtraction (A − B); otherwise, addition (A + B) is performed.  
- *Execution Protocol:* During T4, with a_out = 1, b_out = 1, alu_out = 1, and a_in = 1, the ALU executes A ← A ± B in one step.  
- *Architectural Design:* Ripple-carry adder with mode control; simple hardware with linear carry propagation delay.  
- *Bus Interface:* ALU output drives the system bus only when alu_out = 1 via tri-state logic.

![ALU Implementation](images/fig8.png)
|:--:|
|*Figure 9: ALU implementation with ripple-carry architecture and tri-state bus interface.*|


### Boot/Loader Counter and Phase Generation

The *boot/loader subsystem (ins_loader)* securely transfers program data from ROM to RAM in *Manual/Loader mode*:

1. *Functional Role:* Loads programs from ROM to RAM when debug = 1, disabling normal fetch-execute logic.  
2. *Input Interface:* Accepts clk, bc_reset (counter reset), bc_en (count enable), and debug for mode selection.  
3. *Address Sequencing:* Uses a 4-bit CTR4 counter for upward counting, producing addresses bc_address[3:0] for systematic RAM writes.  
4. *Phase Control:* Generates two non-overlapping clock phases (Φ and ¬Φ) via a D flip-flop with feedback inversion to synchronize data transfer and prevent contention.

![Boot/Loader Subsystem](images/fig9.png)
|:--:|
|*Figure 10: Boot/loader subsystem with sequential address generation and dual-phase clocking.*|

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
![SAP-1 ControlSystem](images/fig10.png)
|:--:|
| *Figure 11: SAP-1 for Manual mode* |

The Manual Mode control sequencer provides a simplified execution pathway, supporting only the `ADD` instruction.  
This minimal configuration is intended for **step-wise verification** and **manual debugging** of instruction flow, allowing clear observation of **bus activity** and **control signal sequencing** during arithmetic operations.
![SAP-1 ControlSystem](images/fig11.png)
|:--:|
| *Figure 12: SAP-1 for Automatic mode* |

The Automatic Mode control sequencer manages the full **fetch–decode–execute cycle** and supports `ADD`, `SUB`, and `JMP` instructions.  
Instruction execution is orchestrated by combining **ring counter timing states** with **opcode decoding logic**, producing deterministic micro-operation sequences.  
This implementation ensures **efficient bus utilization** and **precise timing coordination** across all program phases.

## Timing Control Generator

The timing subsystem uses a **ring counter** generating six phases (T1–T6) to orchestrate the **fetch–decode–execute cycle**, ensuring deterministic micro-operation sequencing and synchronization across datapath components.

### Universal Fetch Sequence
For every instruction:  
- **T1**: Program Counter (PC) output enabled and captured by Memory Address Register (MAR ← PC).  
- **T2**: Memory read initiated (`sram rd`) and Instruction Register loaded (IR ← M[MAR]).  
- **T3**: Program Counter incremented (PC ← PC + 1).  

![SAP-1 TimingControlGenerator](images/fig12.png)
|:--:|
| *Figure 13: Six phase ring counter* |

### Representative Execute Sequences
- **LDA addr**:  
  - T4: Operand address (IR[3:0]) placed on bus and loaded into MAR.  
  - T5: Memory value read and latched into A register (A ← M[MAR]).  
- **ADD**:  
  - T4: A and B register outputs drive ALU; result fed back to A (alu_sub = 0).  
- **SUB**:  
  - T4: Similar to ADD but with subtraction enabled (alu_sub = 1, A ← A – B).  
- **JMP addr**:  
  - T4: Operand placed on bus and loaded into Program Counter (PC ← IR[3:0]).

## Automatic Operation Control Logic

The automatic operation control logic defines core minterms using:  

- **C = CPU ~debug**  
- **L = ~i2 (loader idle)**  

These signals coordinate the fetch–decode–execute cycle, ensuring precise timing and correct activation of micro-operations during automatic instruction execution.
![SAP-1 ALU](images/fig11.png)
|:--:|
| *Figure 14: Automatic mode control sequencer demonstrating full instruction set.* |

**Table 1: Fetch Control Equations**

| Signal          | Equation                                                       |
|-----------------|----------------------------------------------------------------|
| pc_out          | T1 & C                                                         |
| mar_in_en       | (T1 & C) \| (T4 & C & (insLDA \| insLDB \| insSTA \| insJMP))  |
| sram_rd         | (T2 & C) \| (T5 & C & (insLDA \| insLDB))                      |
| ins_reg_in_en   | T2 & C                                                         |
| pc_en           | T3 & C                                                         |

**Table 2: ALU and Register Control Equations**

| Signal  | Equation                                              |
|---------|------------------------------------------------------|
| alu_out | T4 & C & (insADD \| insSUB)                          |
| alu_sub | T4 & C & insSUB                                      |
| a_in    | (T5 & C & insLDA) \| (T4 & C & (insADD \| insSUB))   |
| b_in    | T5 & C & insLDB                                      |
| a_out   | (T4 & C & (insADD \| insSUB)) \| (T5 & C & insSTA)   |
| b_out   | T4 & C & (insADD \| insSUB)                          |

## Manual/Loader Operation Control

In Manual/Loader mode, the control logic uses `debug = 1` to **mask normal CPU fetch–decode–execute operations**, suspending standard execution for safe program loading.  

Program transfer follows a **handshake protocol**:  
- `i1` enables address placement into the Memory Address Register (MAR).  
- `i2` activates write operations into SRAM.  

This stepwise handshake ensures **safe memory loading** without bus contention, while debug masking prevents interference from normal CPU execution.
![SAP-1 ALU](images/fig9.png)
|:--:|
| *Figure 15: Architecture of the Manual/Loader control system.* |


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
