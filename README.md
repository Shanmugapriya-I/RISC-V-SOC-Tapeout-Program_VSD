# RISC V SOC Tapeout Program VSD

<details>
  <summary>Week 0 </summary>

# Week 0 - Tools Installation

### **System Requirements**
- 6 GB RAM
- 50 GB HDD
- Ubuntu 20.04 or higher
- 4 vCPU

### <ins>**Yosys**</ins>
```bash
$ sudo apt-get update
$ git clone https://github.com/YosysHQ/yosys.git
$ cd yosys
$ sudo apt install make (If make is not installed please install it)
$ sudo apt-get install build-essential clang bison flex \
libreadline-dev gawk tcl-dev libffi-dev git \
graphviz xdot pkg-config python3 libboost-system-dev \
libboost-python-dev libboost-filesystem-dev zlib1g-dev
$ make config-gcc
$ make
$ sudo make install
```
![Alt Text](Images/Yosys.png)

### <ins>**Icarus Verilog**</ins>
```bash
$ sudo apt-get update
$ sudo apt-get install iverilog
```
![Alt Text](Images/IcarusVerilog.png)

### <ins>**Gtkwave**</ins>
```bash
$ sudo apt-get update
$ sudo apt install gtkwave
```
![Alt Text](Images/gtkwave.png)

</details>

<details>
  <summary>Week 1 </summary>

# Week 1 

   <details>
       <summary>Day 1 </summary>

## **Day 1 - Introduction to Verilog RTL Design and Synthesis**

### <ins>**Introduction to open-source simulator Iverilog**</ins>

Folder structure of the git clone:
- `lib` - will contain sky130 standard cell library
- `my_lib/verilog_models` - will contain standard cell verilog model
- `verilog_files` -contains the lab experiments source files

![Alt Text](Images/IverilogFlow.png)


Example of a design good_mux.v 

```
module good_mux (input i0 , input i1 , input sel , output reg y);
always @ (*)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
```
Example of a testbench tb_good_mux.v 

```
`timescale 1ns / 1ps
module tb_good_mux;
	// Inputs
	reg i0,i1,sel;
	// Outputs
	wire y;

        // Instantiate the Unit Under Test (UUT)
	good_mux uut (
		.sel(sel),
		.i0(i0),
		.i1(i1),
		.y(y)
	);

	initial begin
	$dumpfile("tb_good_mux.vcd");
	$dumpvars(0,tb_good_mux);
	// Initialize Inputs
	sel = 0;
	i0 = 0;
	i1 = 0;
	#300 $finish;
	end

