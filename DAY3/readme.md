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


> **Note:** Running `flatten` before `abc` dissolves `U1` (`a & 1'b1`) into a direct wire from `a` to `A1`, removing the `sub_module1` block completely.



## 2. Sequential Logic Optimization

Sequential logic optimization improves circuits containing flip-flops and registers, where the output depends on both current inputs and previously stored states.

It focuses on optimizing state representation, register placement, timing, and the overall structure of sequential circuits.

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

