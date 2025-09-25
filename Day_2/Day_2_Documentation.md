---

# **Day 2: Timing Libraries, Hierarchical vs Flat Synthesis, and Efficient Flip-Flop Coding**

Welcome back! 🚀
This is **Day 2** of my journey into digital design. On **Day 1**, we explored the basics of RTL design and synthesis. Today, we dive into **timing libraries, synthesis styles, and efficient flip-flop coding techniques**.

---

## **1. Introduction to `.lib` Files**

1. **Standard cells** included in a `.lib` (Liberty) file include **NAND, NOR, AND, OR**, and other gates.
2. The `.lib` file contains **fast and slow cells** along with technology-specific data.
3. To open a `.lib` file:

   ```bash
   gvim ../lib/sky130_fd_sc_hd__tt_025C1v80.lib
   
   ```

   ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/lib.png?raw=true)
   
5. **Tip:** If the text is hard to read:

   * Press `Esc`
   * Type `Shift + :` and enter:

     ```vim
     syn off
     ```

     This turns off syntax highlighting for better readability.

---

## **2. Understanding PVT and Library Naming**

1. `.lib` files describe **Process, Voltage, and Temperature (PVT)** corners.

2. Example: `sky130_fd_sc_hd__tt_025C_1v80.lib`

   * **sky130** → 130nm SkyWater technology
   * **tt** → Typical process
   * **025C** → Temperature: 25°C
   * **1v80** → Supply voltage: 1.8V

3. Libraries include:

   * Technology files
   * Delay models
   * Operating conditions
   * Capacitance, resistance, current, and power data
   * Many standard cells with leakage, area, and functional expressions

---

## **3. Browsing Cells in the `.lib` File**

1. **In gvim**, to locate and explore cells:

   ```vim
   :se h/s      " Highlight start of cell definitions
   :se nu       " Show line numbers
   /cell        " Search for the keyword 'cell'
   :g//         " Display all cell definitions
   ```

2. **Example cell definition:**

   ```liberty
   cell("sky130_fd_sc_hd__a2111o_4") { ... }
   ```

   * `a2111o_4` → Contains 2-input AND and OR logic gates.

     ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/cell.png?raw=true)

3. **To see the corresponding Verilog implementation:**

   ```vim
   :sp ../my_lib/verilog_model/sky130_fd_sc_hd.v
   /sky130_fd_sc_hd__a2111o
   ```

    ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/a2111o_4.png?raw=true)
   
   * This opens the Verilog logic code of the cell to understand its behavior.

---

## **4. Cell Area and Power Examples**

| Cell Name | Area (µm²) | Notes                    |
| --------- | ---------- | ------------------------ |
| and2_o    | 6.25       | Small cell, less power   |
| and2_2    | 7.07       | Medium-sized cell        |
| and2_4    | 8.04       | Wider cell, higher power |

* **Wider cells** → greater drive strength → more power
* **Smaller cells** → less area and power

---

So far, we have used **single modules**, but in real designs, we often deal with **multiple modules**. For example, in the folder `verilog_files`, we have:

* `multiple_modules.v` → top module
* Inside it: `sub_module1` and `sub_module2`

  ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/sub_module_block.png?raw=true)

we can open the file by

  ```bash
    gvim multiple_modules.v
  ```
  ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/multiple_module_verilog.png?raw=true)

We can synthesize `multiple_modules` using **Yosys** as follows:

---

## **1. Start Yosys**

```bash
yosys
```

---

## **2. Read the Technology Library**

```yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

* This loads the **SkyW130nm standard cell library** for synthesis.

---

## **3. Read the Multi-Module Verilog File**

```yosys
read_verilog multiple_modules.v
```

* This reads the **top module** along with all **sub-modules** defined inside.

---

## **4. Synthesize the Top Module**

```yosys
synth -top multiple_modules
```

* `-top` specifies which module to synthesize.
* Sub-modules are synthesized **hierarchically** as part of the top module.

---

## **5. Add Technology Mapping**

```yosys
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

* Maps the RTL to **actual standard cells** in the library.

---

## **6. Visualize the Netlist**

```yosys
show multiple_modules
```

* Opens a **graphical view** of the synthesized module and its sub-modules.

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/multiple_module_synth.png?raw=true)

---

## **7. Write the Synthesized Verilog (netlist)**

```yosys
write_verilog multiple_modules_hier.v
```

---

## **8. Open the Synthesized File in gvim**

```bash
!gvim multiple_modules_hier.v
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/multiple_modules.png?raw=true)

---

## **9. Optional: Write Verilog Without Attributes**

```yosys
write_verilog -noattr multiple_modules_hier.v
```

---

## **10. Verify in gvim Again**

```bash
!gvim multiple_modules_hier.v
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/multiple_modules_hier.png?raw=true)

---

### 5. Flattening a Hierarchical Design

In a hierarchical RTL design, modules are organized like this:

```
Top
 ├── Submodule1
 ├── Submodule2
 └── Submodule3
```

* Using **flattening**, all submodules are combined into a **single module**.
* This can be done in Yosys with:

```yosys
flatten
write_verilog -noattr multiple_modules_flat.v
```

* open the flattened file in GVim:

```bash
!gvim multiple_modules_flat.v
```

* To compare the hierarchical and flattened designs, use vertical split in GVim:

```vim
:vsp multiple_modules_hier.v
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/VSP_command.png?raw=true)

**Key point:**

* After flattening, **no submodules remain**; all logic is combined into a single module.

---

### 6. Module-level Synthesis

Sometimes, we want to **synthesize at the submodule level** instead of the full design. This is useful when:

* The full design is very large (divide and conquer).
* We want to reuse submodules multiple times without re-synthesizing them.

**Steps in Yosys:**

```yosys
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog sub_module1.v
synth -top sub_module1
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/submodule1.png?raw=true)


