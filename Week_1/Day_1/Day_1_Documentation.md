# Welcome, 

The repository is included in the Verilog RTL Design and Synthesis Workshop with open-source tools, such as, Icarus Verilog (Iverilog), GTKWave, and Yosys. It will take you through the entire process of digital design - starting with writing Verilog RTL and simulating, visualizing waveform, and ultimately converting it into a netlist at the gate level.

---

# 1. Introduction to Verilog RTL Design and Synthesis

This repository introduces the fundamentals of **Verilog RTL design**, **simulation using Icarus Verilog (Iverilog)**, **waveform analysis using GTKWave**, and **logic synthesis using Yosys**. You'll learn the flow from writing Verilog RTL to synthesizing it into a gate-level netlist using open-source tools.

---

# 2. Introduction to Open Source Simulator – Icarus Verilog (Iverilog)

## 2.1 What is a Simulator?

A **simulator** is a tool used to simulate digital designs to check the designs.

## 2.2 Simulator Used: Iverilog

We will use **Icarus Verilog (Iverilog)** in this course. It is an open-source Verilog simulation and synthesis tool.

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/Simulator.png?raw=true)

---

# 3. Understanding Design and Testbench

## 3.1 Design

* A **design** is a Verilog module that describes the required functionality.
* It typically contains:

  - **Primary Inputs**: Used to apply test vectors (stimulus).
  - **Primary Outputs**: Used to observe the behavior.

## 3.2 Testbench

* A **testbench** is used to apply stimulus to the design and verify its behavior.
* It does **not** have any primary inputs or outputs.
* It instantiates the design and provides input values over time.

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/Testbench.png?raw=true)
---

# 4. How Simulators Work

* Simulators monitor changes in input signals.
* If inputs change, the simulator **evaluates** and updates the outputs.
* If no change in inputs, it **skips evaluation** to save time.

---

# 5. Iverilog-Based Simulation Flow

Even though we use **Iverilog**, the same flow is applicable for other simulators.

### Simulation Flow:

1. Compile **Design** and **Testbench** using Iverilog.
2. A **VCD(Vlaue Change Dump Format) File** is created.
3. View the waveform using **GTKWave**.

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/Iverilog_simulation_flow.png?raw=true)

---

# 6. Lab 1 – Using Iverilog and GTKWave

## 6.1 Setting Up the Environment

```bash
sudo -i
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop
ls
cd my_lib
ls
cd verilog_model
cd verilog_files/
```

This Verilog files contain the **design and testbench** code.

---

## 6.2 Running Iverilog Simulation and GTKwave(Part 1)

Use the following command to compile the design and testbench:

```bash
iverilog good_mux.v tb_good_mux.v
```

* `good_mux.v`: Design file.
* `tb_good_mux.v`: Testbench file.

This generates an executable called `a.out`.

Now run the simulation:

```bash
./a.out
```

This produces the **`tb_good_mux.vcd`** file containing waveform data.

---

# 7. GTKWave – Viewing Simulation Waveforms

To open the VCD file in GTKWave:

```bash
gtkwave tb_good_mux.vcd
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/gtkwave_output.png?raw=true)

You can now graphically view all input/output transitions.

---

# 8. Structure of Design and Testbench Files

Use the following command to open both files in split view:

```bash
gvim tb_good_mux.v -o good_mux.v
```

This helps you understand how testbenches apply stimuli to the design.

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/Design_and_tb_files.png?raw=true)

---

#  9. Introduction to Yosys and Logic Synthesis

## 9.1 What is a Synthesizer?

A **synthesizer** converts **RTL code** (usually written in Verilog) into a **gate-level netlist** using a technology library (`.lib` file).

* We use **Yosys**, an open-source synthesis tool.
* It maps the RTL design to standard cells (like AND, OR, NAND, etc.) defined in `.lib`.

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/Synthesizer.png?raw=true)

---

## 9.2 Yosys Synthesis Flow

1. **Read Verilog file**:

   ```bash
   read_verilog good_mux.v
   ```

2. **Read Liberty file (.lib)**:

   ```bash
   read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```

3. **Synthesize**:

   ```bash
   synth -top good_mux
   ```

4. **Technology Mapping**:

   ```bash
   abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```

5. **Write Netlist**:

   ```bash
   write_verilog good_mux_netlist.v
   ```

6. **Visualize** the synthesized netlist:

   ```bash
   show
   ```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/mux2x1.png?raw=true)

---

# 🔍 10. Verifying the Synthesis Output

* Use the same **testbench** with the synthesized **netlist**.
* Simulate using Iverilog again to generate a VCD file.
* Open the VCD in GTKWave to verify that **outputs match RTL simulation**.

✅ If the outputs match, the synthesis is functionally correct.

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Week_1/Day_1/Verify_the_synthesis.png?raw=true)

---

# 11. Logic Synthesis and Gate-Level Modeling

* RTL design is usually described using **behavioral modeling**.
* However, digital hardware is built using **gates**.
* The synthesizer converts RTL into a **gate-level representation**, called a **netlist**.

This process ensures your design can be **fabricated** using real-world transistors and logic cells.

---

# 12. Faster Cell vs Slower Cell

| Feature           | Fast Cell                  | Slow Cell                    |
| ----------------- | -------------------------- | ---------------------------- |
| Delay             | Shorter (Faster Switching) | Longer (Slower Switching)    |
| Power Consumption | Higher                     | Lower                        |
| Area              | Larger (Wide Transistors)  | Smaller (Narrow Transistors) |
| Use Case          | Timing-critical paths      | Power-efficient paths        |

* **Fast Cells**:

  * Use **wide transistors** to reduce propagation delay.
  * Trade-off: Higher **area** and **power**.
  * Can cause **Hold Time Violations**.

* **Slow Cells**:

  * Use **narrow transistors**.
  * Good for reducing **leakage** and saving area.

---

#  13. Selection of Cells in Synthesis

Choosing the right cells is critical:

* Using **too many fast cells**:

  * Increases area and power.
  * May cause hold time issues.

* Using **too many slow cells**:

  * Saves power but hurts performance.

✅ Solution: Use **constraints** to guide the synthesizer to balance speed, area, and power.

---

# 14. Yosys – Synthesis Example (good\_mux.v)

### Yosys Commands Recap:

```bash
yosys
read_liberty -lib /lib/to/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_mux.v
synth -top good_mux
abc -liberty /lib/to/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

This will generate a **netlist** that maps your design to real standard cells.

---

# 15. Summary

* Simulate your design using **Iverilog**.
* View waveforms using **GTKWave**.
* Synthesize design using **Yosys**.
* Understand the trade-offs between **fast** and **slow** cells.
* Learn to guide synthesis using **constraints**.
* Verify synthesis using testbenches and waveform comparison.

---