always #75 sel = ~sel;
always #10 i0 = ~i0;
always #55 i1 = ~i1;
endmodule
```
Command to run the design and testbench
```
iverilog good_mux.v tb_good_mux.v
```
The output of the iverilog is a .vcd file and a.out file is created. By executing a.out iverilog dump the vcd file.

### <ins>**Introduction to GTKWave**</ins>

Gtkwave will be used to generate the waveforms and display in visual format.

Command to view the vcd file in gtkwave 
```
gtkwave tb_good_mux.vcd
```
The waveform in gtwave is shown below


![Alt Text](Images/IverilogFlow.png)

### <ins>**Introduction to Yosys**</ins>

It is the synthesizer used to convert RTL to netlist.
Netlist should be the same as the Design but represented in the form of standard cells.
The same testbench can be used to verify RTL and Synthesized Netlist.


![Alt Text](Images/YosysFlow.png)

### <ins>**Introduction to Logic Synthesis**</ins>


![Alt Text](Images/SynthesisIllustration.png)

### <ins>**Lab using Yosys and Sky130 PDKs**</ins>


</details>

<details>
  <summary>Day 2 </summary>

	
## **Day 2 - Timing libs, Hierarchical vs Flat Synthesis and Efficient Flop Coding Styles**

### <ins>** Introduction to timing .libs**</ins>

Libraries are characterized based on PVT (process, voltage, temperature) \
Process -> Variations due to fabrication \
Voltage -> Variations due to voltage \
Temperature -> Variations due to temperature 

As seen in the screenshot below \
tt stands for typical in the .lib name \
025C stands for temperature of 25 C in the .lib name \
1v80 stands for voltage of 1.8V in the .lib name

![Alt Text](Images/LibraryFile.png)

-cell defines the beginning of the cell. Other information of cells mentioned are:
- Leakage power based on the combination of inputs
- Area
- Power ports
- Input capacitance
- Power associated with the pin
- Transition
- Delay

### <ins>**Hierarchical vs Flat Synthesis**</ins>

### <ins>**Hierarchical Synthesis**</ins>

Report after synthesizing multiple_modules.v. As shown below the sub_modules statistics are printed. For example, sub-module1 has 1 AND gate and sub-module2 has 1 OR gate. This is an example of Hierarchical Synthesis.

![Alt Text](Images/SubmoduleCode.png)

Hierarchy is preserved. sub_module1 and sub_module2 are instantiated separately in the synthesized Verilog netlist. Rather than seeing AND or OR gate, we see sub_modules when we run the command 'show' as shown in the screenshot.

![Alt Text](Images/SubmodeleBlock.png)

If we look into the sub_module2 in synthesized netlist 'multiple_modules_hier.v', we see that rather than OR gate, the inputs a & b, pass through the inverter and then NAND gate. It is because in CMOS, stacking PMOS, which happens in 'OR' gate is bad as PMOS has lower mobility and always have to be wider to get some meaningful output. The next step is to check .lib file for the answer.

### <ins>**Flat Synthesis**</ins>

The design can be flattened by using the command `flatten`.

Screenshot shows the command, synthesized netlist and the logical diagram.

![Alt Text](Images/FlattenImage.png)

### <ins>**Sub-module Level Synthesis**</ins>

RTL (Register Transfer Level) designs are often modular, with various functional blocks or sub-modules. Sub-module level synthesis allows each of these sub-modules to be synthesized independently.

Why is the sub-module level synthesis necessary?

- Optimization and Area Reduction: By synthesizing sub-modules separately, the synthesis tool can optimize each one individually. It performs logic optimization, technology mapping, and area minimization for each sub-module. This leads to more efficient use of resources and reduced overall chip area.
- Resuability: Each submodule can be designed, verified, and optimized independently. They can be reused in a large design multiple times saving time and enhancing efficiency. 
- Parallel Processing: Different sub-modules can be synthesized concurrently, improving efficiency. For large designs, parallel synthesis significantly reduces turnaround time.

The commands to run sub-module synthesis
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top sub_module1
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The screenshot shows that when sub_module1 is synthesized, only AND gate is generated. 

![Alt Text](Images/Submodel1.png)

### <ins>**Various Flop Coding Styles and Optimization**</ins>

### <ins>**Why do we need flops and how do they prevent glitches in the circuit?**</ins>

Glitches can occur in digital circuits due to various reasons such as signal delays, noise, or timing issues. Flops prevent glitches during the operation in the following ways:

- Synchronization: Flops are edge-triggered devices, meaning they respond only to transitions of the input signal (e.g., rising edge, falling edge). This synchronization ensures that the output changes only at specific points, reducing the likelihood of glitches caused by transient signal variations.
- Timing Control: Flops are typically controlled by a clock signal, ensuring that all circuit operations occur synchronously. This eliminates timing issues that could lead to glitches due to data arriving at different times.

![Alt Text](Images/WhyFlops.png)

### <ins>**Different types of flops**</ins>

To initialize flops, we need to `set` and `reset` which can be synchronous or asynchronous.

![Alt Text](Images/TypesOfFlop.png)


The screenshot below shows DFF with asynchronous reset HDL simulation in Iverilog and  waveform display in GTKwave. Irrespective of the clock and d, as soon as async_reset=1, q=0.

![Alt Text](Images/DffAsyncresWave.png)

### <ins>**Synthesizing flops**</ins>

The command to synthesize ***DFF with asynchronous reset*** as an example
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
![Alt Text](Images/DffAsyncresBlock.png)


On synthesizing ***DFF with synchronous reset*** we get NOR gate with inverted `d` as shown in the screenshot below. However,on evaluating the boolean expression, we reached the same logic realization. 


![Alt Text](Images/DffSyncresBlock.png)


### <ins>**Synthesizing mult2 (multiply by 2)**</ins>

 
To implement `y[3:0] = 2*a[2:0]`, we append a `1'b0 `to the `a[2:0]` i.e, `y[3:0] = {a[2:0],0}`. This is also equal to left shift the input bits by 1.
This can be realized by just wiring.
So we expect no hardware which is also seen in the screenshot below, analysis after synthesis and show. The command 'abc' is not required for mapping when there are no cells.

