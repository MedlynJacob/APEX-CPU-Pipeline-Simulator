# ⚙️ APEX CPU Pipeline Simulator

### Out-of-Order CPU Pipeline Simulation in C

> A cycle-by-cycle CPU simulator implementing instruction fetch, register renaming, out-of-order execution, speculative execution, data forwarding, and in-order retirement.

---

```text
     █████╗ ██████╗ ███████╗██╗  ██╗
    ██╔══██╗██╔══██╗██╔════╝╚██╗██╔╝
    ███████║██████╔╝█████╗   ╚███╔╝
    ██╔══██║██╔═══╝ ██╔══╝   ██╔██╗
    ██║  ██║██║     ███████╗██╔╝ ██╗
    ╚═╝  ╚═╝╚═╝     ╚══════╝╚═╝  ╚═╝

    $ apex boot

    Initializing CPU...
    Loading instruction memory...
    Initializing register files...
    Initializing reorder buffer...
    Initializing reservation stations...
    Initializing load/store queue...
    Initializing branch predictor...

    CPU Ready.
```

---

# > what is this?

APEX CPU Pipeline Simulator is a C-based processor simulator that models an **out-of-order execution pipeline** cycle by cycle.

The simulator accepts an assembly program, fetches and decodes instructions, performs register renaming and dispatch, executes instructions through different functional units, forwards results to dependent instructions, and commits completed instructions through the reorder buffer.

The simulator also models speculative execution and branch prediction.

---

# > mission status

```text
    CORE PIPELINE
    -------------------------
    ✓ Instruction Fetch
    ✓ Instruction Decode
    ✓ Register Renaming
    ✓ Instruction Dispatch
    ✓ Out-of-Order Execution
    ✓ Data Forwarding
    ✓ In-Order Retirement


    CPU STRUCTURES
    -------------------------
    ✓ Architectural Register File
    ✓ Physical Register File
    ✓ Rename Table (RAT)
    ✓ Reorder Buffer (ROB)
    ✓ Reservation Stations
    ✓ Load/Store Queue (LSQ)
    ✓ Condition-Code Register File


    EXECUTION UNITS
    -------------------------
    ✓ Integer Functional Unit
    ✓ Multi-cycle Multiply Unit
    ✓ Multi-stage Memory Unit


    SPECULATION
    -------------------------
    ✓ Branch Target Buffer
    ✓ Call Target Predictor
    ✓ Return Address Prediction
    ✓ Branch Misprediction Recovery
    ✓ Pipeline Flushing
```

---

# > architecture

```text
                    Assembly Program
                           |
                           ▼
                    ┌─────────────┐
                    │     F1      │
                    │    Fetch    │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │     F2      │
                    │    Fetch    │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │    D1/RN    │
                    │Decode/Rename│
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   RN2/DIS   │
                    │   Dispatch  │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         ┌────────┐   ┌────────┐   ┌────────┐
         │ IntFU  │   │ MulFU   │   │ MemFU  │
         │        │   │ 3-cycle │   │2-stage │
         └────┬───┘   └────┬───┘   └────┬───┘
              │            │            │
              └────────────┼────────────┘
                           ▼
                    Data Forwarding
                           |
                           ▼
                    Reorder Buffer
                           |
                           ▼
                    In-Order Commit
```

---

# > processor organization

The simulator models several structures used by modern out-of-order processors.

```text
    Architectural Registers
              |
              ▼
        Rename Table
              |
              ▼
      Physical Registers
              |
       ┌──────┴──────┐
       ▼             ▼
 Reservation       Reorder
   Stations         Buffer
       |               |
       └───────┬───────┘
               ▼
        Functional Units
               |
               ▼
        Data Forwarding
               |
               ▼
         ROB Retirement
```

### Register Renaming

Instructions receive physical registers through the Rename Table and physical-register free list, allowing multiple instructions to execute without unnecessary false dependencies.

### Reservation Stations

Ready instructions wait in reservation stations until their source operands are available and the appropriate execution unit is free.

### Reorder Buffer

The ROB tracks in-flight instructions and allows execution to occur out of order while maintaining **in-order architectural retirement**.

### Load/Store Queue

Memory instructions are tracked through the LSQ to support address calculation and ordered memory operations.

---

# > execution model

Different instruction classes use different execution paths.

```text
    Integer Instructions
            |
            ▼
          IntFU
            |
            ▼
        Forwarding


    Multiply Instructions
            |
            ▼
       MulFU-1
            |
            ▼
       MulFU-2
            |
            ▼
       MulFU-3
            |
            ▼
        Forwarding


    Memory Instructions
            |
            ▼
       MemFU-1
            |
            ▼
       MemFU-2
            |
            ▼
        Forwarding
```

This allows the simulator to model different instruction latencies rather than treating every instruction as a single-cycle operation.

---

# > speculative execution

Branch prediction can be enabled when launching the simulator.

The simulator includes:

