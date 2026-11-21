# INTRODUCTION TO LOGIC OPTIMIZATION
## Logic Optimization

Logic optimization is the process of improving a digital circuit's logic structure to achieve better **area, power consumption, performance (timing), and efficiency**, while preserving the original functionality.

It transforms a given logic design into an optimized implementation by reducing redundant logic, simplifying Boolean expressions, minimizing the number of gates, reducing unnecessary switching activity, and improving the circuit's timing paths.

Logic optimization is broadly classified into:

1. **Combinational Logic Optimization**
2. **Sequential Logic Optimization**

---

## 1. Combinational Logic Optimization

Combinational logic optimization improves circuits whose outputs depend only on their current inputs. It focuses on simplifying Boolean expressions, reducing redundant gates, minimizing logic depth, and improving signal propagation delay.

### 1.1 Constant Propagation

Constant propagation identifies signals whose values are permanently `0` or `1` and propagates those values throughout the logic network.

By replacing constant-valued signals and evaluating expressions containing them, unnecessary logic gates can be eliminated.

**Example:**

Before:
```text
Y = A AND 1
Z = B OR 0
```

After:
```text
Y = A
Z = B
```

This eliminates redundant gates and simplifies the circuit.

### 1.2 Boolean Logic Optimization

Boolean logic optimization simplifies logic expressions using Boolean algebra, logic identities, and techniques such as Karnaugh maps or automated logic minimization.

Its objective is to reduce the number of logic operations, redundant terms, and gate complexity while maintaining equivalent functionality.

**Example:**

Before:
```text
Y = (A AND B) OR (A AND C)
```

After applying Boolean factoring:
```text
Y = A AND (B OR C)
```

This can reduce hardware complexity by sharing common logic.

---
This is a small example of combinational logic optimization, and the image you shared is a synthesis-generated gate-level netlist (or synthesized logic view) showing how Yosys mapped your Verilog into a standard-cell gate.
The objective is to understand how a behavioral Verilog description is translated into a gate-level implementation and how synthesis tools optimize logic while preserving the original functionality.

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
```
read_verilog opt_check.v
```
```
synth -top opt_check
```
```
opt_clean -purge
```
```
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```
```
```
show
```
The opt_clean -purge command removes unused wires and cells, including certain unused internal signals and objects.


RTL Design
Verilog Code :
```
module opt_check (
    input a,
    input b,
    output y
);

   assign y = a ? b : 0;