![Alt Text](Images/Mul2Block.png)

### <ins>**Synthesizing mult9 (multiply by 9 or 8+1)**</ins>

`y=9*a` can be considered `8*a+1*a`
To implement `y[5:0] = 9*a[2:0]`, we append `000` to `a[2:0]` and then add `a` i.e, `y[5:0] = {a[2:0],000} + a[2:0]`.
This can be realized just by wiring.
So we expect no hardware which is also seen in the screenshot below, analysis after synthesis and show. The command 'abc' is not required for mapping when there are no cells.

![Alt Text](Images/Mul8Block.png)

</details>

<details>
	<summary>Day 3</summary>
	
## **Day 3 - Combinational and Sequential Optimizations**

### <ins>**Introduction to Optimizations**</ins>

### <ins>**Combinational Logic Optimization**</ins>

It means squeezing the logic to get the most optimized design in terms of area and power. the most commonly used techniques are:
1) Constant propagation using direct optimization
2) Boolean logic optimization using K-map and Quine McKlusky

An example of constant propagation optimization is highlighted below.

![Alt Text](Images/ConstantPropagation.png)

An example of boolean optimization is highlighted below.

![Alt Text](Images/BooleanLogicOptimization.png)

### <ins>**Sequential Logic Optimization**</ins>

The technqiues used are:

1) Basic
   - Sequential constant propagation
2) Advanced (not covered as part of lab)
   - Static optimization
   - Retiming
   - Sequential logic cloning (floorplan aware synthesis)

An example of sequential constant propagation is highlighted below of DFF with asynchronous reset where D input is grounded. To note, the same technique cannot be applied to DFF with the asynchronous set because while `Q=1` when `Set=1`, but `Q=0` at `Set=0` at the next CLK pulse. Q is dependent not only on Set but also on the clock edge.

![Alt Text](Images/SequentialConstant.png)

Retiming is a technique to improve the performance of the circuit.

![Alt Text](Images/StateOpt.png)

### <ins>**Combinational Logic Optimizations**</ins>

Commands for optimization

```
opt_clean -purge
```
### <ins>**Optimization of opt_check.v**</ins>

Syntax for opt_check.v
```
module opt_check (input a , input b , output y);
        assign y = a?b:0;
endmodule
```
For opt_check.v the assignment `y = a?b:0` reduces to `y = ab`. The screenshot shown below explains this

![Alt Text](Images/OptCheck.png)

The logic implementation after synthesis for opt_check.v is shown below, showing only AND gate.

![Alt Text](Images/OptCheckBlock.png)


### <ins>**Optimization of opt_check2.v**</ins>

Syntax for opt_check2.v
```
module opt_check2 (input a , input b , output y);
        assign y = a?1:b;
endmodule
```
For opt_check2.v the assignment `y = a?1:b` reduces to `y = a + b`. 

The logic implementation after synthesis for opt_check2.v is shown below, showing only OR gate.

![Alt Text](Images/OptCheck2Block.png)

### <ins>**Optimization of opt_check3.v**</ins>

Syntax for opt_check3.v
```
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```
For opt_check.v the assignment `y = a?(c?b:0):0` reduces to `y = abc`. The screenshot shown below explains this.

![Alt Text](Images/OptLogic.png)

The logic implementation after synthesis for opt_check3.v is shown below, showing 3 input AND gate.

![Alt Text](Images/OptCheck3Block.png)

### <ins>**Optimization of multiple_module_opt.v**</ins>

