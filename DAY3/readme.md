# INTRODUCTION TO LOGIC OPTIMIZATION
# Logic Optimization

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

```read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```
```read_verilog opt_check.v```
```synth -top opt_check```
```opt_clean -purge```
```abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```
```show```
The opt_clean -purge command removes unused wires and cells, including certain unused internal signals and objects.


RTL Design
Verilog Code :
module opt_check (
    input a,
    input b,
    output y
);

   assign y = a ? b : 0;

endmodule

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

## 3. Functional Analysis

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