endmodule
```

Description

The module opt_check consists of two inputs, a and b, and one output, y.

The output is assigned using the ternary conditional operator:

a ? b : 0

The ternary operator evaluates a condition and selects one of two values depending on whether the condition is true or false.

Syntax:

condition ? expression_if_true : expression_if_false

In this design:

a is the condition.

b is selected when a is logic HIGH (1).

0 is selected when a is logic LOW (0).

## Functional Analysis

The behavior of the circuit can be described using the following truth table:

| Input `a` | Input `b` | Output `y` | Explanation |
|:---------:|:---------:|:-----------:|-------------|
| 0 | 0 | 0 | Condition is false; output is 0 |
| 0 | 1 | 0 | Condition is false; output is 0 |
| 1 | 0 | 0 | Condition is true; output follows `b` (0) |
| 1 | 1 | 1 | Condition is true; output follows `b` (1) |

The output becomes HIGH only when both `a` and `b` are HIGH. This behavior is equivalent to a two-input AND gate.

The RTL is synthesized using Yosys. Synthesis converts the behavioral Verilog description into a structural representation of logic and maps the logic to cells available in the target technology library.

The target library used in the displayed netlist is the SkyWater SKY130 High-Density standard-cell library (sky130_fd_sc_hd).


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c288a1ef-2345-4b8a-8f68-0a9f72881331" />


This circuit is a combinational circuit because its output is determined entirely by the current values of its inputs.

This experiment demonstrates a basic example of combinational logic optimization using Verilog and Yosys. The conditional expression a ? b : 0 is simplified through Boolean algebra to the equivalent expression a & b.

During synthesis, the simplified logic is mapped to a two-input AND standard cell from the SkyWater SKY130 High-Density library. The generated netlist provides a structural representation of the implementation, including the AND gate, buffers, and signal interconnections.

## Similar examples :

# opt_check2

A multiplexer with one input tied to logic 1 simplifies to an OR gate, depending on the select and input connections.

```
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```
The ternary operator `a ? 1 : b` describes a 2-to-1 multiplexer:
* When `a = 1`, the output `y` is driven to `1`.
* When `a = 0`, the output `y` follows `b`.

### Truth Table

| `a` | `b` | `y` (`a ? 1 : b`) |
| :-: | :-: | :-: |
| 0 | 0 | **0** |
| 0 | 1 | **1** |
| 1 | 0 | **1** |
| 1 | 1 | **1** |

From the truth table, the expression simplifies directly to a **2-input OR gate**:
$$y = a + b$$
---

### Synthesized Netlist (`sky130`)

### Schematic Breakdown
* **Logic Simplification:** During synthesis, Yosys detects that a multiplexer with a constant `1` on its `true` branch is functionally identical to an OR gate, eliminating unnecessary hardware.
* **Standard Cell Mapping:** The simplified logic is mapped to the `sky130_fd_sc_hd__or2_0` cell (2-input OR gate from the SkyWater 130nm High-Density library).
* **Ports & Buffers (`BUF`):** Inputs `a` and `b` feed into pins `A` and `B` of the OR gate, and output `X` drives port `y`. The `BUF` blocks represent internal Yosys net-connection buffers generated for schematic visualization.

---
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/01c3a3ff-859e-4213-ae93-acaa26933a62" />


# opt_check3
## Logic Optimization with Yosys: `opt_check3`

This repository demonstrates how **Yosys** optimizes nested conditional (ternary) logic in Verilog into a single multi-input standard cell mapped to the **SkyWater 130nm (`sky130`)** standard cell library.

---

### Verilog RTL Code

```verilog
module opt_check3 (
  input  a,
  input  b,
  input  c,
  output y
);
  assign y = a ? (c ? b : 0) : 0;
endmodule
```

### How It Works
The nested ternary operators `a ? (c ? b : 0) : 0` describe two cascaded 2-to-1 multiplexers:
* **Outer condition (`a ? ... : 0`):** When `a = 0`, the output `y` is driven to `0`. When `a = 1`, `y` evaluates the inner condition.
* **Inner condition (`c ? b : 0`):** When `c = 0`, `y` is `0`. When `c = 1`, `y` follows `b`.

### Truth Table

| `a` | `b` | `c` | `y` (`a ? (c ? b : 0) : 0`) |
| :-: | :-: | :-: | :-: |
| 0 | 0 | 0 | **0** |
| 0 | 0 | 1 | **0** |
| 0 | 1 | 0 | **0** |
| 0 | 1 | 1 | **0** |
| 1 | 0 | 0 | **0** |
| 1 | 0 | 1 | **0** |
| 1 | 1 | 0 | **0** |
| 1 | 1 | 1 | **1** |

From the truth table, `y` is `1` **only when all three inputs (`a`, `b`, and `c`) are `1`**, simplifying directly to a **3-input AND gate**:
$$y = a \cdot b \cdot c$$

---
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/83a78857-e512-4a6f-a1c2-171232cfe806" />

### Synthesized Netlist (`sky130`)

### Netlist & Schematic Breakdown
* **Logic Simplification:** During synthesis, Yosys eliminates the two cascaded multiplexers with constant `0` branches and collapses the logic into a single 3-input gate.
* **Standard Cell Mapping:** The simplified logic is mapped to the `sky130_fd_sc_hd__and3_1` cell (3-input AND gate with drive strength `1` from the SkyWater 130nm High-Density library).
* **Pin Mapping:** Inputs `b`, `c`, and `a` connect to pins `A`, `B`, and `C` respectively, and output pin `X` drives port `y`.
* **Wire Buffers (`BUF`):** Pseudo-cells generated by Yosys's `show` command to visualize direct net connections between top-level ports and standard cell pins.


# opt_check4

## Logic Optimization with Yosys: `opt_check4`

This repository demonstrates how **Yosys** simplifies redundant nested conditional (ternary) logic and eliminates unused inputs (`b`), optimizing the circuit into a single 2-input XNOR gate mapped to the **SkyWater 130nm (`sky130`)** standard cell library.

---

## Verilog RTL Code

```verilog
module opt_check4 (
  input  a,
  input  b,
  input  c,
  output y
);
  assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