**Benefits of module-level synthesis:**

1. Easier handling of large designs.
2. Faster compilation since only one submodule is synthesized.
3. Promotes **modular reuse**—synthesized submodules can be instantiated multiple times.

   ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/sub_modules%20in%20top.png?raw=true)
   

**Divide and Conquer:**

* Instead of giving a massive design to the tool, break it into smaller modules.
* Synthesize each module independently.
* This ensures better tool performance and easier debugging.

---

Got it! I’ll continue your GitHub-ready notes, starting numbering from **7** and explaining **why we need flops** clearly and concisely:

---

### 7. Why We Need Flip-Flops (D Flip-Flops)

In a **purely combinational circuit**:

* Multiple gates are connected in series or parallel.
* **Each gate has a propagation delay**.
* When inputs change, intermediate signals can produce **glitches** (temporary incorrect values).
* The more gates or combinational logic you have, the **higher the chance of glitches** at the output.

**Problem:**

* Glitches can propagate and cause **unstable outputs** if we directly use combinational outputs in the next stage.

**Solution: Use D Flip-Flops**

* D flip-flops are placed **between outputs of combinational circuits**.
* They **sample and store the output at the clock edge**, stabilizing it.
* This ensures that the next stage always sees a **stable, glitch-free value**.

  ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/remove_gitches.png?raw=true)
  

---

### 8. Initialization of Flip-Flops

* At power-up, flip-flops can hold **random or “garbage” values**.
* To avoid this, flip-flops are **initialized** using **reset** or **set** pins:

| Pin Type | Function         | Notes                              |
| -------- | ---------------- | ---------------------------------- |
| Reset    | Sets output to 0 | Can be synchronous or asynchronous |
| Set      | Sets output to 1 | Can be synchronous or asynchronous |

* **Synchronous reset/set:** Takes effect only at the clock edge.
* **Asynchronous reset/set:** Takes effect immediately, independent of the clock.
* Initialization ensures that combinational circuits **see a valid input** from flip-flops, avoiding garbage propagation.

---

### 9. Flip-Flop with Asynchronous and Synchronous Reset/Set

Flip-flops often need **initialization** so that they don’t start with garbage values. This is done using **reset** (clear output to `0`) or **set** (force output to `1`).
These can be either **asynchronous** or **synchronous**.

---

#### 9.1 Asynchronous Reset

* The reset takes effect **immediately**, independent of the clock.
* If `reset = 1`, the output goes to `0` right away.
* If `reset = 0`, the output follows `D` only at the next clock edge.

```verilog
// Asynchronous Reset
always @(posedge clk or posedge reset) begin
    if (reset)
        q <= 1'b0;   // Output forced to 0 immediately
    else
        q <= d;      // Capture D at clock edge
end
```
## GTKWave

``` bash
# Compile design and testbench
iverilog dff_asyncres.v tb_asyncres.v

# Run simulation
./a.out

# Open waveform in GTKWave
gtkwave tb_dff_asyncres.vcd
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/asyncres_gtk.png?raw=true)

## Synthesis of Asynchronous Reset

```bash
# Load standard cell library (Sky130)
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read design
read_verilog dff_asyncres.v

# Run synthesis
synth -top dff_asyncres

# Map DFFs to technology-specific library cells
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Optimize and map combinational logic
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 

# Show schematic
show
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/Asyncres_syn.png?raw=true)

---

#### 9.2 Asynchronous Set

* If `set = 1`, the output goes to `1` immediately.
* If `set = 0`, the output follows `D` at the next clock edge.

```verilog
// Asynchronous Set
always @(posedge clk or posedge set) begin
    if (set)
        q <= 1'b1;   // Output forced to 1 immediately
    else
        q <= d;      // Capture D at clock edge
end
```
## GTKWave

``` bash
# Compile design and testbench
iverilog dff_async_set.v tb_dff_async_set.v

# Run simulation
./a.out

# Open waveform in GTKWave
gtkwave tb_dff_async_set.vcd
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/Asynchronous_set.png?raw=true)

## Synthesis

```bash
# Load standard cell library (Sky130)
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read design
read_verilog dff_async_set.v

# Run synthesis
synth -top dff_async_set

# Map DFFs to technology-specific cells
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Optimize and map logic
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 

# Show schematic
show

```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/async_set_Synth.png?raw=true)

---

#### 9.3 Synchronous Reset

* Reset takes effect **only at the clock edge**.
* If `reset = 1`, output becomes `0` at the **next rising edge**.

```verilog
// Synchronous Reset
always @(posedge clk) begin
    if (reset)
        q <= 1'b0;   // Reset only at clock edge
    else
        q <= d;
end
```
## GTKWave

``` bash
# Compile design and testbench
iverilog dff_syncres.v tb_dff_syncres.v

# Run simulation
./a.out

# Open waveform in GTKWave
gtkwave tb_dff_syncres.vcd
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/Syncres_gtk.png?raw=true)

## Synthesis

```bash
# Load standard cell library (Sky130)
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read design
read_verilog dff_syncres.v

# Run synthesis
synth -top dff_syncres

# Map DFFs to technology-specific cells
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Optimize and map logic
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib 

# Show schematic
show

```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_2/syncres_synth.png?raw=true)

---

#### 9.4 Asynchronous + Synchronous Reset Together

* When **both asynchronous and synchronous resets** are present, **asynchronous reset has higher priority**.
* Typical code structure:

```verilog
// Async + Sync Reset with Priority
always @(posedge clk or posedge async_reset) begin
    if (async_reset)
        q <= 1'b0;          // Highest priority
    else if (sync_reset)
        q <= 1'b0;          // Next priority
    else
        q <= d;             // Normal operation
end
```

---




