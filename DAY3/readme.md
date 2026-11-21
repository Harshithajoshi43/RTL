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

## Yosys Synthesis Flow

To reproduce this synthesis and view the schematic:

```tcl
# Read Verilog design
read_verilog dff_const1.v

# Synthesize design
synth -top dff_const1

# Map flip-flops and combinational logic to SkyWater 130nm library
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib

# Clean up and display schematic
clean
show
```

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

  
# dff_const2
# Sequential Constant Optimization with Yosys: `dff_const2`

This repository demonstrates how **Yosys** completely eliminates a D flip-flop when both its asynchronous reset state and its normal clocked data state evaluate to the same constant (`1'b1`), tying the output directly to logic high in the **SkyWater 130nm (`sky130`)** flow.

---

## Verilog RTL Code

```verilog
module dff_const2 (
  input      clk,
  input      reset,
  output reg q
);
  always @(posedge clk, posedge reset)
  begin
    if (reset)
      q <= 1'b1;
    else
      q <= 1'b1;
  end
endmodule
```

### How It Works
Unlike `dff_const1` (which toggled between `0` during reset and `1` after a clock edge), `dff_const2` assigns `1'b1` in **both** branches:
* **When `reset = 1`:** `q` is set to `1'b1`.
* **When `reset = 0`:** On the rising edge of `clk`, `q` is also set to `1'b1`.

Because `q` never changes state under any condition, no memory element (flip-flop) is needed.

---

## Testbench (`tb_dff_const2.v`)

```verilog
`timescale 1ns / 1ps

module tb_dff_const2;
  reg  clk, reset;
  wire q;

  dff_const2 uut (
    .clk(clk),
    .reset(reset),
    .q(q)
  );

  initial begin
    \$dumpfile("tb_dff_const2.vcd");
    \$dumpvars(0, tb_dff_const2);
    clk   = 0;
    reset = 1;
    #3000 \$finish;
  end

  always #10 clk     = ~clk;
  always #1547 reset = ~reset;
endmodule
```

---

## RTL Simulation Waveform (`GTKWave`)
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/1c96dd7e-e7a5-48e5-9f88-a1d6eff4a1a1" />


### Waveform Breakdown
* **Constant Output (`q = 1`):** From `t = 0 ns` through the entire simulation, `q` remains a flat line at logic `1`.
* **No Transition on Reset Toggle (`t = 1547 ns`):** Even when `reset` drops from `1` to `0` at `1547 ns` and `clk` continues toggling every `10 ns`, `q` never drops to `0` or changes value.

---

## Synthesized Netlist (`sky130`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/bb5b35a3-58a1-4123-a4df-6c4c9e2a2223" />

### Netlist & Schematic Breakdown
* **Flip-Flop Elimination (`0` Sequential Cells):** During synthesis (`opt`), Yosys detects that the flip-flop's reset value (`1'b1`) and data input (`1'b1`) are identical. The flip-flop is completely deleted.
* **Constant Logic Tie-High (`1'1 -> q`):** Output port `q` is wired directly to a constant logic `1'1` through a `BUF` pseudo-cell.
* **Disconnected Inputs (`clk` and `reset`):** Because `q` no longer depends on clock edges or reset assertions, both `clk` and `reset` remain unconnected floating ports in the schematic.

### Equivalent Gate-Level Verilog Netlist

```verilog
module dff_const2(clk, reset, q);
  input clk;
  input reset;
  output q;

  assign q = 1'b1;
endmodule
```

---
# dff_const3
### Sequential Constant Optimization with Yosys: `dff_const3`

This repository demonstrates how **Yosys** synthesizes two cascaded D flip-flops with mismatched reset and steady-state values, preserving both sequential elements and mapping them to reset-type and set-type standard cells in the **SkyWater 130nm (`sky130`)** library.

---

### Verilog RTL Code

```verilog
module dff_const3 (
  input      clk,
  input      reset,
  output reg q
);
  reg q1;

  always @(posedge clk, posedge reset)
  begin
    if (reset)
    begin
      q  <= 1'b1;
      q1 <= 1'b0;
    end
    else
    begin
      q1 <= 1'b1;
      q  <= q1;
    end
  end