### How It Works
Evaluating the nested ternary expression algebraically reveals why input `b` has no effect on the output `y`:
* **When `a = 1`:** The inner expression evaluates `b ? (1 & c) : c`, which simplifies to `b ? c : c`. Regardless of whether `b` is `0` or `1`, the result is always **`c`**.
* **When `a = 0`:** The outer expression evaluates **`!c`**.

Combining both cases gives:
$$y = a \cdot c + \overline{a} \cdot \overline{c} = \overline{a \oplus c}$$

### Truth Table

| `a` | `b` | `c` | `y` (`a ? (b ? (a & c) : c) : (!c)`) |
| :-: | :-: | :-: | :-: |
| 0 | 0 | 0 | **1** |
| 0 | 0 | 1 | **0** |
| 0 | 1 | 0 | **1** |
| 0 | 1 | 1 | **0** |
| 1 | 0 | 0 | **0** |
| 1 | 0 | 1 | **1** |
| 1 | 1 | 0 | **0** |
| 1 | 1 | 1 | **1** |

From the truth table, `y` is `1` whenever `a` and `c` are equal (and is completely independent of `b`), simplifying directly to a **2-input XNOR gate**.

---

## Synthesized Netlist (`sky130`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/eb3aa250-f5f1-4584-8c76-f67a2170f391" />

### Netlist & Schematic Breakdown
* **Redundant Input Elimination (`b`):** Yosys recognizes that `y` is independent of `b`. Input port `b` is left unconnected to any logic gate and terminates at a dangling internal wire node (`$b`).
* **Standard Cell Mapping:** The simplified Boolean logic is mapped to a single `sky130_fd_sc_hd__xnor2_1` cell (2-input XNOR gate with drive strength `1` from the SkyWater 130nm High-Density library).
* **Pin Mapping:** Inputs `c` and `a` connect to pins `A` and `B` respectively, and output pin `Y` drives port `y`.
* **Wire Buffers (`BUF`):** Pseudo-cells generated by Yosys's `show` command to visualize direct net connections between top-level ports and standard cell pins.

# multiple_module_opt
# Hierarchical Logic Optimization with Yosys: `multiple_module_opt`

This repository demonstrates how **Yosys** performs dead-code (unused logic) elimination across hierarchical sub-modules and maps complex combinational expressions into compound standard cells in the **SkyWater 130nm (`sky130`)** library.

---

## Verilog RTL Code

```verilog
module sub_module1 (
  input  a,
  input  b,
  output y
);
  assign y = a & b;
endmodule

module sub_module2 (
  input  a,
  input  b,
  output y
);
  assign y = a ^ b;
endmodule

module multiple_module_opt (
  input  a,
  input  b,
  input  c,
  input  d,
  output y
);
  wire n1, n2, n3;

  sub_module1 U1 (.a(a),  .b(1'b1), .y(n1));
  sub_module2 U2 (.a(n1), .b(1'b0), .y(n2));
  sub_module2 U3 (.a(b),  .b(d),    .y(n3));

  assign y = c | (b & n1);
endmodule
```

### How It Works
The top-level module `multiple_module_opt` instantiates three sub-modules, though only one contributes to the final output:
* **`U1` (`sub_module1`):** Computes `n1 = a & 1'b1` (which simplifies logically to `a`) and drives wire `n1`.
* **`U2` (`sub_module2`):** Computes `n2 = n1 ^ 1'b0`. However, wire `n2` is never read anywhere in the module.
* **`U3` (`sub_module2`):** Computes `n3 = b ^ d`. Wire `n3` is also completely unused.
* **Output Logic (`y`):** Evaluates `y = c | (b & n1)`. Since `n1 = a`, the overall Boolean function is:
  $$y = c + (b \cdot a)$$

---

### Netlist & Schematic Breakdown

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/1060a99d-2c9f-48a4-b8f8-1053dc835740" />


