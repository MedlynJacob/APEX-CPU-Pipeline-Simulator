<div align="center">

# ⚙️ APEX CPU Pipeline Simulator

### *Out-of-Order CPU Pipeline Simulation in C*

> A cycle-by-cycle simulator where instructions execute out of order, branches make questionable decisions, and the ROB keeps everyone accountable.

**💻 [GitHub Repository](https://github.com/MedlynJacob/APEX-CPU-Pipeline-Simulator)**

<img src="https://readme-typing-svg.demolab.com?font=VT323&size=30&pause=1200&color=00D9FF&center=true&vCenter=true&width=700&lines=Booting+APEX...;Loading+Pipeline...;Initializing+ROB...;Predicting+Branches...;System+Ready" />

![C](https://img.shields.io/badge/C-Programming-blue?style=for-the-badge)
![CPU Architecture](https://img.shields.io/badge/CPU-Out--of--Order-orange?style=for-the-badge)
![Pipeline](https://img.shields.io/badge/Pipeline-Simulation-purple?style=for-the-badge)
![Branch Prediction](https://img.shields.io/badge/Branch-Prediction-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Operational-success?style=for-the-badge)

</div>

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

    > CPU HAS LOST THE PLOT
    

---

# > what is this?

**APEX** is a C-based **out-of-order CPU pipeline simulator** that models how instructions move through a modern-style processor.

Instead of simply executing instructions one after another, APEX models:

```text
Fetch → Decode/Rename → Dispatch → Issue → Execute → Forward → Commit
```

Instructions can execute when their operands are ready while the **ROB** ensures they eventually retire in program order.

---

# > architecture

```text
                  ┌─────────────┐
                  │    FETCH    │
                  └──────┬──────┘
                         ↓
                ┌─────────────────┐
                │ DECODE / RENAME │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │  RESERVATION    │
                │    STATIONS     │
                └────────┬────────┘
                         ↓
           ┌─────────────┼─────────────┐
           ↓             ↓             ↓
      ┌─────────┐   ┌─────────┐   ┌─────────┐
      │ Integer │   │  Mul FU  │   │ Memory  │
      │   FU    │   │          │   │   FU    │
      └────┬────┘   └────┬────┘   └────┬────┘
           └─────────────┼─────────────┘
                         ↓
                  ┌─────────────┐
                  │     ROB     │
                  │   COMMIT    │
                  └─────────────┘
```

---

# > core features

```text
✓ Out-of-order execution
✓ Register renaming
✓ Reservation stations
✓ Reorder Buffer (ROB)
✓ Load/Store Queue (LSQ)
✓ Data forwarding
✓ Multi-cycle execution
✓ Speculative execution
✓ Branch prediction
✓ Misprediction recovery
✓ Cycle-by-cycle pipeline visualization
```

---

# > speculation mode

Branches are where things get interesting.

APEX includes:

```text
Branch Target Buffer      → BTB
Call Target Predictor     → CTP
Return Address Prediction → RAP Stack
```

When a prediction is wrong:

```text
Branch
  ↓
Prediction
  ↓
Execute
  ↓
Wrong?
  ↓
FLUSH → RESTORE → RECOVER
  ↓
Continue
```

The simulator restores speculative state and resumes execution from the correct path.

---

# > execution model

Different functional units operate with different execution latencies:

```text
INTEGER      ████
MULTIPLY     ████████████
MEMORY       ████████
```

The simulator advances **one cycle at a time**, allowing the internal state of the processor to be inspected throughout execution.

```text
Cycle 12

F1       → ...
F2       → ...
D1/RN    → ...
RN2/DIS  → ...
IntFU    → ...
MulFU    → ...
MemFU    → ...

ROB      → 2 / 16
LSQ      → 0 / 6
```

---

# > instruction set

APEX supports arithmetic, logical, memory, and control-flow instructions.

```text
ARITHMETIC
ADD  SUB  MUL  ADDL  SUBL

LOGICAL
AND  OR  XOR  CML  CMP

MEMORY
LOAD  STORE

CONTROL FLOW
MOVC  JUMP  JAL  JALP  RET
BZ  BNZ  BP  BN

CONTROL
NOP  HALT
```

---

# > technology stack

```text
LANGUAGE
────────
C

CPU / ARCHITECTURE
──────────────────
Out-of-Order Execution
Register Renaming
Reservation Stations
Reorder Buffer
Load/Store Queue

EXECUTION
─────────
Integer FU
Multiply FU
Memory FU
Data Forwarding

SPECULATION
───────────
BTB
Call Target Predictor
Return Address Stack
Misprediction Recovery

TOOLS
─────
GCC
Git
Linux / macOS
```

---

# > project structure

```text
APEX-CPU-Pipeline-Simulator/
│
├── apex_cpu.c       # CPU & pipeline implementation
├── apex_cpu.h       # CPU structures & definitions
├── main.c           # CLI & simulator entry point
│
├── input.asm        # Sample program
├── input1.asm       # Additional test program
│
├── README.md
└── .gitignore
```

---

# > run it

### Compile

```bash
gcc -Wall -Wextra -o apex_sim main.c apex_cpu.c
```

### Run

```bash
./apex_sim input.asm
```

### Enable branch prediction

```bash
./apex_sim input.asm 1
```

### Interactive commands

```text
simulate
simulate 10
display
single_step
setmem <address> <value>
exit
```

---

# > mission status

```text
APEX CPU SIMULATOR
────────────────────────────────────

Pipeline                 ✓ ONLINE
Register Renaming        ✓ ONLINE
Reservation Stations     ✓ ONLINE
Reorder Buffer           ✓ ONLINE
Memory Pipeline          ✓ ONLINE
Data Forwarding          ✓ ONLINE
Branch Prediction        ✓ ONLINE
Speculative Execution    ✓ ONLINE
Recovery                 ✓ ONLINE
Cycle Simulation         ✓ ONLINE

STATUS: OPERATIONAL
```

---

<div align="center">

### ☕ Built with C, CPU architecture, and an unreasonable number of pipeline states.

```bash
> execute
> speculate
> mispredict
> recover
> repeat

Connection terminated.
```

</div>