Syntax of multiple_module_opt.v
```
module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule

module sub_module2(input a , input b , output y);
 assign y = a^b;
endmodule

module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));

assign y = c | (b & n1); 
endmodule
```
The logic implementation after synthesis for multiple_module_opt.v is shown below.

![Alt Text](Images/MultipleModelOpt.png)

### <ins>**Sequential Logic Optimizations**</ins>

Both the dff_const1.v and dff_const2 are explained below.

![Alt Text](Images/SequentialLogic.png)

### <ins>**Optimizing dff_const1.v**</ins>

Syntax for dff_const1.v
```
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end

endmodule
```
For dff_const1.v, `q=0` as long as `reset=1`. However, when `reset=0` `q` doesn't immediately becomes `1` rather at the next rising edge of the `clk` as shown below. ***So the optimization cannot be applied***.

![Alt Text](Images/TbDffConst1.png)

The commands to run the synthesis
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The logic implementation after synthesis for dff_const1.v is shown below.

![Alt Text](Images/DffConst1Block.png)

***complete dff_const2,4,5***

### <ins>**Optimizing dff_const3.v**</ins>

Syntax for dff_const3.v
```
module dff_const3(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```
For dff_const3.v, there are two flops.  `q1=0` as long as `reset=1`. However, when `reset=0` `q1` doesn't immediately becomes `1` rather at the next rising edge of the `clk` with some propagation delay as shown below. `q=1` as long as `reset=1`, acting as `set` rather than `reset`. However, when `reset=0` `q` samples `q1` as `0` as there are some propagation delay for `q1`as shown below. At the next `clk` edge `q` samples `q1` as `1`.
***So the optimization cannot be applied***.

![Alt Text](Images/DffConst3Logic.png)

The command to run HDL simulation
```
iverilog dff_const3.v tb_dff_const3.v
./a.out
gtkwave tb_dff_const3.vcd
```
The HDL simulation is shown below.

![Alt Text](Images/DffConst3Wave.png)

The commands to run the synthesis
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const3.v
synth -top dff_const3
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The logic implementation after synthesis for dff_const3.v is shown below.

![Alt Text](Images/DffConst3Block.png)

### <ins>**Sequential Optimzations for Unused Outputs**</ins>

### <ins>**Optimization of Case1: 3-bit Up Counter with q[0] used (counter_opt.v)**</ins>

Example of a counter where bits at the position of [2] and [1] are unused.

```
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count[0];

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule
```
The screenshot explains the logic of the counter. Only q[0] is used. ***So the optimization can be applied***.

![Alt Text](Images/Opt.png)


</details>

<details>

 <summary>Day 4 </summary>
 
## **Day 4 - GLS, Blocking vs Non-blocking and Synthesis-Simulation Mismatch**

### <ins>**GLS, Synthesis-Simulation Mismatch, and Blocking/Non-blocking Statements**</ins>

### <ins>**Why is Gate Level Simulation (GLS) necessary?**</ins>

- Verify the correctness of the design after synthesis
- Ensure the timing of the design is met which is done with delay annotation (timing aware)

![Alt Text](Images/GLSUsingIverilog.png)
  
### <ins>**Synthesis Simulation Mismatches**</ins>

It happens because of the following reasons
- Missing sensitivity list
- Blocking vs non-blocking assignments
- Non-standard verilog coding

### <ins>**(1) Missing sensitivity list**</ins>

As shown in the screenshot below, `always` block is evaluated only when `sel` is changing. So output `y` is not evaluated when `sel` is not changing although `i0` and `i1` are changing. Rather it acts like a latch. The code on the right side represents the correct design coding for `mux`. In this case `always` is evaluated for any signal changes. 

![Alt Text](Images/MissingSensitivityList.png)

### <ins>**(2) Blocking vs Non-blocking Assignments**</ins>

 ### <ins>**Blocking Statements**</ins>
 
 - Represented by `=`
 - Executes the statements in the order it is written inside always block
 - So the first statement is evaluated before the second statement

### <ins>**Non-Blocking Statements**</ins>

