#  RTL Design using Verilog with    SKY130

![enter image description here](https://github.com/VictorySpecificationII/Sky130-VLSI-Workshop/raw/master/Images/VSD/vsdbanner.png?raw=true)
 **WORKSHOP(Sept 1- Sept 5)**
## Table of Contents

 - Introduction to Verilog RTL Design and Synthesis.
 - Timing libs, Hierarchy versus flat synthesis and efficient flop styles.
 - Combinational and Sequential Optimisations.
 - Gate level Synthesis,Synthesis simulation mismatch.
 - If, Case, For loop, For Generate

 
## *Day1-Introduction to Verilog RTL Design and Synthesis*

**SIMULATION**

Simulator -->The output is evaluated for change in Input.

Design-->Set of Verilog codes that adheres to the specifications.

Test Bench-->Applies test vectors to the design.

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/block_diagram.JPG)

 
**SYNTHESIS**

Synthesizer-->Converts RTL to Netlist.

![RTL to Netlist](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/Verify.JPG)

.lib files
 - Collection of logical modules including basic gates such as And, Or and  			      	  		Not
 -  Contains different flavours of the same gate
 - 2-Input And gate
	-Slow
	-Medium
	-Fast
 - Different flavors of the same cell are needed as they can be used in a variety of applications.  For example, depending on the speed of the cell it can be classified into Faster and Slower Cells.
 - The synthesizer needs to be guided to select the appropriate flavor of cell for optimum implementation of the logic circuit. This guidance is called “Constraints”

![Netlist to Simulation](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/Netlist.JPG)


**Faster Cells vs Slower cells**

Faster cell-->Fast charge, discharge of capacitance-->Wider transistor-->Delay(**↓**)Power(↑) Area (↑)

Slower Cell--> Slow charge, discharge of capacitance-->Narrow transistor-->Delay(↑)Power(**↓**) Area (**↓**)


## Setting up the Environment
**Install iverilog**
```
sudo apt install iverilog
```

**Install GTK wave**

```
 Sudo apt install gtk wave
```

**Cloning Sky 130 Standard cells**
```
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

**SIMULATION OF 2X1 MUX**
**Verilog of module Mux**
```
module good_mux (input i0,input i1,input sel,output reg y);
always @ (*)
begin
if(sel)
	y <= i1;
	else
	y <= i0;
end
endmodule
```
**Testbench of the MUX** 

```
`timescale 1ns / 1ps
module tb_good_mux;
reg i0,i1,sel; 	
wire y;	
good_mux uut (.sel(sel),.i0(i0),.i1(i1),.y(y));	// Instantiate the Unit Under Test (UUT)
initial begin
$dumpfile("tb_good_mux.vcd");
$dumvars(0,tb_good_mux);
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
![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/dumpfile.jpg)

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/gtk_goodmux.jpg)

**Synthesis** 

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/read_liberty.jpg)

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/read_verilog.jpg)

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/synth%20goodmux.jpg)

After abc liberty command in Yosys 0.7 the individual gates are relalized  as shown below

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/yosys.7.JPG)

Wherea in Yosys 0.9 below after the abc liberty command it will be realized as a whole mux instead of gates.

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/yosys.9.JPG)
The above difference is due to the different versions of the Yosys synthesizer

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/write_netlits_mux.jpg)

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/netlist_mux.jpg)

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY1/show_mux.jpg)


# *Day2-Timing libs, Hierarchy versus flat synthesis and efficient flop styles*
To read the library file
```
gvim <Repository Path>/my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/standard_cell_library.jpg)

"sky130_fd_sc_hd__tt_025C_1v80"
where 
130-->Process node
fd-->Foundary
sc-->Ftandard cell library
hd-->High Density
tt-->Typical type
025C-->Temperature
1v80-->Voltage

**sky130_fd_sc_hd library**

 - It is designed for high density.
 - this library enables lower dynamic power consumption, higher routed gated density, leakage power and comparable timing.
 -  Flip-flops and  Latches have scan equivalents for scan chain creation.

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/gate_flavour_type1.jpg)

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/gate_flavour_type2.jpg)

Variations may happen due to the Process, Temperature or voltage
Process. There is a variation in process during fabrication. Voltage and temperature variations also result in behaviour changes.

**Hierarchial Synthesis**

Submodule level synthesis
 - In a design with multiple instances we can use this to synthesize once and replicate it many times and stich together to obtain the netlist file.
 - On the other hand big designs can be broken down synthesised and merged later into a single netlist.

Verilog Code of multiple_modules
```
module sub_module2 (input a, input b, output y);
assign y = a | b;
endmodule
module sub_module1 (input a, input b, output y);
assign y = a&b;
endmodule
module multiple_modules (input a, input b, input c , output y);
	wire net1;
	sub_module1 u1(.a(a),.b(b),.y(net1));  //net1 = a&b
	sub_module2 u2(.a(net1),.b(c),.y(y));  //y = net1|c ,ie y = a&b + c;
endmodule
```
Heirarchial synthesis of multiple_module

