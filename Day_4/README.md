# Day 4 - GLS, blocking vs non-blocking and Synthesis-Simulation mismatch
Topics:
- GLS, Synthesis-Simulation mismatch and Blocking/Non-blocking statements
- Labs on GLS and Synthesis-Simulation Mismatch
- Labs on synth-sim mismatch for blocking statement

## Gate Level Simulation and Synthesis - Simulation Mismatches

What is GLS?
* Running the test bench with Netlist as DUT.
* Netlist logically same as RTL Code. Same Test Bench will align with the Design.

Why GLS?
* Verify the logical correctness of the design after synthesis.
* Ensuring the timing of the design is met. For this GLS needs to be run with delay annotation. (outside the scope of this discussion).

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img1.png"
  />
</p>

## Synthesis Simulation Mismatch
* Missing sensitivity List
* Blocking vs Non-Blocking assignments
* Non Standard Verilog Coding

### Missing Sensitivity List
A **sensitivity list** is a part of a process block in hardware description languages like Verilog. It defines the signals that trigger the execution of the process whenever they change. For example of two different sensitivity list:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img2.png"
  />
</p>

### Blocking vs Non-Blocking assignments
* Inside always block:
    **Blocking** " = "
    - Executes the statements in order it is written (just like C Language).
    - So the first statement is evaluated before the second statement.

    **Non Blocking** " <= "
    - Executes all the RHS when always block is entered and assigns to LHS.
    - Parallel evaluation.

#### Caveats with Blocking Statements
* Using Block statements inside the _always_, the order matters, so by changing the order (in the example below) the synthesis infer one D Flop instead of two :

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img3.png"
  />
</p>

* Other Caveat with Blocking Statement, is to accidentally infer a flop synthesis. Because the code evaluates an older `q0` value first for the output `y`,and then update the `q0`, this 1 cycle storage for `q0` is inferred as a Flop, and could cause a Synthesis Simulation Mismatch:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img4.png"
  />
</p>

### Lab Simulations
For this Lab we are going to use the files `ternary_operator_mux.v`, `bad_mux.v`, and `good_mux.v`.

First open the terminal and compile the files using **iverilog**:

```shell
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
```
Generate the vcd dump file:
```shell
./a.out
```
Open the vcd file with GTKwave
```shell
gtkwave tb_ternary_operator_mux.vcd
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img5.png"
  />
</p>

To synthethize the circuit with Yosys:
Open Yosys:
```shell
yosys
```
Read .lib:
```shell
read_liberty -lib ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read Verilog code:
```shell
read_verilog -noattr ./verilog_files/ternary_operator_mux.v
```
Synthesize:
```shell
synth -top ternary_operator_mux
```
Technology mapping:
```shell
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Write verilog netlist:
```shell
write_verilog -noattr ternary_operator_mux_net.v
```
Show
```shell
show
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img6.png"
  />
</p>

Now, to generate the GLS we will need some other files to compile using **iverilog**:

```shell
iverilog ./my_lib/verilog_model/primitives.v ./my_lib/verilog_model/sky130_fd_sc_hd.v ./verilog_files/ternary_operator_mux.v ./verilog_files/tb_ternary_operator_mux.v
```

Dump VCD files, and open VCD files using gtkwave:
```shell
./a.out

gtkwave tb_ternary_operator_mux.vcd
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img7.png"
  />
</p>

By doing the same process with the file `bad_mux.v` we get:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_4/img/day4_img8.png"
  />
</p>

As we can see, the simulation does not behavior like a mux, because at the beginning, when sel=0, it should outputs i0. Because the _sel_ signal does not have any action, the mux does not work properly (as we see the bad sensitivity list in the code).