endmodule
```

### How It Works
The design implements a 2-stage shift register (`1'b1 -> q1 -> q`) with an asynchronous active-high `reset`:
* **During Reset (`reset = 1`):** `q1` is asynchronously cleared to `1'b0`, while `q` is asynchronously preset to `1'b1`.
* **First Clock Edge After Reset (`reset = 0`):** Because non-blocking assignments (`<=`) sample old values before updating:
  * `q1` captures `1'b1`.
  * `q` captures the *previous* value of `q1`, which was `1'b0` during reset. As a result, `q` drops to `0` for exactly one clock cycle.
* **Second Clock Edge After Reset:**
  * `q1` remains `1'b1`.
  * `q` captures the updated value of `q1` (`1'b1`), returning to `1` and staying `1` until `reset` is asserted again.

Because `q` exhibits a single-cycle `1 -> 0 -> 1` pulse after `reset` de-asserts, **neither flip-flop can be optimized away**.

---

### Testbench (`tb_dff_const3.v`)

```verilog
`timescale 1ns / 1ps

module tb_dff_const3;
  // Inputs
  reg clk, reset;

  // Output
  wire q;

  // Instantiate the Unit Under Test (UUT)
  dff_const3 uut (
    .clk(clk),
    .reset(reset),
    .q(q)
  );

  initial begin
    \$dumpfile("tb_dff_const3.vcd");
    \$dumpvars(0, tb_dff_const3);
    // Initialize Inputs
    clk   = 0;
    reset = 1;
    #3000 \$finish;
  end

  always #10 clk     = ~clk;
  always #1547 reset = ~reset;
endmodule
```

### Testbench Line-by-Line Breakdown
* `` `timescale 1ns / 1ps ``: Sets the simulation time unit to `1 ns` and time precision to `1 ps`.
* **`reg clk, reset;` & `wire q;`**: `reg` types drive stimulus into the module's inputs, while `wire` monitors the module's output.
* **`dff_const3 uut (...)`**: Instantiates the Unit Under Test (`dff_const3`) and connects testbench signals to the module ports.
* **`$dumpfile` & `$dumpvars`**: Creates `tb_dff_const3.vcd` and records all signal transitions across all hierarchy levels (`0`), including the internal register `q1`, for viewing in GTKWave.
* **`clk = 0; reset = 1;`**: Starts the simulation with the clock low and asynchronous reset asserted.
* **`#3000 $finish;`**: Ends the simulation after `3000 ns` (`3 us`).
* **`always #10 clk = ~clk;`**: Toggles `clk` every `10 ns`, generating a square-wave clock with a **`20 ns` period** (`50 MHz`). Rising edges occur at `10 ns, 30 ns, 50 ns, ..., 1530 ns, 1550 ns, 1570 ns`.
* **`always #1547 reset = ~reset;`**: Toggles `reset` every `1547 ns` (deliberately chosen as an odd delay so reset de-assertion does not align with a `10 ns` clock edge).

---

### How the RTL Design and Testbench Work Together

Tracing the exact interaction between `tb_dff_const3` and `dff_const3` explains every transition visible in the GTKWave waveform:

1. **`t = 0 ns` to `1547 ns` (`reset = 1`):**
   * The testbench holds `reset` high.
   * Inside `dff_const3`, the `if (reset)` branch dominates regardless of the `20 ns` clock ticks, holding `q1 = 0` and `q = 1`.
2. **`t = 1547 ns` (`reset` falls `1 -> 0`):**
   * The testbench de-asserts `reset` mid-cycle (between the `1530 ns` and `1550 ns` rising clock edges).
   * Because `dff_const3` is edge-triggered on `posedge clk`, nothing changes yet: `q1` stays `0` and `q` stays `1` from `1547 ns` to `1550 ns`.
