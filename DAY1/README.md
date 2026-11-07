# **Day 1: Introduction to Verilog RTL Design and Synthesis**

## Objective

The goal of Day 1 was to build a working understanding of the front half of the digital design flow: describing hardware in Verilog, verifying it through a testbench, simulating it with Icarus Verilog, inspecting waveforms in GTKWave, and finally turning the RTL into a gate-level netlist with Yosys using the SKY130 standard-cell library.

## Toolchain

| Tool | Role in the flow |
|------|------------------|
| **Verilog HDL** | Language used to describe the digital circuit at the register-transfer level |
| **Icarus Verilog (`iverilog`)** | Compiles the design and testbench into an executable simulation model |
| **GTKWave** | Displays the simulated signal activity stored in VCD files |
| **Yosys** | Synthesizes RTL into a gate-level netlist |
| **SKY130 standard-cell library** | Supplies the cell definitions and timing/electrical characteristics used during technology mapping |

---

## 1. Getting Started with Verilog RTL Design

### What is RTL design?

Register Transfer Level (RTL) design is a way of capturing a circuit's behavior by describing how data moves between registers and what logic transforms it along the way. It is written in a hardware description language such as Verilog, and the code is shaped by the functional specification the circuit must satisfy. In short: the spec says *what* the hardware should do, and the RTL expresses *how* it does it.

### What is a testbench?

A testbench is a verification environment wrapped around the design under test (DUT). It drives the DUT with input stimulus and watches the outputs, so you can confirm the design behaves as intended. A testbench is not synthesized into hardware; it exists purely to exercise and check the design.

### What is a simulator?

A simulator models how a Verilog design evolves over simulation time. It keeps track of scheduled events, and whenever a signal changes, it re-evaluates only the logic affected by that change. This event-driven approach is what lets a simulator reproduce circuit behavior efficiently.

For this workshop, **Icarus Verilog** is the simulator of choice.

---

## 2. Simulation Flow

The simulation process follows five steps:

1. **Supply the sources.** Pass the RTL design file and the testbench file to Icarus Verilog.
2. **Compile.** Icarus builds a simulation executable from the two files.
3. **Run.** Executing the compiled output applies the testbench stimulus to the design.
4. **Capture waveforms.** Dump commands inside the testbench write signal activity to a Value Change Dump (VCD) file.
5. **Inspect.** Open the VCD file in GTKWave and examine the signal transitions to verify functionality.

### Flow at a glance

```
RTL Design + Testbench
        │
        ▼
  Icarus Verilog
        │
        ▼
Simulation Executable
        │
        ▼
   VCD Waveform
        │
        ▼
     GTKWave
```

Or, in a single line:

`RTL Design + Testbench → Icarus Verilog → Simulation Executable → VCD Waveform → GTKWave`

---

## Key Takeaways

- RTL describes circuit behavior and data flow, guided by a functional specification.
- A testbench stimulates and observes the design but is never part of the final hardware.
- Simulation is event-driven: only logic affected by a change gets re-evaluated.
- The VCD file is the bridge between simulation and waveform analysis.
- Synthesis with Yosys and SKY130 then carries the verified RTL forward into a gate-level netlist.

---

**2. Lab 1: Repository Setup and RTL Simulation**
        Clone the Workshop Repository
```

git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop
ls -la

```

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/d389ba11-ae04-40ff-88c9-b84a3ac03be7" /> 
        View all the files and directories

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/565c1d53-141e-4ada-a075-bdb8f4871e8b" />

Simulate the 2:1 Multiplexer
Design file: good_mux.v
Testbench file: tb_good_mux.v

Compile the design and testbench:

```
iverilog good_mux.v tb_good_mux.v
```
A successful compilation generates the default executable a.out.

Execute the simulation:
```
./a.out
```
The testbench generates tb_good_mux.vcd 
Open the waveform:
```
gtkwave tb_good_mux.vcd
```
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/741faeb2-2461-49f9-9cfe-c6060e8db1dc" />
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/eec9a7af-0b30-4f6f-92ee-399a1f84fba5" />
The 2:1 multiplexer selects one of two inputs based on the select signal.

When sel = 0, output y follows input i0.
When sel = 1, output y follows input i1.

Simulation tells us whether the RTL behaves correctly. Synthesis is the next step: it takes that verified RTL and turns it into real hardware building blocks. The diagram below shows both halves of the flow side by side.

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/ebc896b5-4bca-4f8b-acf9-db6d8b9b87ee" />
**Complete flow diagram**
<img width="1383" height="886" alt="image" src="https://github.com/user-attachments/assets/39cb9bfd-2e64-47b1-9fcb-44d0ddf14b93" />
                                          

**3. Introduction to Yosys and Logic Synthesis**

What is a Synthesizer?