```
$ yosys
yosys> read_liberty -lib  ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog multiple_module.v
yosys> synth -top multiple_module
yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show multiple_modules
yosys> write_verilog -noattr multiple_modules_hier.v
yosys> show
```

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/show_hierarchy.jpg)

```
yosys> !nano multiple_modules_hier.v
```

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/nano_heirarchy.jpg)

**Flatten**

After the hierarchical synthesis we have to flatten
```
yosys> flatten
yosys> show
```
![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/show_flatten.jpg)

```
yosys> !nano multiple_modules_flat.v
```
![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/nano_flatten.jpg)


## Flipflop coding styles

Why FlipFlops are needed?

 - In a combinational circuit for the given input after a propogation delay we get an output. Due to the propogation delay there will glitch in the output.
 - If there are many combinational circuits in a design then the output will be more glitchy. In order to avoid this in between the combinational circuits we can use Flipflops
 - Flip flops are like storage elements that store values. 
 - Though the inputs of Flipflops are glitching the output of the flop is stable and helps in correct functionality of the circuit.

Verilog Code for asynchronous reset
```
module dff_asyncres ( input clk ,  input async_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```
Command to map to the dfflib
```
yosys> dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Simulation for Asynchronous reset

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/gtk_asyncrest.jpg)

Synthesis for Asynchronous reset

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/show_asyncreset.jpg)

Simulation for Asynchronous and synchronous reset

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/gtk_asyn_syncreset.jpg)

Synthesis for Asynchronous and synchronous reset

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day%202/show_async_syncreset.jpg)


# *DAY-3 COMBINATIONAL AND SEQUENTIAL OPTIMISATION*

**Combinational Optimisation**
Design is optimised for saving area and power. Combinational optimisation can be done by Constant propogationa and Boolean logic optimisation

Command to Optimise
```
opt_clean –purge
```
## Examples

Module opt_check
```
module opt_check (input a,input b,output y);
	assign y = a?b:0;
endmodule
```
Module opt_check acts similar to 2X1 Mux 
If a=1,then y=b
If a=0,then y=0

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/show_and.jpg)

The above module is optimised into an AND gate

Module opt_check3
```
module opt_check3 (input a, input b, input c, output y);
assign y = a?(c?b:0):0;
endmodule
```
Module opt_check3 works as follows
If a=0, then y=0 else check for c
If C is true then y=B
else 0

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/show_optcheck3.jpg)

The above module is optimised into a 3-Input AND gate.


**Sequential Optimisation**
Sequential optimisation can be done by the following methods 
 - Sequential constant propogation
 - State optimisation-->Optimisation of unused state
 - Retiming-->Split the logic towards slack
 - Cloning of Logic-->Physical aware synthesis

## Examples

Module dff_const1	
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
Simulation

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/gtk_diffconst1.jpg)

	In the above GTKwave though reset went to 0 q is waiting for the posedge of the clk.

Synthesis

Map the DFFlib
```
dfflibmap –liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Synthesis

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/show_dffconst1.jpg)

The Reset in Standard Cell  is active low but we have coded it to be active high so an inverter is inferred by the synthesizer for the input of the reset

**OPTIMISATION FOR UNUSED OUTPUT**
In some cases all the outputs present might not be assigned and remain unsed. It is unwise to translate them into hardware. 

## Example

Module counter_opt
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
Synthesis

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/show_unused.jpg)

In the above module count is a 3bit reg but the output q is assigned only to bit 0. Bits 2,1 remain unused.Only 1 Flipflop is inferred though it is a 3bit counter.

## *Day 4-Gate Level Synthesis ,Synthesis Simulation mismatch and Blocking Non blocking statements*

Gate level Simulation (GLS) is needed to verify the logical correctness of the circuit and ensuring to meet the timing design criteria is met.

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/GVLM.jpg)

**GLVM**

Let us take the example of a netlist of the function **Y=(a&b)|c**

**and** u_and(.a(a),.b(b),.y(i0));

**or** u_or(.a(i0),.b(c),.y(y));

In the above sample the AND, OR cell mentioned need not be an AND, OR gate. It can be a simple naming convention. The information about the AND, OR mentioned in the above netlist is contained in the GLVM.

GLVM can be timing aware or functional. Though netlist is a true representation of the RTL the functionality of the netlist is to evaluate and there can be a synthesis and simulation mismatch.

**Synthesis and Simulation Mismatch**

This can happen because of some reasons like Missing sensitivity list, Blocking vs Non blocking assignments

**Missing sensitivity list**

 - The simulation works by change in inputs results in change of outputs
 - There might be mismatch in netlists of simulator and synthesizer as synthesizer does not look at sensitivity lists
 - 

