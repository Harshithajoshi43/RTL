**# Day 1: Introduction to Verilog RTL Design and Synthesis**

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
 ┌─────────────────────────── SIMULATION (does it work?) ───────────────────────────┐
 │                                                                                  │
 │   RTL Design (.v)  ─┐                                                            │
 │                     ├──▶  Icarus Verilog  ──▶  a.out  ──▶  VCD file  ──▶ GTKWave │
 │   Testbench (.v)   ─┘      (iverilog)        (vvp)       (dumpfile)   (waveforms)│
 │                                                                                  │
 └──────────────────────────────────────────────────────────────────────────────────┘
                                         │
                       RTL verified, so we move on to synthesis
                                         ▼
 ┌─────────────────────────── SYNTHESIS (what hardware is it?) ────────────────────┐
 │                                                                                  │
 │   RTL Design (.v)      ──▶  read_verilog  ─┐                                     │
 │                                            ├──▶  Yosys  ──▶ write_verilog ──▶ Netlist│
 │   Liberty file (.lib)  ──▶  read_liberty   ─┘   (synth + abc)                    │
 │                                                                                  │
 └──────────────────────────────────────────────────────────────────────────────────┘


**3. Introduction to Yosys and Logic Synthesis**







































