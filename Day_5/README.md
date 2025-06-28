# Day 5 - Optimization in synthesis
Topics:
- [Constructs: "if" and "Case"](#constructs-if-and-case)
- [Labs on "Incomplete If Case"](#lab-simulation)
- [Labs on "Incomplete overlapping Case"](#lab-simulation)
- [Statements: "for loop" and "for generate"](#statements-for-loop-and-for-generate)
- [Labs on Statements: "for loop" and "for generate"](#statements-for-loop-and-for-generate)

## Constructs "if" and "case"

In Verilog, an incomplete `if` and `case` statements inside a combinational `always` block (e.g., `always @(*)`) can lead to the unintended inference of **latches**. This happens when not all possible conditions are covered, and the synthesis tool assumes the output must retain its previous value for the unspecified cases—requiring storage, hence inferring a latch. Inferred latches can introduce unwanted memory behavior, make timing analysis harder, and often result in unreliable or incorrect circuit operation. To avoid this, always ensure that all conditional branches assign values to outputs, in case of the `case`statement, using the `default`.

Another common caveat when using a `case` statement in Verilog is **partial assignment**, where not all possible case values are covered or the default case is missing. This can cause the synthesis tool to infer **latches**, as the output may need to hold its previous value for unspecified cases. To avoid this, always ensure that all case branches assign values or include a `default` case to cover all possible inputs.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img3.png"
  />
</p>

The `if-else` chain inside a combinational block is typically translated into **multiple cascaded multiplexers**. Each condition is evaluated in sequence, and the result is selected through a series of nested muxes. This can lead to longer combinational paths and increased logic depth if the chain is too long.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img1.png"
  />
</p>

The `case` statement is usually synthesized as **a single large multiplexer**, where all conditions are checked in parallel and one result is selected based on the matching case. This often results in more balanced logic and can be more efficient for selection among many mutually exclusive options.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img2.png"
  />
</p>

### Lab Simulation

To be able to see the synthesis behavior of the caveats mentioned, we will simulate/synthesize some files:
Go to your `verilog_files` directory:

```shell
cd ./VSLI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/
```
Compile the `incomp_if.v` file and it's testbench:
```shell
iverilog incomp_if.v tb_incomp_if.v
```
Dump the VCD file:
```shell
./a.out
```
Open with gtkwave:
```shell
gtkwave tb_incomp_if.vcd
```
<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img4a.png"width=800
  />
</p>

Now for the synthesis, let's use Yosys:

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
read_verilog -noattr ./verilog_files/incomp_if.v
```
Synthesize:
```shell
synth -top incomp_if
```
Technology mapping:
```shell
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```
show the synthesis:
```shell
show
```

It's possible to see the _Inferred Latch_:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img5.png"
  />
</p>

Now, making the exact same process, but with the files `incomp_case.v` and `tb_incomp_case.v`:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img6a.png"width=800
  />
</p>

Then, with the synthesis it's possible to see the _Inferred Latch_ after the mux, due to the incomplete `case` statement:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img_rp1.png"
  />
</p>

To see the caveats with partial assignment in the `case` statement, we will proceed the exact same step-by-step but using the file `partial_case_assign.v`:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img_rp2.png"
  />
</p>

## Statements: "for loop" and "for generate"

A **"for loop" inside an `always` block** is used during simulation and synthesis to **evaluate logic or compute values** within a single hardware block. It does not create multiple hardware copies; instead, it generates repeated logic behaviorally. In contrast, a **"generate for loop"** (using `generate` and `genvar`) is evaluated at elaboration time and is used to **instantiate multiple copies of hardware structures** (like modules or blocks of logic). This physically replicates hardware in the synthesized design, making it ideal for scalable and parameterized hardware generation.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img9.png"
  />
</p>

To understand the use of a `for` loop inside an `always` block, imagine you want to implement a 32-to-1 multiplexer. Instead of writing out all the lines for each `case` condition manually, you can use a `for` loop to **iterate through inputs and perform the necessary evaluations programmatically**, making the code more compact and scalable:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img10.png"
  />
</p>

Now, to understand the use of a `for` loop inside a `generate` block, consider the example of creating multiple AND gates. Instead of manually coding and wiring each AND gate individually, you can use a **`generate for` loop** to instantiate all the gates efficiently in just a few lines of code:

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img11.png"
  />
</p>

## Labs on Statements: "for loop" and "for generate"

To Simulate the behavior of the `for loop` inside the `always` block, we will use the file `mux_generate.v`:

```shell
iverilog mux_generate.v tb_mux_generate.v
```
Generate the VCD file
```shell
./a.out
```
Open with GTKwave
```shell
gtkwave tb_mux_generate.vcd
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img12a.png"width=750
  />
</p>

Now, for the `for` loop inside a `generate` block, imagine implementing a Ripple Carry Adder (RCA) with a configurable width—for example, 5 bits. Instead of manually instantiating each full adder stage, you can use a **`generate for` loop** to automatically create and connect the required number of stages based on the specified width, making the design easily scalable.

In this case, we are using a RCA but instantiating also a full adder(fa):

```shell
iverilog fa.v rca.v tb_rca.v
```
Dump the VCD file
```shell
./a.out
```
Open with GTKwave
```shell
gtkwave tb_rca.vcd
```
<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_5/img/day5_img14b.png"width=900
  />
</p>

## What I’ve learned today:

* Reviewed all key concepts learned in the previous days: RTL coding, simulation, synthesis, and netlist generation.
* Understood that digital circuit optimization is a key goal in synthesis.
* Recognized the use of open-source tools such as Iverilog, GTKWave, Yosys, and Sky130 PDK.
* Learned how each tool fits into a real ASIC frontend design flow.
* Gained a clearer view of the practical, industry-relevant flow of going from RTL to gate-level with full verification.