* **Dead-Logic Elimination (`U2`, `U3`, and Port `d`):** Because wires `n2` and `n3` never reach an output port, Yosys completely removes instances `U2` and `U3` during synthesis optimization (`opt`). Consequently, input port `d` is left unconnected.
* **Sub-Module Hierarchy Preservation (`U1`):** Instance `U1` (`sub_module1`) remains intact as a hierarchical block because its output `n1` drives the output logic, and the design was not flattened (`flatten` was not run prior to `show`).
* **Compound Standard Cell Mapping (`$94`):** Instead of synthesizing separate 2-input AND and 2-input OR gates, `abc` maps `c | (b & n1)` into a single **And-Or compound cell**: `sky130_fd_sc_hd__a21o_1` (`X = (A1 & A2) | B1`).
* **Pin Mapping:**
  * Wire `n1` (from `U1.y`) -> Pin `A1`
  * Input `b` -> Pin `A2`
  * Input `c` -> Pin `B1`
  * Output Pin `X` -> Output port `y`
* **Wire Buffers (`BUF`):** Pseudo-cells generated by Yosys's `show` command to represent direct net connections cleanly in the schematic.


**Note:** Running `flatten` before `abc` dissolves `U1` (`a & 1'b1`) into a direct wire from `a` to `A1`, removing the `sub_module1` block completely.
## Flattened Netlist Optimization (`flatten`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/5ed47a0f-5c07-447c-818f-fa7014bb7e02" />

### Post-Flattening Netlist Breakdown
* **Sub-Module Dissolution (`U1` Removed):** Running `flatten` merges all sub-module logic into `multiple_module_opt`. Yosys propagates the constant `1'b1` into `U1` (`a & 1'b1 = a`), completely eliminating the `sub_module1` block and wiring port `a` directly to pin `A1`.
* **Hierarchical Wire Labels (Diamonds):** The diamond nodes (`U1.a`, `U1.y`, `U2.a`, `U2.y`, `U3.a`, `U3.b`, `n1`, `n2`) are retained internal net names from the flattened instances. Because none of these nets drive active logic, they terminate as unconnected endpoints.
* **Final Hardware Implementation:** The entire multi-module design reduces to a single `sky130_fd_sc_hd__a21o_1` standard cell (`X = (A1 & A2) | B1`), where:
  * Port `a` -> Pin `A1`
  * Port `b` -> Pin `A2`
  * Port `c` -> Pin `B1`
  * Pin `X` -> Output port `y`
 
# multiple_module_opt2
### Hierarchical Logic Optimization with Yosys: `multiple_module_opt2`

This repository demonstrates how preserving sub-module hierarchy prevents cross-boundary constant propagation in **Yosys**, keeping four cascaded 2-input AND sub-modules intact even when the overall Boolean function simplifies to a constant `0`.

---

### Verilog RTL Code

```verilog
module sub_module (
  input  a,
  input  b,
  output y
);
  assign y = a & b;
endmodule

module multiple_module_opt2 (
  input  a,
  input  b,
  input  c,
  input  d,
  output y
);
  wire n1, n2, n3;

  sub_module U1 (.a(a),  .b(1'b0), .y(n1));
  sub_module U2 (.a(b),  .b(c),    .y(n2));
  sub_module U3 (.a(n2), .b(d),    .y(n3));
  sub_module U4 (.a(n3), .b(n1),   .y(y));
endmodule
```

### How It Works
Each instance of `sub_module` implements a 2-input AND gate ($y = a \cdot b$):
* **`U1`:** Computes $n_1 = a \cdot 0 = 0$
* **`U2`:** Computes $n_2 = b \cdot c$
* **`U3`:** Computes $n_3 = n_2 \cdot d = b \cdot c \cdot d$
* **`U4`:** Computes $y = n_3 \cdot n_1 = (b \cdot c \cdot d) \cdot 0$

Algebraically, because $n_1$ is always `0`, the output $y$ is **always `0`** regardless of the values of `a`, `b`, `c`, or `d`:
$$y = (b \cdot c \cdot d) \cdot (a \cdot 0) = 0$$

---

### Synthesized Netlist (Hierarchical View)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/1f76439c-8bcf-47bc-888d-b9e69dcb29b8" />