Command to perform GLS
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
```
Verilog Code

![ternary_mux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/ternary_mux.jpg)

Simulation 

![gtk_ternary_mux](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gtk_ternarymux.jpg)

GLS

![gls_ternary_mux](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gls_ternarymux.jpg)

Synthesis

![show_ternarymux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/show_ternarymux.jpg)

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
RTL Simulation

![gtk_badmux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gtk_badmux.jpg)

GLS 

![gls_badmux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gls_badmux.jpg)

From the above RTL simulation and GLS there is a clear simulation simulation synthesis mismatch

**Blocking and Non Blocking**

**Blocking Statements:**

 - Sequentially executed one statement after the other

**Non Blocking Statements:**

 - Assigns the RHS value to LHS
 - Parallel Execution


**Simulation synthesis mismatch-Blocking Statements**

Simulation

![gtk_blocking.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gtk_blocking.jpg)

Synthesis

![show_blocking.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/show_blocking.jpg)

GLS 
![gls_blocking.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gls_blocking.jpg)


# *Day 5 - If, Case, For loop, For Generate*

**If Statements**

 - They are used to create priority logic.
 - The If statments are used inside the always block
 - Register type variables are used in If statements
 
General Syntax of If
```
				    if <expression1>
				    begin
			        statements
                    end
                    else if <expression2>
                    begin
                    statements
                    end
                    else if <expression3>
                    begin
                    statements
                    end
                    else
                    begin
                    statements
                end
```
Dangers of incomplete If

if (condt1) z=x; else if (condt2)  
z=y;

 - The above incomplete if statement is a bad coding style and it will result in a inferred latch.
 - Combinational circuits cannot have a inferred latch
 - However, in sequential circuits for some cases like that of counters using an incomplete If will not result in a inferred latch.For these cases an incomplete If can be used.
 
Simulation

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/gtk_incompif.jpg)

 Synthesis

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/latch_incompif.jpg)

A latch is inferred in the above diagram

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/show_icompf.jpg)


**Case Statements**
- The case statments are used inside the always block
 - Register type variables are used in case statements
 
 Just like the If statements an incomplete case statement can lead to an inferred latch. In order to overcome this we can code a default case as shown below.
 ```
case(signal)
    <case1> = begin
	     	 Statements
		         end
    <case2> = begin
	          Statements
			     end
    default = begin
              Statements
              end
```

Simulation of Complete case

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/gtk_comcase.jpg)

Synthesis of Complete case

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/show_compcase.jpg)

Simulation of Incomplete case

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/gtk_incomcase.jpg)

Synthesis of Incomplete case

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/show_incompcase.jpg)

**Overlapping Case Statements**
Let us take the below example
```
case(signal)
  2'b00 = begin
          Statements
  	      end
  2'b01 = begin
      	  Statements
 	      end
  2'b10 = begin
      	  Statements
 	      end
  2'b1 = begin
       	 Statements
 	     end
 endcase
```
In the above code the statements 3 and 4 matches and we get an unpredictable output as the case statements check for every case.

Simulation of overlapping case 

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/gtk_badcase.jpg)

Synthesis of overlapping case 

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/show_badcase.jpg)

**FOR Loops**
```
module mux_generate (input i0 , input i1, input i2 , input i3 , input [1:0] sel  , output reg y);
wire [3:0] i_int;
assign i_int = {i3,i2,i1,i0};
integer k;
always @ (*)
begin
for(k = 0; k < 4; k=k+1) begin
	if(k == sel)
	y = i_int[k];
end
end
endmodule
```
Simulation of 4x1 Mux

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/gtk_muxgenerate.jpg)

Synthesis of 4x1 Mux

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/show_muxgenerate.jpg)

Simulation of Demux

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/gtk_demuxgenerate.jpg)

Synthesis of Demux

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/show_demuxgenerate.jpg)

**FOR Generate**
```
module rca (input [7:0] num1 , input [7:0] num2 , output [8:0] sum);
wire [7:0] int_sum;
wire [7:0]int_co;
genvar i;
generate
	for (i = 1 ; i < 8; i=i+1) begin
		fa u_fa_1 (.a(num1[i]),.b(num2[i]),.c(int_co[i-1]),.co(int_co[i]),.sum(int_sum[i]));
	end
endgenerate
fa u_fa_0 (.a(num1[0]),.b(num2[0]),.c(1'b0),.co(int_co[0]),.sum(int_sum[0]));
assign sum[7:0] = int_sum;
assign sum[8] = int_co[7];
endmodule
```
We have to invoke the full adder also otherwise it will throw an error

```
iverilog fa.v rca.v tb_rca.v
./a.out
./gtkwave 
```
Simulation of RCA

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/gtk_rca.jpg)

```
read_verilog rca.v fa.v
synth -top rca
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog rca_net.v
```
Synthesis of RCA

![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/Day5/show_rca.jpg)

## Acknowledgement

 - [Kunal Ghosh- Co-founder (VSD Corp Pvt Limited)](https://www.vlsisystemdesign.com/about-me/)
 - [Shon Taware](https://github.com/ShonTaware)
 - [VSD IAT Platform](https://vsdiat.com/home) 

