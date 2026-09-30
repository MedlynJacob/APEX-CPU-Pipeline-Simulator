# ⚙️ APEX CPU Pipeline Simulator

### Out-of-Order CPU Pipeline Simulation in C

```text
┌──────────────────────────────────────────────┐
│        APEX CPU INITIALIZATION               │
├──────────────────────────────────────────────┤
│                                              │
│  Instructions:          LOADING...           │
│  Register Renaming:     ENABLED              │
│  Reservation Stations:  ONLINE               │
│  Reorder Buffer:        WATCHING EVERYTHING   │
│  Branch Predictor:      MAYBE RIGHT           │
│                                              │
└──────────────────────────────────────────────┘
```

> **SYSTEM STATUS: CPU HAS LOST THE PLOT**

Instructions are arriving faster than they can retire.

Branches are being predicted.
Registers are being renamed.
Instructions are executing out of order.
The ROB is keeping receipts.

Welcome to **APEX** — a cycle-by-cycle simulator of an out-of-order CPU pipeline built in C.

---

## > what is this?

APEX is a **C-based out-of-order CPU pipeline simulator** designed to model how modern processors handle instruction execution beyond simple sequential processing.

Instead of executing every instruction strictly in program order, the simulator models mechanisms that allow instructions to:

* Execute when their operands are ready
* Use register renaming to avoid false dependencies
* Wait in reservation stations
* Execute through different functional units
* Forward results between pipeline stages
* Maintain program-order retirement using a Reorder Buffer
* Handle memory operations through a Load/Store Queue
* Speculate on branches
* Recover from branch mispredictions

The simulator can be run **cycle by cycle**, making the internal state of the processor visible as instructions move through the pipeline.

---

## > mission status

```text
┌────────────────────────────────────────────────┐
│                 APEX STATUS                    │
├────────────────────────────────────────────────┤
│                                                │
│  Language              C                       │
│  Execution Model       Out-of-Order            │
│  Register Renaming     ✓                       │
│  Reservation Stations  ✓                       │
│  Reorder Buffer        ✓                       │
│  Load/Store Queue      ✓                       │
│  Data Forwarding       ✓                       │
│  Speculative Execution ✓                       │
│  Branch Prediction     ✓                       │
│  Pipeline Recovery     ✓                       │
│  Cycle Simulation      ✓                       │
│                                                │
└────────────────────────────────────────────────┘
```

---

## > architecture

The simulator models the major structures involved in an out-of-order processor.

```text
                    ┌──────────────┐
                    │   Program    │
                    │    Memory    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Fetch      │
                    │   F1 / F2    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Decode /     │
                    │ Rename       │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │ Reservation   │
                    │   Stations    │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        ┌─────────┐   ┌─────────┐   ┌─────────┐
        │ Integer │   │ Multiply│   │ Memory  │
        │   FU    │   │   FU    │   │   FU    │
        └────┬────┘   └────┬────┘   └────┬────┘
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    ┌──────────────┐
                    │     ROB      │
                    │  In-Order    │
                    │   Commit     │
                    └──────────────┘
```

---

## > processor organization

### Architectural Registers

The simulator maintains an architectural register file alongside renamed physical registers.

```text
Architectural Registers
        │
        ▼
      RAT
        │
        ▼
Physical Register File
```

The **Register Alias Table (RAT)** tracks the current physical register associated with each architectural register.

This allows multiple instructions to have independent physical destinations while preserving the architectural view of the program.

---

### > reservation stations

Instructions wait in reservation stations until their required operands become available.

The simulator maintains separate reservation structures for:

* Integer operations
* Multiply operations

Instructions can therefore wait independently rather than blocking the entire pipeline.

```text
Instruction
     │
     ▼
Reservation Station
     │
     ├── Source Ready? ──► Execute
     │
     └── Source Missing? ─► Wait
```

---

### > reorder buffer

The **Reorder Buffer (ROB)** allows instructions to execute out of order while still committing their architectural effects in program order.

```text
Execute Out of Order
        │
        ▼
┌─────────────────┐
│      ROB        │
│                 │
│  I1 → DONE      │
│  I2 → DONE      │
│  I3 → EXEC      │
│  I4 → WAIT      │
└────────┬────────┘
         │
         ▼
Commit In Order
```

This separates:

**execution order** from **retirement order**.

---

## > memory subsystem

Memory instructions are tracked through a **Load/Store Queue (LSQ)**.

The simulator models:

* Load operations
* Store operations
* Memory addresses
* Memory dependencies
* Multi-stage memory execution

The memory functional unit operates across multiple pipeline stages rather than completing a memory operation instantly.

---

## > execution model

The simulator advances one processor cycle at a time.