### Netlist & Schematic Breakdown
* **Why All Four Sub-Modules Remain (`U1`–`U4`):** Because `flatten` was **not** executed, Yosys treats each `sub_module` instance as an isolated black-box boundary during top-level optimization. It does not propagate the constant `0` out of `U1` into `U4`.
* **Signal Flow:**
  * `U2` takes inputs `b` and `c` to drive wire `n2`.
  * `U3` takes wire `n2` and input `d` to drive wire `n3`.
  * `U1` takes input `a` and constant `1'b0` to drive wire `n1`.
  * `U4` combines `n3` and `n1` to drive the primary output port `y`.
* **What Happens After `flatten`:** If `flatten` and `opt` are run, Yosys collapses all four sub-modules, recognizes that `y = 0`, and ties output port `y` directly to a constant logic `1'b0` (`sky130_fd_sc_hd__conb_1` `LO` pin), eliminating every gate and input dependency.

### Flattened Netlist Optimization (`flatten`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/b8582fd0-047f-47af-8ad0-78b999eb49c2" />


### Post-Flattening Netlist Breakdown
* **100% Gate Elimination (`0` Standard Cells):** Once `flatten` dissolves the `sub_module` boundaries, Yosys propagates the constant `1'b0` from `U1` (`a & 1'b0 = 0`) into `U4` (`n3 & 0 = 0`). Because `y` is permanently `0`, **every single logic gate (`U1`, `U2`, `U3`, `U4`) is deleted**.
* **Constant Output Driver (`1'0 -> y`):** Look at the very top row of the schematic—a constant **`1'0`** feeds directly through `BUF` pseudo-cells and wire label `U4.y` straight into output port **`y`**.
* **Disconnected Primary Inputs (`a`, `b`, `c`, `d`):** Because `y` is always `0` regardless of input values, none of the four input ports (`a`, `b`, `c`, `d`) connect to any logic; they terminate at dangling internal wire labels (`$a`, `$b`, `$c`, `$d`, `U1.a`, `U2.a`, `U2.b`, `U3.b`).
* **Dangling Internal Nets (`n1`, `n2`, `n3`):** The intermediate wires (`U1.y -> n1 -> U4.b`, `U2.y -> n2 -> U3.a`, `U3.y -> n3 -> U4.a`) remain only as disconnected wire aliases from the flattened hierarchy because `opt_clean -purge` was not run.

## 2. Sequential Logic Optimization

Sequential logic optimization improves circuits containing flip-flops and registers, where the output depends on both current inputs and previously stored states.

It focuses on optimizing state representation, register placement, timing, and the overall structure of sequential circuits.


. What is a Flip-Flop?

All the earlier modules you ran (opt_check2, opt_check3, multiple_module_opt) were combinational logic—they have no memory. If you change an input, the output changes right away.
A flip-flop is sequential logic—it acts like a tiny, 1-bit storage box:
• It holds onto a single bit (0 or 1) and keeps outputting that value (Q) even after the input changes.
• Computers use millions of flip-flops together to build CPU registers, counters, and state machines.


What is a D Flip-Flop?

"D" stands for Data (or Delay). Think of a D flip-flop like a camera taking a snapshot:
• D (Data input): What is currently in front of the camera lens (0 or 1).
• CLK (Clock input): The shutter button. Specifically, a positive-edge (posedge) D flip-flop only takes a photo at the exact split-second the clock signal jumps from 0 to 1 (the rising edge).
• Q (Output): The printed photo on your screen. Between clock ticks, Q stays frozen on the last photo taken, ignoring any changes on D.
• RESET (Reset pin): A "wipe memory" button. In your code (always @(posedge clk, posedge reset)), it is asynchronous, meaning the instant reset jumps to 1, Q is immediately forced to 0 without waiting for a clock tick.

### Verilog RTL Code

```verilog
module dff_const1 (
  input      clk,
  input      reset,
  output reg q
);
  always @(posedge clk, posedge reset)
  begin
    if (reset)
      q <= 1'b0;
    else
      q <= 1'b1;
  end
endmodule
```

### How It Works
The `always` block describes a positive-edge-triggered D flip-flop with an asynchronous active-high reset:
* **When `reset = 1` (Asynchronous Reset):** Regardless of `clk`, output `q` is immediately cleared to `1'b0`.
* **When `reset = 0` (Normal Operation):** On the rising edge of `clk`, output `q` latches the constant value `1'b1`.

