# VSD-RTL-Design-SKY130
#  RTL Design using Verilog with    SKY130


## Introduction to Verilog RTL Design and Synthesis

**SIMULATION**
Simulator -->The output is evaluated for change in Input.
Design-->Set of Verilog codes that adheres to the specifications.
Test Bench-->Applies test vectors to the design.
 
**SYNTHESIS**

Synthesizer-->Converts RTL to Netlist.

.lib files
 - Collection of logical modules including basic gates such as And, Or and  			      	  		Not
 -  Contains different flavours of the same gate
 - 2-Input And gate
	-Slow
	-Medium
	-Fast
 - Different flavors of the same cell are needed as they can be used in a variety of applications.  For example, depending on the speed of the cell it can be classified into Faster and Slower Cells.
 - The synthesizer needs to be guided to select the appropriate flavor of cell for optimum implementation of the logic circuit. This guidance is called “Constraints”

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

## SIMULATION OF 2X1 MUX
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

**SYNTHESIS OF 2X1 MUX**

# Timing libs, Hierarchy versus flat synthesis and efficient flop styles









##DAY-3 COMBINATIONAL AND SEQUENTIAL OPTIMISATION

**Combinational Optimisation**
Design is optimised for saving area and power. Combinational optimisation can be done by Constant propogationa and Boolean logic optimisation

Command to Optimise
```
opt_clean –purge
```


## Examples

**Module opt_check**
```
module opt_check (input a,input b,output y);
	assign y = a?b:0;
endmodule
```
Module opt_check acts similar to 2X1 Mux 
If a=1,then y=b
If a=0,then y=0

The above module is optimised into an AND gate
![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/show_and.jpg)

**Module opt_check3**
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
**Simulation**
![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/gtk_diffconst1.jpg)
	In the above GTKwave though reset went to 0 q is waiting for the posedge of the clk.

**Synthesis**

Map the DFFlib
```
dfflibmap –liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```


The Reset in Standard Cell  is active low but we have coded it to be active high so an inverter is inferred by the synthesizer for the input of the reset
![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/show_dffconst1.jpg)

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
![enter image description here](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY3/show_unused.jpg)
In the above module count is a 3bit reg but the output q is assigned only to bit 0. Bits 2,1 remain unused.

Only 1 Flipflop is inferred though it is a 3bit counter.

## Gate Level Synthesis ,Synthesis Simulation mismatch and Blocking Non blocking statements

Gate level Simulation (GLS) is needed to verify the logical correctness of the circuit and ensuring to meet the timing design criteria is met.

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

**Blocking and Non Blocking**

**Blocking Statements:**

 - Sequentially executed one statement after the other

**Non Blocking Statements:**

 - Assigns the RHS value to LHS
 - Parallel Execution

## Example

![gtk_blocking.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gtk_blocking.jpg)
![show_blocking.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/show_blocking.jpg)
![gls_blocking.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gls_blocking.jpg)


**Simulation synthesis mismatch-Missing sensitivity list**

![iverilog.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/iverilog.jpg)

Command to perform GLS
```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
```
![ternary_mux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/ternary_mux.jpg)

![ternary_tb.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/ternary_tb.jpg)

![ternary_tb2.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/ternary_tb2.jpg)

![gtk_ternary_mux](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gtk_ternarymux.jpg)

![gls_ternary_mux](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gls_ternarymux.jpg)

![show_ternarymux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/show_ternarymux.jpg)


**Simulation synthesis mismatch-Blocking Statements**
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
![gtk_badmux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gtk_badmux.jpg)
![gls_badmux.jpg](https://github.com/Krishnakumar-Pugazhenthi/VSD-RTL-Design-SKY130/blob/main/DAY%204/gls_badmux.jpg)


# Day 5 - If, Case, For loop, For Generate

**If Statements**

 - They are used to create priority logic.
 - The If statments are used inside the always block
 - Register type variables are used in If statements
 
General Syntax of If
```
				    if <expression1>
				    begin
			        ---------
                    ---------
                    end
                    else if <expression2>
                    begin
                    ---------
                    ---------
                    end
                    else if <expression3>
                    begin
                    ---------
                    ---------
                    end
                    else
                    begin
                    ---------
                    ---------
                end
```
Dangers of incomplete If

if (condt1) z=x; else if (condt2)  
z=y;

 - The above incomplete if statement is a bad coding style and it will result in a inferred latch.
 - Combinational circuits cannot have a inferred latch
 - However, in sequential circuits for some cases like that of counters using an incomplete If will not result in a inferred latch.For these cases an incomplete If can be used.
 
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
