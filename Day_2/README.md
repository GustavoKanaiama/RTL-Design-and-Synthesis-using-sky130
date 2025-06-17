# Day 2 - Timing libs, hierarchical vs flat synthesis and efficient flop coding styles
Topics:
- [Introduction to timing .libs](#introduction-to-timing-libs)
- [Hierarchical vs Flat Synthesis](#hierarchical-vs-flat-synthesis)
- [Various Flop Coding Styles and optimization](#various-flop-coding-style-optimizations)

## Introduction to timing .libs

Analyzing the file `.lib`: 

If you want, open with a text editor:
`/address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib`.

It's important to pay attention in these three parts for the design tool work:

P - Process(can vary due to fabrication), V - Voltage, T - Temperature.

These three characteristics determines whether the silicon(semiconductor) will work faster, slower, etc.

It's possible to find these three information on the top of the `.lib` document:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img1.png"
  />
</p>

Also in this document It's possible to analyze the _cells_ objects. The _cells_ indicates the specific the physical characteristics of the type of gates, as It possible to see in the image, It's a specification of an AND/OR gate, with different type of connections (Output = (A1 & A2) | B1 | C1 | D1):

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img2.png", width=0.8
  />
</p>

Then, entering in the `behavioral.v` file of the specific gate (mentioned above) It's possible to see information about the power, timing, and area occupied:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img3.png"
  />
</p>

Comparison of different types of AND gate and It's areas:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img4.png"
  />
</p>

## Hierarchical vs Flat Synthesis

### Hierarchical Synthesis

- **Definition:** Maintains the original design hierarchy during synthesis.
- **Structure:** Synthesizes each module or block separately before integrating them.
- **Benefits:**
  - Better management of large and complex designs.
  - Easier reuse of previously synthesized modules (IP reuse).
  - Faster synthesis time for incremental changes.
  - Reduces memory usage during synthesis.
- **Limitations:**
  - May miss cross-module optimization opportunities.
  - Slightly less aggressive timing and area optimization compared to flat synthesis.

### Flat Synthesis

- **Definition:** Flattens the design hierarchy, treating the entire design as a single large block during synthesis.
- **Structure:** All modules are combined into one flattened netlist before synthesis.
- **Benefits:**
  - Allows global optimization across the entire design.
  - Can result in better area and timing performance.
- **Limitations:**
  - Requires more memory and longer run-times, especially for large designs.
  - Harder to debug and analyze after synthesis.
  - Not suitable for extremely large designs due to scalability limits.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img5.png"
  />
</p>



```shell
yosys

read_liberty -lib ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

```shell
read_verilog ./verilog_files/multiple_modules.v
```

```shell
synth -top multiple_modules
```

```shell
abc -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

```shell
show multiple_modules
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img6.png"
  />
</p>

```shell
write_verilog multiple_modules_hier.v
```

Open the `.v` file, It's possible to see the clearly **hierarchical** design:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img7.png"
  />
</p>

### Flat Design

Now to see the flat design:

```shell
write_verilog multiple_modules_flat.v
```

And then open the `.v` file, there's no submodules, because the design It is **flat**:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img8.png"
  />
</p>

## Various Flop Coding style Optimizations


- **Glitches** are unwanted, short-duration voltage spikes in digital signals.
- They occur due to differences in signal propagation delays in combinational logic.
- Glitches can cause incorrect behavior, especially when sampled by downstream circuits.

### Role of Flip-Flops (Flops)

- **Flip-flops** are edge-triggered storage elements used to synchronize signals with a clock.
- They **capture data only on specific clock edges** (rising or falling), making the output immune to short-term input glitches.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img9.png"
  />
</p>

### How Flip-Flops Prevent Glitches

- Flip-flops **"filter out" transient glitches** by sampling inputs only at clock edges.
- As long as the input signal is stable during the setup and hold time windows, the flip-flop ignores glitches occurring at other times.
- This ensures **clean, stable outputs** for the next stages in the design.

### Key Benefits

- **Signal Stability:** Reduces chances of glitch propagation in sequential circuits.
- **Reliable Timing:** Synchronizes data flow with the system clock, making the circuit predictable and easier to analyze.
- **Noise Immunity:** Provides a level of immunity to small, fast glitches generated by combinational logic transitions.

### Anaylizing the Flop code

First, open the `./verilog_files/dff_asyncres_syncres.v ` file:

```verilog
module dff_asyncres_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```

A **D flip-flop with asynchronous reset** captures the input data (`d`) on the active clock edge (usually rising edge) and outputs it at `q`. 

However, when the **reset signal (`rst`) is asserted**, the output `q` is immediately forced to `0`, **regardless of the clock**. This happens asynchronously, meaning it doesn't wait for the clock edge.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img10.png"
  />
</p>

### Simulation

Start Yosys:
```shell
yosys
```
Read Liberty library:
```shell
read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Read Verilog code:
```shell
read_verilog /path/to/dff_asyncres.v
```
Synthesize:
```shell
synth -top dff_asyncres
```
Map flip-flops:
```shell
dfflibmap -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Technology mapping:
```shell
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Visualize the gate-level netlist:
```shell
show
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_2/img/day2_img11.png"
  />
</p>

## What I’ve learned today:

* Understood how .lib timing files define gate behavior based on Process, Voltage, and Temperature (PVT).
* Explored cell characteristics like area, delay, and logic function in .lib files.
* Learned the difference between Hierarchical and Flat Synthesis:
    * Hierarchical synthesis maintains modularity.
    * Flat synthesis merges modules for global optimization.
* Saw how synthesis output differs between hierarchical and flat modes.
* Studied glitches and how flip-flops help avoid them by synchronizing signal changes.
* Analyzed D Flip-Flop behavior with asynchronous and synchronous resets.
* Synthesized and visualized a DFF using Yosys and Sky130 library.