---
## RTL Simulation Waveform (`GTKWave`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/820edf9a-fe1d-4655-8eee-211642740cda" />


### Waveform Breakdown
* **Reset Asserted (`reset = 1`):** Whenever `reset` goes high (for example, around `1550 ns`), `q` immediately drops to `0` asynchronously.
* **Reset De-asserted (`reset = 0`):** Once `reset` returns to `0`, `q` stays `0` until the very next rising edge of `clk`, at which point `q` transitions to `1` and stays `1` until `reset` is asserted again.

---
## Synthesized Netlist (`sky130`)
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/896b1e2a-4a18-439f-b8b7-4c4b5adcb6af" />


### Netlist & Schematic Breakdown
* **Why the Flip-Flop Is NOT Removed:** Even though the data input `D` is tied to a constant `1'b1`, `q` is **not** a constant `1` because `reset` dynamically forces `q` to `0` and requires a clock edge to recover back to `1`. Therefore, sequential state storage is required.
* **Flip-Flop Cell (`$80`):** Mapped to `sky130_fd_sc_hd__dfrtp_1` (positive-edge-triggered D flip-flop with an asynchronous **active-low** reset `RESET_B`).
* **Reset Polarity Inversion (`$85`):** Because the Verilog RTL uses an **active-high** `reset` (`if (reset)`), while the `sky130` cell expects an **active-low** `RESET_B`, Yosys automatically inserts an inverter (`sky130_fd_sc_hd__clkinv_1`) between port `reset` and pin `RESET_B`.
* **Pin Mapping:**
  * Port `clk` -> Pin `CLK`
  * Constant `1'1` -> Pin `D`
  * Port `reset` -> Inverter (`sky130_fd_sc_hd__clkinv_1`) -> Pin `RESET_B`
  * Pin `Q` -> Output port `q`
## Testbench (`tb_dff_const1.v`)

```verilog
`timescale 1ns / 1ps

module tb_dff_const1;
  reg  clk, reset;
  wire q;

  dff_const1 uut (
    .clk(clk),
    .reset(reset),
    .q(q)
  );

  initial begin
    \$dumpfile("tb_dff_const1.vcd");
    \$dumpvars(0, tb_dff_const1);
    clk   = 0;
    reset = 1;
    #3000 \$finish;
  end

  always #10 clk     = ~clk;
  always #1547 reset = ~reset;
endmodule
```

### Key Timing Observations
* **Asynchronous Reset (`t = 0 ns` to `1547 ns`):** While `reset = 1`, `q` is held at `0` regardless of `clk` edges.
* **Synchronous Recovery (`t = 1547 ns` to `1550 ns`):** When `reset` de-asserts (`1 -> 0`) at `1547 ns`, `q` remains `0` until the next rising edge of `clk` at `1550 ns`, demonstrating why Yosys cannot optimize away the flip-flop.

### 2.1 State Optimization

State optimization reduces the number of states required in a finite state machine (FSM) or improves how states are encoded.

Equivalent or redundant states can be merged, and state encoding can be modified to reduce logic complexity or improve performance.

**Objectives:**
- Reduce the number of flip-flops where possible.
- Simplify next-state logic.
- Reduce area and power consumption.
- Improve state-transition implementation.

### 2.2 Retiming

Retiming is a sequential optimization technique that moves registers (flip-flops) across combinational logic boundaries without changing the overall input-output behavior or the intended cycle-level latency of the design.

It redistributes registers through a logic network to improve timing balance and reduce critical path delays.

**Objectives:**
- Improve clock frequency.
- Balance delays between pipeline stages.
- Reduce critical path delay.
- Improve timing performance without changing the overall computation.

### 2.3 Sequential Logic Cloning

Sequential logic cloning duplicates registers or sequential elements to reduce fanout, improve signal driving capability, and help meet timing constraints.

When a single register drives many loads or physically distant logic blocks, the synthesis or physical optimization tool may create copies of that register to distribute the load.

**Objectives:**
- Reduce fanout on individual registers.
- Improve signal propagation and timing.
- Reduce routing congestion in some cases.
- Improve physical implementation of high-load signals.

---

