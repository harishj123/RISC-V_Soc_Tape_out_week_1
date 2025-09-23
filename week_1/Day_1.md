
# Introduction to verilog RTL Design and Synthesis

# Introduction to Open Source simulator Iverilog

## Simulator

Simulator is a tool used for simulating the design for checking the design.

## Simulator Used  
## Iverilog

Iverilog is a simulator which is going to use in this course.

## Design
Design is a verilog code which has the actual functionality of the required specifications.

Design has multiple primary Inputs and Primary Outputs.

Primary Inputs are used to generate stimulus or test vectors.

Primary Outputs are used to observe the stimulus.

## Testbench
Testbench is used to apply stimulus or test vectors to check the functionalities of the design.

Testbench does not have any primary inputs and primary outputs.

## Working of Simulator
- Mainly,Simulator looks for the change in the input signal.

- If any input changes,then simulator evaluates the output.

- If no change in the inputs,then does not evaluate the outptut.

## Iverilog based simulation flow
image...

Any simulator can be used not only Iverilog,different simulator can be used which can changes the inputs are dumped in the changes int he outptut according to the changes in the inputs.

Design and Testbench code are compiled in the iverilog and the VCD(Value Change Dump Format) file is created.The VCD file is visualized graphically with the help of GTKWave.

## Lab 1 - Using Iverilog and GTKWave

## Intro to Lab

To start the login shell use,

`
Sudo -i
`

Clone Git file for design and testbench code,

```git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git```

After Clone, Change directory

```cd sky130RTLDesignAndSynthesisWorkshop```

List the files inside the directory

``` ls ```

Change directory to my_lib

``` cd my_lib```

``` ls ```

change to Verilog model,

``` cd verilog model```

change directory to Verilog files,

``` cd verilog_files/```


These verilog files contains all the design and testbench file.

## Intro to Iverilog and GTKWave Part-1

Use both Design and Testbench file from verilog files to run in iverilog

use command,

``` iverilog good_mux.v tb_good_mux.v```

- where good_mux.v is a design file.

- tb_good_mux.v is a testbench file.

Then ./a.out file is created

./a.out is used to dump outputs to the testbench.

tb_good.vcd is a VCD file which has all the dumped outputs.

# GTKWave

 Use GTKWAve to visiuasize the VCD file in Graphical manner.

Use Command,

``` gtkwave tb_mux.vcd```

We can visualize the vcd file graphically.

## Structure of files

Find the structure of Design and Testbench file Using

``` gvim tb_good_mux.v -o good_mux.v```

image....

##  Intro to Yosys and Logic Synthesis

# Synthesizer

Synthesizer is tool for convertinf RTL to netlist.

Yosys is one of the synthesizer.

Here the Design file and .lib file(which contains standard cells like AND,OR,NAND....)

- Both Design and .lib files are given to yosys then it will generate a netlist file.

image....

use command for Design,

``` read_verilog```

use command for .lib,

``` read_liberty```

netlist file,

```write_verilog```

Netlist is a representation of design in the form of cells in .lib file.

# Verify the Synthesis

To verify the synthesis use following steps,

- Netlist file and Testbench file are given to iverilog 

- Iverilog creates a VCD file.

- VCD file can be represented in waves using GTKWave.

- The output need to be same as we observed in the RTL Simulation.

- A Set of Primary inputs and primary outputs will remain same between RTL Design and synthesized netlist.So the same Testbench can be used like in RTL.

## Logic Synthesis

- Usually,RTL Design is represented in Behavioural modelling.

image....

- But Computer Hardware can understand Digital logic circuits only.

# How this RTL code and Dgital Logic circuits are matched by synthesis?

- RTL is converted to Gate Level Modelling.
- Design is converted to gates and the connections are made between the gates.
- This is given out as a file called netlist.

Why Different Flavours of Gate?