```text
    ✓ Branch Target Buffer (BTB)
    ✓ Call Target Predictor (CTP)
    ✓ Return Address Prediction (RAP)
    ✓ Branch History
    ✓ Misprediction Detection
    ✓ Pipeline Flush
    ✓ Rename-State Recovery
```

When a misprediction is detected, speculative state is discarded and the processor restores the appropriate rename and execution state before continuing from the correct control-flow path.

---

# > cycle-by-cycle simulation

The simulator exposes the internal processor state after each simulation cycle.

```text
+-----------------------------------------------------------------------------+
| Cycle: 12   | PC: 4040   | Stalled: NO   | Flushed: NO   | ROB: 2/16       |
+-----------------------------------------------------------------------------+
| STAGE   | INSTRUCTION                                                       |
+-----------------------------------------------------------------------------+
| F1      | ...                                                               |
| F2      | ...                                                               |
| D1/RN   | ...                                                               |
| RN2/DIS | ...                                                               |
| IntFU   | ...                                                               |
| MulFU-1 | ...                                                               |
| MulFU-2 | ...                                                               |
| MulFU-3 | ...                                                               |
| MemFU-1 | ...                                                               |
| MemFU-2 | ...                                                               |
+-----------------------------------------------------------------------------+
```

The display also exposes:

* Rename Table state
* Architectural registers
* Reservation stations
* Reorder Buffer
* Load/Store Queue
* Pipeline stalls and flushes
* Branch predictor state when enabled

This makes the simulator useful for observing how instructions move through an out-of-order processor over time.

---

# > supported instructions

The simulator supports arithmetic, logical, memory, control-flow, and system instructions including:

```text
    Arithmetic / Logical
    ---------------------
    ADD
    SUB
    MUL
    AND
    OR
    XOR
    ADDL
    SUBL
    CML
    CMP


    Data / Memory
    ---------------------
    MOVC
    LOAD
    STORE


    Control Flow
    ---------------------
    JUMP
    JAL
    JALP
    RET
    BZ
    BNZ
    BP
    BN


    Control
    ---------------------
    NOP
    HALT
```

---

# > running the simulator

### Compile

```bash
gcc -Wall -Wextra -o apex_sim main.c apex_cpu.c
```

### Run without branch prediction

```bash
./apex_sim input.asm
```

### Run with branch prediction

```bash
./apex_sim input.asm 1
```

---

# > simulator commands

Once the simulator starts:

```text
    simulate
    simulate 10
    display
    single_step
    setmem <address> <value>
    exit
```

### Example

```text
APEX CPU Initialized
--- PREDICTOR DISABLED ---

simulate 10

Cycle: 12
PC: 4040

F1      ...
F2      ...
D1/RN   ...
RN2/DIS ...
IntFU   ...
MulFU   ...
MemFU   ...

ROB: 2/16
LSQ: 0/6
```

`single_step` can be used to inspect the processor one cycle at a time.

---

# > project structure

```text
APEX-CPU-Pipeline-Simulator/
│
├── apex_cpu.c          # CPU implementation and pipeline logic
├── apex_cpu.h          # CPU structures and declarations
├── main.c              # Simulator entry point and CLI
│
├── input.asm           # Sample assembly program
├── input1.asm          # Additional test program
│
├── README.md
└── apex_sim             # Compiled executable (local only)
```

---

# > technical stack

```text
    Language
    --------
    C


    Processor Concepts
    ------------------
    Out-of-Order Execution
    Register Renaming
    Reservation Stations
    Reorder Buffer
    Load/Store Queue
    Speculative Execution
    Branch Prediction
    Data Forwarding
    In-Order Retirement


    Development
    ------------
    GCC
    Linux / Unix
    Command-Line Interface
```

---

# > project highlights

```text
    🧠 OUT-OF-ORDER EXECUTION
       Instructions can execute when their operands
       become available rather than strictly in program order.


    🔄 REGISTER RENAMING
       Physical registers and RAT-based renaming
       reduce false dependencies.


    📦 REORDER BUFFER
       Maintains precise architectural state while
       allowing speculative execution.


    ⚡ MULTI-CYCLE EXECUTION
       Different functional units model different
       execution latencies.


    🔀 BRANCH PREDICTION
       BTB, call-target prediction, and return-address
       prediction support speculative control flow.


    ↩ RECOVERY
       Mispredictions trigger pipeline flushing and
       restoration of speculative processor state.
```

---

# > status

```text
    Developer      : Medlyn Jacob
    Language       : C
    Project        : APEX CPU Pipeline Simulator
    Execution      : Out-of-Order
    Simulation     : Cycle-by-Cycle
    Predictor      : Optional
    Status         : RUNNING ✓
```

---

### ⚙️ Built with C, computer architecture, and an unreasonable number of pipeline states.

### 🚀 One cycle at a time.

```text
    $ exit

    Flushing pipeline...
    Saving processor state...
    Simulation terminated.

    Connection terminated.
```