- Represented by `<=`
- Executes all the RHS when always block is entered and assigns to LHS
- Parallel execution

   The left side of the screenshot below gives us the correct execution. While the right side can lead to serious issues as `d` is assigned to `q` directly. ***So choosing non-blocking statements is best practice*** (highlighted in the screenshot below).

![Alt Text](Images/CaveatsWithBlocking.png)

 ### <ins>**Blocking Statements Leading to Synthesis Simulation Mismatch**</ins>

In the code shown below, `y` gets the old `q0` value. This will mimic delay or flop. But when you synthesize, there will be no flop. If the order is changed (right side code), latest value of `q0` is assigned to `y`. 

When synthesized, both will lead to the same circuit. However, simulation will result in different behavior. For the left side of the code, `y` gets the old `q0` value and for the right side of the code, `y` gets the latest `q0` value leading to a synthesis simulation mismatch. 

This issue is resolved by using ***non-blocking statements***.

![Alt Text](Images/Caveats.png)

### <ins>**Labs on GLS and Synthesis-Simulation Mismatch**</ins>

### <ins>**Ternary operator MUX (ternary_operator_mux.v)**</ins>

The Verilog code of ternary_operator_mux.v
```
module ternary_operator_mux (input i0 , input i1 , input sel , output y);
	assign y = sel?i1:i0;
	endmodule
```
The command to run HDL simulation
```
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```
HDL Simulation waveform of ternary_operator_mux.v is shown in the screenshot below

![Alt Text](Images/TernaryWave.png)

The commands to run the synthesis for ternary_operator_mux.v
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog ternary_operator_mux_net.v
```

![Alt Text](Images/TernaryBlock.png)

The commands to do GLS for ternary_operator_mux.v
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```
The GLS output is shown below.

![Alt Text](Images/TernaryGLSWave.png)

### <ins>**Bad MUX (bad_mux.v)**</ins>

The `always` block is executed only at `sel` signal. It works like a flop rather than mux.
The Verilog code of bad_mux.v
```
module bad_mux (input i0 , input i1 , input sel , output reg y);
always @ (sel)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
```

The command to run HDL simulation
```
iverilog bad_mux.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```
HDL Simulation waveform of bad_mux.v is shown in the screenshot below

![Alt Text](Images/BadMuxWave.png)

The commands to run the synthesis for bad_mux.v.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_mux.v
synth -top bad_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog bad_mux_net.v
```

The synthesis report shows it is still inferring the mux but not the flop.

![Alt Text](Images/BadMuxBlock.png)

The commands to do GLS for bad_mux.v
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```
The GLS output is shown below. This shows correct functionality which is different from HDL simulation, leading to ***synthesis simulation mismatch***.

![Alt Text](Images/BadMuxGLSWave.png)

### <ins>**Labs on Synthesis-Simulation Mismatch for Blocking Statements**</ins>

### <ins>**Blocking Caveat (blocking_caveat.v)**</ins>

The logic to simulate is shown below.

![Alt Text](Images/BlockingCaveats.png)

The Verilog code of blocking_caveat.v
```
module blocking_caveat (input a , input b , input  c, output reg d); 
reg x;
always @ (*)
begin
	d = x & c;
	x = a | b;
end
endmodule
```

The command to run HDL simulation
```
iverilog blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```
HDL Simulation waveform of blocking_caveat.v is shown in the screenshot below. `d` takes the old value of `x` causing incorrect functionality.

![Alt Text](Images/BlockingCaveatsWave.png)


The commands to run the synthesis for bad_mux.v.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog blocking_caveat_net.v
```

The synthesis report and logic synthesis is shown below.

![Alt Text](Images/BlockingCaveatsBlock.png)


The commands to do GLS for bad_mux.v
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat_net.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```
The GLS output is shown below. In this case, `d` takes the current value of `x` causing incorrect functionality.The waveform shows correct functionality which is different from HDL simulation, leading to ***synthesis simulation mismatch***.

![Alt Text](Images/BlockingCaveatsGLSWave.png)

</details>




  
  