A synthesizer converts a synthesizable RTL description into a logic-level representation and can then map that logic onto cells from a chosen standard-cell library. It is the bridge between “what the circuit does” and “which physical gates implement it.”

In this workshop, Yosys plays that role.
Yosys is an open-source tool used for Verilog synthesis.

Yosys performs operations such as:

Reading Verilog RTL code
Analyzing and optimizing the design
Converting RTL into a logic representation
Technology mapping
Generating a gate-level netlist

What is Logic Synthesis?

Logic synthesis translates a behavioral or RTL description into a logic-level implementation. It unfolds in four stages:

Read and elaborate the RTL, resolving the design structure and hierarchy.
Optimize the logic, simplifying it to reduce redundancy.
Map the optimized logic to the standard cells that are available.
Generate a gate-level netlist describing the cells and their connections.
 RTL ──▶ Elaboration ──▶ Optimization ──▶ Technology Mapping ──▶ Gate-level Netlist

 What is a .lib File?

A Liberty (.lib) file is the library’s datasheet, written in a format tools can read. For every standard cell, it records:

Timing information (delays, transition behavior)
Area of the cell
Power characteristics
Pin details (direction, capacitance, function)'

What are Synthesis Constraints?

Constraints steer the synthesis tool toward the requirements the design must meet. For example, timing constraints describe the clocks and the assumed input and output timing. Beyond that, the chosen library and cell selections shape the final area, power and delay. Constraints are how you tell the tool what matters most.

Launching Yosys
```
yosys
```
This opens the interactive Yosys shell. Everything below is typed at its prompt.
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/e790f013-6d61-49e1-a39f-c5cb10e1d3c8" />


Step 1: Load the standard-cell library
```read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```

This introduces Yosys to the SKY130 cells (the typical-corner, 25 °C, 1.8 V characterization). The -lib option loads each cell as a black box, meaning Yosys learns the cell names and pins without needing their internal details.

Step 2: Load the RTL design
```read_verilog good_mux.v```
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/eeff3193-e051-4b49-8f3d-09b12bd8f027" />

Yosys parses the Verilog source and brings the good_mux module into its internal representation.

Step 3: Run generic synthesis
```synth -top good_mux```
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/64fccec1-9301-4577-9c2e-468ee7ec2570" />

Here -top good_mux tells Yosys which module is the top of the design. The command elaborates the design and applies generic optimizations, producing logic built from Yosys’s internal gate primitives, not yet from SKY130 cells.

Step 4: Map to SKY130 cells
```abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```

This is the technology-mapping stage. ABC takes the generic logic, optimizes it, and rebuilds it using real cells from the Liberty file. After this step, the design is expressed in SKY130 standard cells.
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/baae7e2e-0de6-412d-81d9-5166b1ed8c0c" />

Step 5: Look at the result
```show```
This draws the synthesized logic as a graph, so you can see which cells were chosen and how they are connected. For a 2-to-1 mux, you should see the select line steering between the two data inputs through a mux cell or an equivalent small group of gates.

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/f698a241-e85a-467c-8ed2-e35c1ce29b1d" />

Step 6: Generate the Netlist
```write_verilog -noattr good_mux_netlist.v```
The command writes the synthesized design to a Verilog netlist file.

To inspect it in GVim, if installed:

```!gvim good_mux_netlist.v```

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/efae5a37-6140-44b0-bb43-ee1abed66226" />

**4. Verification of synthesis**

What Is Verification?

Verification is the process of confirming that a design behaves according to its specification. It answers one question: does this circuit do what it was meant to do? In practice, you apply known inputs, observe the outputs, and check them against the expected behavior.
Verification happens at more than one stage of the flow:
RTL verification checks that the behavioral code matches the specification. This was our first simulation.
Gate-level verification checks that the synthesized netlist still matches that behavior. This is the stage shown in the diagram above.
The second check matters because synthesis rewrites the design. The netlist should do exactly what the RTL did, and verification is how we confirm it instead of assuming it.

<img width="1362" height="846" alt="image" src="https://github.com/user-attachments/assets/955e8f02-fa01-4920-8ee3-55c70ced6c2a" />

Why the Same Testbench Is Reused

Simulation and verification are not two different things
Simulation is the activity of running a design over time. Verification is the goal: proving the design does what the specification says. Simulation is the main tool for verification, so the same testbench serves both purposes. It is also reused at a second stage, after synthesis.
The simulation we ran earlier was done on the RTL. After synthesis, we simulate the gate-level netlist with the same testbench to confirm that synthesis did not change the behavior.
Why it works: the testbench only sees the ports
A testbench never looks inside the design. It drives the input ports and observes the output ports of the DUT, and nothing else. Yosys preserves the top module’s name and port list, so the netlist is a drop-in replacement for the RTL. From the testbench’s point of view, the two files are indistinguishable:





































