#  Day 1 - Introduction to Verilog RTL design and Synthesis
Topics:
- Introduction to open-source simulator iverilog
- Labs using iverilog and gtkwave
- Introduction to Yosys and Logic synthesis
- Labs using Yosys and Sky130 PDKs
## Introduction to open-source simulator iverilog
### Design vs Testbench
**Design** is the actual Verilog code or set of Verilog codes which has the intended funcionality to meet with the required specifications.

**Testbench** is the setup to apply stimulus(test_vectors) to the design to check its funcionality.

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_1/img/day_img1.png"
  />
</p>

### Simulator
* RTL design is checked for adherence to the spec by simulating the design
* Simulator is the tool used for simulating the design. 
* This course will use **iverilator** as compiler tool.
#### How does Simulator Works
Simulator looks for the changes on the input signals. Upon change to the input, the output is evaluated. If no change to the input, no change to the output either.

#### Iverilog Based Simulation Flow
<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_1/img/day1_img2.png"
  />
</p>

## Labs using iverilog and gtkwave
First, we need to clone the git repository to get all the files we will need to run the simulations.
Before that, make sure to install "git":
Linux OS:
```shell
`sudo apt install git`
```

Create a specific folder to your projects, let's name "VSLI":

```shell
mkdir VSLI
```

Now, change directory to the created folder and clone the repository:

```shell
cd ./VSLI
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

Now, we can easily access the verilog/testbench verilog files in the following directory:

`./VSLI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/`

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_1/img/day1_img3.png"
  />
</p>

#### Installing Iverilog and GTKWave
To install iverilog and gtkwave open the Linux terminal and type the command:

```shell
sudo apt install iverilog
sudo apt install gtkwave
```
Iverilog is a typical simulator for verilog. Both the design and testbench are provided as input to iverilog and the simulator produces a `.vcd` file for waveform viewing in GTKWave.

#### Compile and Simulate
Let's compile and simulate a simple multiplexer.

First, find the folder where is the `verilog_files` then compile the `good_mux.v` file and it's testbench `tb_good_mux.v`:

```shell
cd ./VSLI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/

iverilog good_mux.v tb_good_mux.v
```

Run the simulation:

```shell
./a.out
```

View the waveform:

```shell
gtkwave tb_good_mux.vcd
```

<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_1/img/day1_img4.png"
  />
</p>

## Introduction to Yosys and Gate Libraries

**Yosys** is a powerful open-source synthesis tool for digital hardware. It takes your Verilog code and converts it into a gate-level netlist—a hardware blueprint.

#### Yosys Features

- **Synthesis:** Converts HDL to a logic circuit
- **Optimization:** Improves speed or area
- **Technology Mapping:** Matches logic to actual hardware cells
- **Verification:** Checks correctness
- **Extensibility:** Supports custom flows

###  Why Do Libraries Have Different Gate "Flavors"?

A `.lib` file contains many versions of each gate (like AND, OR, NOT) with different properties:

- **Performance:** Faster gates for critical paths, slower for power savings
- **Power:** Some gates use less energy
- **Area:** Smaller gates for compact chips
- **Drive Strength:** Stronger gates to drive more load
- **Signal Integrity:** Specialized gates for noise/performance
- **Mapping:** Synthesis tools pick the best flavor for your needs

---

### 6. Synthesis Lab with Yosys

Let’s synthesize the `good_mux` design using Yosys. 

Start Yosys:
```shell
yosys
```

Read the liberty library
```shell
read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read the Verilog code
```shell
read_verilog ./VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/good_mux.v
```

Synthesize the design
```shell
synth -top good_mux
```

Technology mapping
```shell
abc -liberty ./VSLI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Visualize the gate-level netlist
```shell
show
```
<p align="center">
  <img src="https://raw.githubusercontent.com/GustavoKanaiama/RTL-Design-and-Synthesis-using-sky130/refs/heads/main/Day_1/img/day1_img5.png"
  />
</p>