A cycle processes the pipeline through stages including:

```text
Data Forwarding
      ↓
ROB Commit
      ↓
Memory Execution
      ↓
Multiply Execution
      ↓
Integer Execution
      ↓
Instruction Issue
      ↓
Rename / Dispatch
      ↓
Decode / Rename
      ↓
Fetch
      ↓
Next Cycle
```

Different functional units can therefore be active simultaneously.

---

## > multi-cycle execution

Not every instruction takes the same amount of time.

The simulator models dedicated execution paths for different instruction classes.

```text
Integer FU
──────────
1-stage execution

Multiply FU
───────────
3-stage execution

Memory FU
──────────
2-stage execution
```

This allows the simulator to demonstrate how instructions overlap while different functional units operate concurrently.

---

## > data forwarding

Waiting instructions do not necessarily need to wait until a result reaches the architectural register file.

The simulator implements **data forwarding** so completed results can be propagated to dependent instructions.

```text
Instruction A
     │
     │ result
     ▼
Forwarding
     │
     ├──────────────► Instruction B
     │
     └──────────────► Instruction C
```

This models an important mechanism for reducing unnecessary pipeline stalls.

---

## > speculative execution

Branches introduce uncertainty.

Instead of waiting for every branch to resolve before fetching subsequent instructions, the simulator can speculate on the direction or target of control-flow instructions.

When speculation is incorrect, younger instructions are flushed and the processor state is restored.

```text
             Branch
               │
        ┌──────┴──────┐
        ▼             ▼
     Predict A      Predict B
        │             │
        └──────┬──────┘
               ▼
          Branch Resolves
               │
        ┌──────┴──────┐
        ▼             ▼
      Correct        Wrong
        │             │
        │          Flush + Recover
        │             │
        └──────┬──────┘
               ▼
          Continue
```

---

## > branch prediction

When enabled, the simulator maintains several structures for control-flow prediction.

### Branch Target Buffer — BTB

The BTB stores information about previously encountered conditional branches, including:

* Branch PC
* Target PC
* Branch history
* LRU information

### Call Target Predictor — CTP

The CTP tracks targets for call-style control-flow instructions.

### Return Address Prediction

A return-address stack is used to predict return targets.

Together, these structures allow the fetch stage to continue speculatively rather than waiting for every control-flow instruction to resolve.

---

## > branch recovery

A wrong prediction is not the end of the simulation.

When a misprediction is detected, the simulator can restore speculative processor state, including:

* RAT state
* Condition-code RAT state
* Free physical registers
* ROB state
* Younger instructions
* Pipeline state

The processor then resumes execution from the correct path.

```text
Wrong Prediction
       │
       ▼
Branch Resolves
       │
       ▼
Misprediction Detected
       │
       ▼
Restore Speculative State
       │
       ▼
Flush Younger Instructions
       │
       ▼
Correct PC
       │
       ▼
Continue Execution
```

---

## > supported instructions

The simulator supports a range of arithmetic, logical, memory, and control-flow instructions.

### Arithmetic / Logical

```text
ADD
SUB
MUL
ADDL
SUBL
AND
OR
XOR
```

### Comparison / Conditional

```text
CML
CMP
BZ
BNZ
BP
BN
```

### Memory

```text
LOAD
STORE
```

### Control Flow

```text
MOVC
JUMP
JAL
JALP
RET
```

### Pipeline Control

```text
NOP
HALT
```

---

## > cycle-by-cycle simulation

One of the main goals of the simulator is to make the pipeline state observable.

For every cycle, the simulator can display information such as:

```text
Cycle: 12
PC:    4040

F1:       ...
F2:       ...
D1/RN:    ...
RN2/DIS:  ...
IntFU:    ...
MulFU:    ...
MemFU:    ...

ROB:  2/16
LSQ:  0/6
```

It can also expose internal processor state including:

* Register Alias Table
* Architectural Register File
* Reservation Stations
* Reorder Buffer
* Load/Store Queue
* Predictor state
* Pipeline stalls
* Pipeline flushes

This makes it possible to follow an instruction from **fetch → rename → issue → execute → forward → commit**.

---

## > running the simulator

### 1. Clone the repository

```bash
git clone https://github.com/MedlynJacob/APEX-CPU-Pipeline-Simulator.git
cd APEX-CPU-Pipeline-Simulator
```

### 2. Compile

```bash
gcc -Wall -Wextra -o apex_sim main.c apex_cpu.c
```

### 3. Run without branch prediction

```bash
./apex_sim input.asm
```

### 4. Run with branch prediction

```bash
./apex_sim input.asm 1
```

The second argument enables the simulator's branch-prediction mechanisms.

---

## > simulator commands

