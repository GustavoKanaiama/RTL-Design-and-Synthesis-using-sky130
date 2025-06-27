# Day 3 - Combinational and sequential optmizations
Topics:

- [Introduction to optimizations](#introduction-to-logic-optimizations)
- [Combinational logic optimizations](#introduction-to-logic-optimizations)
- [Sequential logic optimizations](#sequential-logic-optimizations)
- [Sequential optimizations for unused outputs](#unused-outputs-optimization)

## Introduction to Logic Optimizations

* Squeezing the logic to get the most optimized design, so **Area** and **Power** savings.
* Constant propagation, so Direct Optimization.
* Boolean Logic Optimization, using K-Map and Quine McKluskey techniques.

### Constant Propagation Example

When the value of an input or intermediate signal is known to be constant, the synthesis tool replaces related logic expressions with their simplified constant result. This reduces unnecessary logic gates, minimizes area, and can improve timing by eliminating redundant computations in the circuit.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img1a.png"width=300
  />
</p>

#### Lab
This part going to use the files starting with `opt_` inside the folder `./verilog_files/`. The expect behavior of `opt_check.v`, `opt_check2.v`, and `opt_check3.v` files:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img4a.png"width=300
  />
</p>

Open Yosys:
```shell
yosys
```

Read Liberty library:
```shell
read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Read Verilog code:
```shell
read_verilog /verilog_files/opt_check.v
```
Synthesize:
```shell
synth -top opt_check
```
Optimization:
```shell
opt_clean -purge
```
Technology mapping:
```shell
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img5.png"
  />
</p>

Do the same for `opt_check2.v`:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img6.png"
  />
</p>

Do the same for `opt_check3.v`, in this case, because of the constant propagation, we are able to see an optimization in a multiplexer, synthesizing in an AND gate:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img7.png"
  />
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img8.png"
  />
</p>


### Boolean Logic Optimization

By reducing the number of logic gates and minimizing logic depth, this optimization helps decrease circuit area, power consumption, and propagation delay. Techniques like Karnaugh maps and Quine-McCluskey are commonly used to identify and eliminate redundant logic, merge equivalent terms, and achieve a more efficient hardware implementation.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img2a.png"width=300
  />
</p>

## Sequential Logic Optimizations
* Basic - Sequential Constant propagation.
* Advanced [Not covered as part of Lab] - State Optimization, Retiming, and Sequential Logic Cloning(Floor Plan Aware Synthesis).

### Sequential Constant

Sequential constant optimization is a synthesis technique that identifies and removes sequential elements (like flip-flops) whose outputs remain constant throughout operation. If analysis shows that a register always holds the same value (e.g., always 0 or 1) due to design constraints or logic conditions, the synthesis tool can eliminate the flip-flop and replace its output with the constant. This reduces circuit area, saves power, and simplifies timing by removing unnecessary sequential logic.


#### Lab
This part going to use the files starting with `dff_const` inside the folder `./verilog_files/`.


Open Yosys:
```shell
yosys
```

Read Liberty library:
```shell
read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Read Verilog code:
```shell
read_verilog /verilog_files/dff_const1.v
```
Synthesize:
```shell
synth -top dff_const1
```
Read the specific dfflibmap for flop optimization(Sequential Circuits):
```shell
dfflibmap -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Technology mapping:
```shell
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img10.png"
  />
</p>

Do the same for `dff_const2.v` file:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img11.png"
  />
</p>


## Unused Outputs Optimization
Unused output optimization is a synthesis process where logic driving unused outputs is identified and removed from the design. If certain outputs are not connected to any downstream logic or are not required for the final functionality, the synthesis tool eliminates the associated logic that generates these signals. This helps reduce circuit area, power consumption, and can improve overall performance without affecting the intended operation of the design.

For this example, Let's analyze the `counter_opt.v` file:

```verilog
module counter_opt (input clk, input reset, output q);
reg [2:0] count;
assgin q = count[0];

always @(posedge clk, posedge reset)
begin
    if(reset)
        count <= 3'b0000;
    else
        count <= count + 1;

end
endmodule
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img12.png"
  />
</p>




Open Yosys:
```shell
yosys
```

Read Verilog code:
```shell
read_verilog /verilog_files/counter_opt.v
```
Synthesize:
```shell
synth -top counter_opt
```
Read the specific dfflibmap:
```shell
dfflibmap -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Technology mapping:
```shell
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_3/img/day3_img13.png"
  />
</p>

## What I’ve learned today:

* Learned combinational logic optimization techniques like constant propagation and Boolean simplification (K-Map, Quine-McCluskey).
* Understood how synthesis tools can reduce gate count and improve performance by identifying redundant logic.
* Ran synthesis labs that demonstrated logic reduction using test designs.
* Learned sequential optimizations such as:
  * Constant value flops can be removed.
  * Sequential logic simplification (e.g., unused outputs).
* Identified and removed unused outputs to save power and area.
* Understood how synthesis tools decide what logic to retain or discard based on usage.