3. **`t = 1550 ns` (First `posedge clk` after reset release):**
   * Both flip-flops sample their inputs simultaneously:
     * `q1 <= 1'b1` (jumps from `0 -> 1`).
     * `q <= q1` (samples `q1`'s pre-clock value of `0`, so `q` drops from `1 -> 0`).
4. **`t = 1570 ns` (Second `posedge clk` after reset release):**
   * On the next rising edge `20 ns` later:
     * `q1 <= 1'b1` (stays `1`).
     * `q <= q1` (samples `q1`'s updated value of `1`, so `q` rises back from `0 -> 1`).

---

### RTL Simulation Waveform (`GTKWave`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/f9869d6d-84d3-4df9-a5ad-f7875464fbba" />


### Waveform Highlights
* **Marker at `1547 ns`:** Shows `reset` transitioning from `1` to `0` just before the `1550 ns` rising clock edge.
* **Single-Cycle Low Glitch on `q` (`1550 ns` to `1570 ns`):** Clearly shows `q1` rising to `1` at `1550 ns` while `q` drops to `0` for one full `20 ns` clock period before returning to `1` at `1570 ns`.

---

### Synthesized Netlist (`sky130`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/24d9ece7-153c-487c-97e8-c1233f1528a0" />


### Netlist & Schematic Breakdown
* **Two Distinct Flip-Flop Cells Preserved:**
  * **Stage 1 (`$82` -> `q1`):** Mapped to `sky130_fd_sc_hd__dfrtp_1` (positive-edge D flip-flop with active-low asynchronous **reset** `RESET_B`), because `q1` resets to `1'b0`. Its `D` pin is tied to `1'1`.
  * **Stage 2 (`$81` -> `q`):** Mapped to `sky130_fd_sc_hd__dfstp_2` (positive-edge D flip-flop with active-low asynchronous **set** `SET_B`), because `q` presets to `1'b1` during reset. Its `D` pin receives `q1`.
* **Reset Polarity Inverters (`$90` & `$89`):** Because the Verilog RTL uses an active-high `reset`, while the `sky130` flip-flops require active-low `RESET_B` and `SET_B` pins, Yosys inserts `sky130_fd_sc_hd__clkinv_1` inverters on the reset paths.
* **Shared Clock Network:** Port `clk` drives the `CLK` pins of both `$82` and `$81` simultaneously.

# dff_const4
## Sequential Constant Optimization with Yosys: `dff_const4`

This repository demonstrates how **Yosys** performs sequential constant propagation across cascaded D flip-flops when both registers reset to `1'b1` and are driven by `1'b1`, completely eliminating both flip-flops and tying the output permanently high in the **SkyWater 130nm (`sky130`)** flow.

---

### Verilog RTL Code

```verilog
module dff_const4 (
  input      clk,
  input      reset,
  output reg q
);
  reg q1;

  always @(posedge clk, posedge reset)
  begin
    if (reset)
    begin
      q  <= 1'b1;
      q1 <= 1'b1;
    end
    else
    begin
      q1 <= 1'b1;
      q  <= q1;
    end
  end
endmodule
```

### How It Works
The design describes a 2-stage shift register (`1'b1 -> q1 -> q`) where every reset and clocked assignment resolves to `1'b1`:
* **During Reset (`reset = 1`):** Both `q` and `q1` are asynchronously set to `1'b1`.
* **After Reset (`reset = 0`):** On every rising edge of `clk`:
  * `q1` captures `1'b1` (remaining `1`).
  * `q` captures `q1` (which was already `1` during reset and stays `1` forever).

Unlike `dff_const3` (where `q1` reset to `0`, causing a 1-cycle low pulse on `q`), **neither `q1` nor `q` ever leaves the `1'b1` state**.

---

### Testbench (`tb_dff_const4.v`)

```verilog
`timescale 1ns / 1ps

module tb_dff_const4;
  // Inputs
  reg clk, reset;

  // Output
  wire q;

  // Instantiate the Unit Under Test (UUT)
  dff_const4 uut (
    .clk(clk),
    .reset(reset),
    .q(q)
  );

  initial begin
    \$dumpfile("tb_dff_const4.vcd");
    \$dumpvars(0, tb_dff_const4);
    // Initialize Inputs
    clk   = 0;
    reset = 1;
    #3000 \$finish;
  end

  always #10 clk     = ~clk;
  always #1547 reset = ~reset;
endmodule
```

### Testbench Line-by-Line Breakdown
* `` `timescale 1ns / 1ps ``: Sets the simulation time unit to `1 ns` and resolution to `1 ps`.
* **`reg clk, reset;` & `wire q;`**: `reg` variables drive stimulus into the module inputs, while `wire` monitors the module output.
* **`dff_const4 uut (...)`**: Instantiates `dff_const4` as the Unit Under Test (`uut`) and maps the testbench signals to its ports.
* **`$dumpfile` & `$dumpvars`**: Dumps all signal changes across all hierarchy levels (`0`), including internal register `q1`, into `tb_dff_const4.vcd` for GTKWave visualization.
* **`clk = 0; reset = 1;`**: Initializes the simulation with `clk` at `0` and asynchronous `reset` active (`1`).
* **`#3000 $finish;`**: Stops the simulation at `3000 ns` (`3 us`).
* **`always #10 clk = ~clk;`**: Inverts `clk` every `10 ns`, producing a `20 ns` clock period (`50 MHz`) with rising edges at `10 ns, 30 ns, ..., 1530 ns, 1550 ns, 1570 ns`.
* **`always #1547 reset = ~reset;`**: Toggles `reset` every `1547 ns` so reset release falls asynchronously between clock edges.

---

### How the RTL Design and Testbench Work Together

1. **`t = 0 ns` to `1547 ns` (`reset = 1`):**
   * The testbench asserts `reset = 1`.
   * Inside `dff_const4`, the `if (reset)` block executes and sets both `q1 = 1` and `q = 1`.
2. **`t = 1547 ns` (`reset` drops `1 -> 0`):**
   * The testbench de-asserts `reset` between the `1530 ns` and `1550 ns` rising clock edges.
   * Both `q1` and `q` remain `1`.
3. **`t = 1550 ns` and Beyond (Subsequent `posedge clk` ticks):**
   * At `1550 ns` (and every `20 ns` thereafter), `q1` samples `1'b1` and `q` samples `q1` (`1'b1`).
   * Because both registers are already `1`, no signal transition occurs at `1550 ns`, `1570 ns`, or anywhere else in the simulation.

---

## RTL Simulation Waveform (`GTKWave`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/20da3f56-b05b-41a0-bfc7-169efe3ec24b" />



### Waveform Highlights
* **Flat High Signals (`q = 1`, `q1 = 1`):** Both `q` and internal register `q1` remain steady at logic `1` across the entire timeline.
* **Marker at `1547 ns`:** Confirms that when `reset` falls from `1` to `0` prior to the `1550 ns` clock edge, neither `q1` nor `q` glitches or changes state.

---

## Synthesized Netlist (`sky130`)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/eb41517c-be6c-4fc0-b3bb-2bbdb6d9157d" />


### Netlist & Schematic Breakdown
* **Both Flip-Flops Eliminated (`0` Sequential Cells):** During synthesis optimization (`opt`), Yosys detects that `q1` is permanently `1'b1` (its reset value `1'b1` matches its data input `1'b1`). Once `q1` is reduced to a constant `1'b1`, the second flip-flop (`q`) also has identical reset (`1'b1`) and data (`1'b1`) values, so Yosys deletes **both** flip-flops.
* **Constant Logic Tie-High (`1'1 -> q` and `1'1 -> q1`):**
  * Output port `q` is driven directly by a constant `1'1` through a `BUF` pseudo-cell.
  * Internal wire `q1` (shown as a diamond node) is also tied directly to `1'1` through a `BUF` pseudo-cell.
* **Disconnected Inputs (`clk` and `reset`):** Because no sequential cells remain in the design, ports `clk` and `reset` are completely unconnected floating octagons in the schematic.

# dff_const5
## Sequential Constant Optimization with Yosys: `dff_const5`

This repository demonstrates how **Yosys** synthesizes a 2-stage shift register (`1'b1 -> q1 -> q`) when both flip-flops reset to `1'b0` and shift in a constant `1'b1`, preserving both sequential elements to maintain the exact two-clock-cycle propagation delay after reset release.

---

### Verilog RTL Code

```verilog
module dff_const5 (
  input      clk,
  input      reset,
  output reg q
);
  reg q1;

  always @(posedge clk, posedge reset)
  begin
    if (reset)
    begin
      q  <= 1'b0;
      q1 <= 1'b0;
    end
    else
    begin
      q1 <= 1'b1;
      q  <= q1;
    end
  end
endmodule
```

### How It Works
The design implements a 2-stage shift register (`1'b1 -> q1 -> q`) with an asynchronous active-high `reset`:
* **During Reset (`reset = 1`):** Both `q1` and `q` are asynchronously cleared to `1'b0`.
* **First Clock Edge After Reset (`reset = 0`):** Because non-blocking assignments (`<=`) evaluate right-hand values before updating:
  * `q1` captures `1'b1` (transitioning `0 -> 1`).
  * `q` captures the *previous* value of `q1` (`1'b0`), remaining `0` for one full clock cycle.
* **Second Clock Edge After Reset:**
  * `q1` stays `1'b1`.
  * `q` captures the updated value of `q1` (`1'b1`), finally transitioning `0 -> 1` two clock edges after reset de-asserts.

Because `q` is delayed by two rising clock edges relative to reset release, **neither flip-flop can be eliminated**.

---

### Testbench (`tb_dff_const5.v`)

```verilog
`timescale 1ns / 1ps

module tb_dff_const5;
  // Inputs
  reg clk, reset;

  // Output
  wire q;

  // Instantiate the Unit Under Test (UUT)
  dff_const5 uut (
    .clk(clk),
    .reset(reset),
    .q(q)
  );

  initial begin
    \$dumpfile("tb_dff_const5.vcd");
    \$dumpvars(0, tb_dff_const5);
    // Initialize Inputs
    clk   = 0;
    reset = 1;
    #3000 \$finish;
  end

  always #10 clk     = ~clk;
  always #1547 reset = ~reset;
endmodule
```

### Testbench Line-by-Line Breakdown
* `` `timescale 1ns / 1ps ``: Defines `1 ns` as the simulation time unit and `1 ps` as the timing precision.
* **`reg clk, reset;` & `wire q;`**: `reg` types drive stimulus into the inputs, while `wire` monitors the output.
* **`dff_const5 uut (...)`**: Instantiates `dff_const5` as the Unit Under Test (`uut`) and maps testbench signals to the module ports.
* **`$dumpfile` & `$dumpvars`**: Creates `tb_dff_const5.vcd` and records all signal transitions across all hierarchy levels (`0`), including internal register `q1`, for GTKWave inspection.
* **`clk = 0; reset = 1;`**: Starts the simulation with `clk` low and asynchronous `reset` asserted (`1`).
* **`#3000 $finish;`**: Terminates the simulation after `3000 ns` (`3 us`).
* **`always #10 clk = ~clk;`**: Toggles `clk` every `10 ns`, creating a `20 ns` clock period (`50 MHz`) with rising edges at `10 ns, 30 ns, ..., 1530 ns, 1550 ns, 1570 ns`.
* **`always #1547 reset = ~reset;`**: Toggles `reset` every `1547 ns` so reset de-assertion occurs asynchronously between rising clock edges.

---

### How the RTL Design and Testbench Work Together

Tracing the interaction between `tb_dff_const5` and `dff_const5` explains the two-stage delayed rise seen in GTKWave:

1. **`t = 0 ns` to `1547 ns` (`reset = 1`):**
   * The testbench holds `reset` high.
   * Inside `dff_const5`, `if (reset)` overrides the clock, holding both `q1 = 0` and `q = 0`.
2. **`t = 1547 ns` (`reset` falls `1 -> 0`):**
   * The testbench de-asserts `reset` between the `1530 ns` and `1550 ns` rising clock edges.
   * Because the flip-flops only sample on `posedge clk`, both `q1` and `q` stay `0` from `1547 ns` to `1550 ns`.
3. **`t = 1550 ns` (First `posedge clk` after reset release):**
   * `q1 <= 1'b1` (`q1` jumps from `0 -> 1`).
   * `q <= q1` (`q` samples `q1`'s old value of `0`, so `q` stays `0`).
4. **`t = 1570 ns` (Second `posedge clk` after reset release):**
   * `q1 <= 1'b1` (`q1` stays `1`).
   * `q <= q1` (`q` samples `q1`'s new value of `1`, so `q` jumps from `0 -> 1` exactly one clock period after `q1`).

---

### RTL Simulation Waveform (`GTKWave`)
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/aec00ad0-a618-4c99-9d24-088e06882615" />


### Waveform Highlights
* **Marker at `1547 ns`:** Marks `reset` dropping from `1` to `0` prior to the `1550 ns` rising clock edge.
* **Staggered Rising Transitions (`1550 ns` and `1570 ns`):** `q1` transitions `0 -> 1` on the first rising edge at `1550 ns`, and `q` transitions `0 -> 1` one full `20 ns` clock cycle later at `1570 ns`.

---

### Synthesized Netlist (Generic Yosys Cells)

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/85b6f4ca-ad5b-4296-b708-ad268fb485c9" />


### Netlist & Schematic Breakdown
* **Two Generic Flip-Flop Cells Preserved (`$_DFF_PP0_`):**
  * Notice that `show` was executed right after `synth` (before `dfflibmap` technology mapping to `sky130`), so Yosys displays its internal generic flip-flop primitive `$_DFF_PP0_` (**P**ositive-edge `C`, **P**ositive/active-high asynchronous reset `R`, resetting to **`0`**).
  * **Stage 1 (`$82` -> `q1`):** Receives `1'1` on `D`, `clk` on `C`, and `reset` on `R`, driving internal net `q1`.
  * **Stage 2 (`$81` -> `q`):** Receives `q1` on `D`, `clk` on `C`, and `reset` on `R`, driving output port `q`.
* **Technology Mapping Note (`sky130`):** When `dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib` is run, each `$_DFF_PP0_` maps to `sky130_fd_sc_hd__dfrtp_1` alongside a `sky130_fd_sc_hd__clkinv_1` inverter on `reset` to convert active-high `reset` into active-low `RESET_B`.


### Sequential Constant Optimization with Yosys (`dff_const1` – `dff_const5`)

This repository explores how **Yosys** analyzes sequential logic (D flip-flops) with constant data inputs and asynchronous resets, demonstrating the exact conditions under which flip-flops are **preserved** versus **optimized away** during synthesis.

---

### Summary Comparison: When Does Yosys Keep vs. Delete Flip-Flops?

**Core Rule:** Yosys deletes a flip-flop **only when its reset state and its clocked steady-state are identical** (meaning the output `q` never changes value under any sequence of `reset` or `clk` events). If `q` changes state even for a single clock cycle, Yosys preserves the required flip-flop(s).

| Module | Stages | Reset State (`reset = 1`) | Clocked Behavior (`reset = 0`) | Waveform Behavior of `q` | Flip-Flops Kept | Synthesized Hardware (`sky130` / Generic) | Why? |
| :--- | :---: | :--- | :--- | :--- | :---: | :--- | :--- |
| **`dff_const1`** | 1 | `q <= 0` | `q <= 1` | `0` during reset $\rightarrow$ jumps to `1` on 1st `posedge clk` | **1** | `1x dfrtp_1` + `1x clkinv_1` | Output transitions `0 -> 1`; state memory is required. |
| **`dff_const2`** | 1 | `q <= 1` | `q <= 1` | Constant `1` forever (never changes) | **0** | Constant `1'b1` (`BUF` to `q`) | Reset value (`1`) equals clocked value (`1`); flip-flop is redundant. |
| **`dff_const3`** | 2 | `q1 <= 0`<br>`q  <= 1` | `q1 <= 1`<br>`q  <= q1` | `1` during reset $\rightarrow$ drops to `0` on 1st `posedge clk` $\rightarrow$ returns to `1` on 2nd `posedge clk` | **2** | `1x dfrtp_1` (`q1`) + `1x dfstp_2` (`q`) + `2x clkinv_1` | `q` produces a 1-cycle low pulse (`1 -> 0 -> 1`) after reset release. |
| **`dff_const4`** | 2 | `q1 <= 1`<br>`q  <= 1` | `q1 <= 1`<br>`q  <= q1` | Constant `1` forever (`q1` and `q` never change) | **0** | Constant `1'b1` (`BUF` to `q` and `q1`) | `q1` is constant `1`, which makes `q` constant `1`; both flip-flops are deleted. |
| **`dff_const5`** | 2 | `q1 <= 0`<br>`q  <= 0` | `q1 <= 1`<br>`q  <= q1` | `0` during reset $\rightarrow$ `q1` rises on 1st `posedge clk` $\rightarrow$ `q` rises on 2nd `posedge clk` | **2** | `2x $_DFF_PP0_` (or `2x dfrtp_1` + inverters) | Both flip-flops are needed to create the 2-clock-cycle delay (`0 -> 0 -> 1`). |

---

## Module-by-Module Breakdown

### 1. `dff_const1` (1 Flip-Flop Kept)
* **RTL Logic:** Asynchronous reset sets `q <= 0`; clocked branch sets `q <= 1`.
* **Synthesis Result:** Yosys keeps the flip-flop (`sky130_fd_sc_hd__dfrtp_1`) with `D = 1'b1` and adds an inverter (`sky130_fd_sc_hd__clkinv_1`) to convert active-high `reset` into active-low `RESET_B`.
* **Takeaway:** A constant `D` input does **not** automatically delete a flip-flop if the reset state differs from `D`.

### 2. `dff_const2` (0 Flip-Flops Kept)
* **RTL Logic:** Asynchronous reset sets `q <= 1`; clocked branch also sets `q <= 1`.
* **Synthesis Result:** The flip-flop is completely eliminated, and `q` is tied directly to `1'b1`. Both `clk` and `reset` become unconnected ports.
* **Takeaway:** When reset and clocked assignments always resolve to the same constant, sequential logic collapses into a static wire.

### 3. `dff_const3` (2 Flip-Flops Kept — Mixed Set/Reset)
* **RTL Logic:** Reset sets `q1 <= 0` and `q <= 1`; clocked branch shifts `1'b1 -> q1 -> q`.
* **Synthesis Result:** Yosys maps `q1` to an active-low reset flip-flop (`dfrtp_1`) and `q` to an active-low set flip-flop (`dfstp_2`).
* **Takeaway:** Even though `q` starts at `1` and ends at `1`, it dips to `0` for one clock cycle after reset release (`1550 ns` to `1570 ns`), forcing Yosys to preserve both stages.

### 4. `dff_const4` (0 Flip-Flops Kept — Cascaded Constant Propagation)
* **RTL Logic:** Reset sets `q1 <= 1` and `q <= 1`; clocked branch shifts `1'b1 -> q1 -> q`.
* **Synthesis Result:** Yosys first optimizes `q1` to a constant `1'b1` (identical to `dff_const2`), then propagates that constant into the second stage, eliminating `q`'s flip-flop as well.
* **Takeaway:** Constant optimization propagates stage-by-stage through shift registers until all redundant sequential elements are removed.

### 5. `dff_const5` (2 Flip-Flops Kept — 2-Cycle Shift Delay)
* **RTL Logic:** Reset sets `q1 <= 0` and `q <= 0`; clocked branch shifts `1'b1 -> q1 -> q`.
* **Synthesis Result:** Both flip-flops are preserved (`$_DFF_PP0_` post-`synth`, or `dfrtp_1` post-`dfflibmap`).
* **Takeaway:** Both registers are required so `q` waits two rising clock edges (`1550 ns` and `1570 ns`) after reset release (`1547 ns`) before transitioning from `0` to `1`.

---

### Common Testbench Setup (`tb_dff_const1` – `tb_dff_const5`)

All five modules are verified using the same asynchronous stimulus pattern:
* **Clock (`clk`):** `always #10 clk = ~clk;` generates a `20 ns` period (`50 MHz`) clock with rising edges at `10 ns, 30 ns, ..., 1530 ns, 1550 ns, 1570 ns`.
* **Asynchronous Reset (`reset`):** Initialized to `1` and toggled via `always #1547 reset = ~reset;`. Using `1547 ns` ensures reset de-asserts asynchronously mid-cycle (`3 ns` before the `1550 ns` rising clock edge), clearly separating asynchronous reset behavior from synchronous clock-edge sampling in GTKWave.

---
















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