Once the simulator starts, commands can be entered interactively.

```text
initialize
```

Initialize the CPU state.

```text
simulate
```

Run the simulation for the default number of cycles.

```text
simulate 10
```

Run exactly 10 cycles.

```text
display
```

Display the current processor and pipeline state.

```text
single_step
```

Advance the processor by one cycle.

```text
setmem <address> <value>
```

Set a value directly in data memory.

```text
setmem <file>
```

Load memory values from a file.

```text
exit
```

Terminate the simulator.

---

## > example program

The repository includes sample assembly programs for testing the simulator.

Example:

```asm
MOVC R0 #2
MOVC R1 #3
JALP R2 #12
MUL R0 R1 R0
HALT
NOP
RET R2
MOVC R1 #10
ADD R1 R1 R1
HALT
```

Run it with:

```bash
./apex_sim input.asm
```

Then inspect execution using:

```text
simulate
display
single_step
```

---

## > project structure

```text
APEX-CPU-Pipeline-Simulator/
│
├── apex_cpu.c       # CPU implementation and pipeline logic
├── apex_cpu.h       # CPU structures, constants, and declarations
├── main.c           # CLI interface and simulator entry point
│
├── input.asm        # Sample assembly program
├── input1.asm       # Additional test program
│
├── README.md
├── .gitignore
│
└── apex_sim         # Local executable (ignored by Git)
```

---

## > technical stack

```text
Language
└── C

Core Concepts
├── Out-of-Order Execution
├── Register Renaming
├── Reservation Stations
├── Reorder Buffer
├── Load/Store Queue
├── Speculative Execution
├── Branch Prediction
├── Data Forwarding
├── Pipeline Recovery
└── In-Order Retirement

Development
├── GCC
├── Git
└── Linux / macOS / Unix-like environments
```

---

## > project highlights

```text
[01] OUT-OF-ORDER EXECUTION
     Instructions can execute as operands become ready.

[02] REGISTER RENAMING
     Physical registers reduce false dependencies.

[03] SPECULATIVE EXECUTION
     The pipeline continues execution across predicted branches.

[04] BRANCH RECOVERY
     Incorrect speculation triggers state restoration and flushing.

[05] MULTI-CYCLE FUNCTIONAL UNITS
     Integer, multiply, and memory operations have
     different execution latencies.

[06] DATA FORWARDING
     Results can be forwarded directly to dependent instructions.

[07] CYCLE VISIBILITY
     Internal processor state can be inspected after each cycle.
```

---

## > why build a CPU simulator?

Because looking at:

```text
ADD R1 R2 R3
```

and knowing what the instruction *means* is very different from understanding what the processor actually has to do with it.

APEX was built to explore the machinery underneath instruction execution:

```text
Instruction
    ↓
Fetch
    ↓
Decode
    ↓
Rename
    ↓
Dispatch
    ↓
Wait for operands
    ↓
Issue
    ↓
Execute
    ↓
Forward
    ↓
Retire
```

And sometimes:

```text
        ↓
   "That branch was wrong."
        ↓
      FLUSH
        ↓
      RESTORE
        ↓
      TRY AGAIN
```

---

## > status

```text
┌──────────────────────────────────────────────┐
│               PROJECT STATUS                 │
├──────────────────────────────────────────────┤
│                                              │
│  CPU Pipeline              ✓ IMPLEMENTED     │
│  Register Renaming         ✓ IMPLEMENTED     │
│  Reservation Stations      ✓ IMPLEMENTED     │
│  Reorder Buffer            ✓ IMPLEMENTED     │
│  Load/Store Queue          ✓ IMPLEMENTED     │
│  Data Forwarding           ✓ IMPLEMENTED     │
│  Multi-Cycle Execution     ✓ IMPLEMENTED     │
│  Branch Prediction         ✓ IMPLEMENTED     │
│  Speculative Execution     ✓ IMPLEMENTED     │
│  Misprediction Recovery    ✓ IMPLEMENTED     │
│  Cycle-by-Cycle Debugging  ✓ IMPLEMENTED     │
│                                              │
│  STATUS: OPERATIONAL                        │
│                                              │
└──────────────────────────────────────────────┘
```

---

## > final transmission

```text
APEX CPU SIMULATOR
────────────────────────────────────

Instructions don't always execute
in the order you wrote them.

That's kind of the point.

        FETCH
          ↓
        RENAME
          ↓
        DISPATCH
          ↓
        EXECUTE ──────┐
          ↓           │
        FORWARD       │
          ↓           │
        COMMIT ◄──────┘

────────────────────────────────────
SYSTEM STATUS: OPERATIONAL
────────────────────────────────────
```

Built in C to explore the mechanics of out-of-order CPU execution.
