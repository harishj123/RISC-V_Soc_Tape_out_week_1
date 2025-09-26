# Day 3: Introduction to Logic Optimisation

## 🔹 Combinational Logic Optimisation 

Combinational logic optimisation is the process of **squeezing the logic** to get the most optimised design, which saves **area and power**.

### ✨ Common Techniques

* **Constant Propagation**
* **Boolean Logic Optimisation**

---

### 🔸 Constant Propagation

Constant propagation is a direct optimisation technique used to simplify logical expressions.

**Example:**

We have an **AND gate** and a **NOR gate**. Inputs `A` and `B` go to the AND gate, whose output and another input `C` go to the NOR gate.

$Y = !{(AB + C)}$

 ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/const_logic_example.png?raw=true)

* If **A = 0** (grounded), then:
  $Y = \overline{C}$
  → Only one **inverter** is needed for the output.

📉 **Transistor Count in CMOS:**

* For $Y = !{(AB + C)}$: needs **6 transistors**
     ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/constant_propagation.png?raw=true)
  
* For $Y = \overline{C}$: needs **2 transistors**
     ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/inv.png?raw=true)

✅ This demonstrates **constant propagation optimisation**.

---

### 🔸 Boolean Logic Optimisation

Boolean optimisation uses methods like **K-map** or **Quine–McCluskey** to simplify logic expressions.

**Example:**

```verilog
assign y = a ? (b ? c : (c ? a : 0)) : (!c);
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/Boolean_optimisation_example.png?raw=true)

It is not a optimised one.To optimise it we need to do boolean expression reduction technique.

Naively, this requires **three multiplexers**. Let’s optimise step by step:

1. First MUX simplifies:
   $(c \times 0) + ac = ac$

2. Second MUX simplifies:
   $bc + a\bar{b}c$

3. Third MUX output:
   $a(bc + a\bar{b}c) + \bar{a}\bar{c}$
   $= abc + a\bar{b}c + \bar{a}\bar{c}$
   $= ac[b + \bar{b}] + \bar{a}\bar{c}$
   $= ac + \bar{a}\bar{c}$
   $= a \odot c$

✅ Final Optimised Expression:
$y = a \odot c$  (XNOR)

So, instead of **3 multiplexers**, only an **XNOR gate** is required. 🎉

---

## 🔹 Sequential Logic Optimisation

Sequential logic optimisation focuses on improving circuits with storage elements (like flip-flops).

### Techniques:

* **Sequential Constant Propagation**
* **State Optimisation**
* **Retiming**
* **Sequential Logic Cloning (Floorplan-Aware Synthesis)**

---

#### 🔸 Sequential Constant Propagation

Consider a **D Flip-Flop with reset** connected to a NAND gate:

* If **reset = 1**, output $Q = 0$.
* If **reset = 0**, but $D = 0$, then again $Q = 0$.

  ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/seq_const_reset.png?raw=true)
  

Similarly for a **set-enabled Flip-Flop**:

* If **set = 1**, output $Q = 1$.
* If **set = 0**, and $D = 0$, then $Q = 0$.

  ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/seq_const_set.png?raw=true)
  

⚠️ Problem: The circuit cannot determine a **constant sequential value**, since at times $Q = set$ and $Q = \bar{set}$, which introduces delay. For sequential constant propagation, the output should remain constant — but in this case, it does not. Hence, it **cannot be optimised** this way.

---

Some Advanced Techniques as follows:

#### 🔸 State Optimisation

This technique is used to eliminate or repurpose **unused states** in finite state machines (FSMs), leading to simpler and faster circuits.

---

#### 🔸 Logic Cloning (Physical-Aware Synthesis)

 ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/cloning.png?raw=true)
 
Suppose logic drives multiple flip-flops: `FF_A`, `FF_B`, and `FF_C`. If `A` is placed **far from both B and C**, delay increases. To fix this:

 ![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/cloning_second_way.png?raw=true)

* Instead of routing the same logic all the way, we **clone the flip-flop**.
* Logic is given to one flip-flop and its duplicate distributes signals closer to `B` and `C`.

✅ This reduces long interconnect delays and improves performance.

---

#### 🔸 Retiming

Retiming is shifting logic across flip-flops to balance delays:

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/retiming.png?raw=true)


* Two D flip-flops are connected sequentially with two logic blocks.
* Logic 1 has **5 ns delay**, Logic 2 has **2 ns delay**.

📊 Performance:

* Max frequency is set by the slowest path.
* Logic 1 → $f = 1/5ns = 200 MHz$.
* Logic 2 → $f = 1/2ns = 500 MHz$.
* Minimum operating frequency = **200 MHz**.

Now, by **shifting 1 ns of logic** from block 1 to block 2:

* Logic 1 delay = 4 ns → $f = 250 MHz$
* Logic 2 delay = 3 ns → $f = 333 MHz$

✅ Effective system frequency increases beyond 200 MHz. Retiming improves **overall throughput and performance**.

---

# **Digital Logic Optimization Experiment**

This experiment demonstrates **logic optimization in Verilog** using multiplexers (`mux`) and synthesis in Yosys. We show how simple Boolean expressions are reduced to their optimal gate-level implementations.

---

## **1️⃣ List and Open Files**

First, list all the files related to optimization:

```bash
ls *opt*
```

From these, we will focus on `opt_check` files:

```bash
ls *opt_check*
```

Open the first file for inspection:

```bash
gvim opt_check.v
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check.v.png?raw=true)

---

## **2️⃣ `opt_check.v` Example**

### **Verilog Code**

```verilog
assign y = a ? b : 0;
```

### **Boolean Expansion**

Digital Logic Expansion:

```
y = a̅ ⋅ 0 + a ⋅ b
y = 0 + a ⋅ b
y = a ⋅ b

✅ Optimized to a single AND gate

```

✅ The **multiplexer logic** is optimized to a single **AND gate**.

---

## **3️⃣ `opt_check2.v` Example**

Open the second file:

```bash
gvim opt_check2.v
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check2.png?raw=true)

### **Verilog Code**

```verilog
assign y = a ? 1 : b;
```

### **Boolean Expansion**

Digital Logic Expansion:

```
y = a ⋅ 1 + a̅ ⋅ b
y = a + a̅ ⋅ b
y = a + b   (by Consensus Theorem)

```

✅ Here, the **logic is optimized to a single OR gate**.
**Note:** The simplification uses the **Consensus Theorem**:

$$
A + \bar{A}B = A + B
$$

---

## **4️⃣ Synthesis Using Yosys**

We can now synthesize and visualize the optimized design.

```bash
yosys
```

### **Yosys Commands**

```tcl
# Load timing library
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read Verilog file
read_verilog opt_check.v

# Synthesize top module
synth -top opt_check

# Clean unused logic and wires
opt_clean -purge

# Map to standard cells
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Show schematic
show
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check_synth.png?raw=true)

Like the same do synthesis for opt_check2.v

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check2_synth.png?raw=true)

---

Do the same step for opt_check3.v ,opt_check4.v and multiple_modules.v files

opt_check3.v

 ``` bash
    gvim opt_check3.v
 ```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check3.png?raw=true)

synthesis opt_check3.v file

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check3_synth.png?raw=true)

opt_check4.v

``` bash
    gvim opt_check4.v
 ```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check4.png?raw=true)

Synthesis opt_check4.v file

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/opt_check4_synth.png?raw=true)

multiple_modules.v

``` bash
  gvim multiple_modules.v
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/multiple_module.png?raw=true)


Synthesis multiple_modules.v

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/multiple_module_synth.png?raw=true)

---

# Sequential Logic Optimization – DFF Constant Files

This section demonstrates **sequential logic optimization** by synthesizing **D flip-flops with constant inputs**.

---

## 1️⃣ List DFF Constant Files

Use the following command to list all relevant files:

```bash
ls *dff*const*
```

---

## 2️⃣ Open Multiple Files in `gvim`

To edit and view multiple files at once:

```bash
gvim dff_const1.v -o dff_const2.v
```

* The `-o` option opens **both files in split windows**.
* You can now compare and edit the **DFF constant implementations** side by side.

---

## dff_const1.v

## GTKwave

use command,

``` bash
 iverilog dff_const1.v tb_dff_const1.v
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/DFF_const1.png?raw=true)


## 3️⃣ Yosys Synthesis Workflow

**Step 1 – Start Yosys**

```bash
yosys
```

**Step 2 – Load Standard Cell Library**

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Step 3 – Read Verilog Files**

```tcl
read_verilog dff_const1.v
```

**Step 4 – Synthesize Top Module**

```tcl
synth -top dff_const1 
```

**Step 5 – Clean Unused Logic**

```tcl
opt_clean -purge
```

**Step 5 – dff map library**

```tcl
dfflibmap ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Step 6- map to standard cells**

``` bash
 abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

```tcl
opt_clean -purge
```

**Step 7 – Visualize the Design**

```tcl
show
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const1_synthesis.png?raw=true)

---

Perform same for remaining dff files.

----
**dff_const2.v**

open gtkwave

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/Dff_const2.png?raw=true)

synthesis

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const2_synth.png?raw=true)

---
**dff_const3.v**

open gtkwave

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const3.png?raw=true)

synthesis

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const3_synth.png?raw=true)

---
**dff_const4.v**

open gtkwave

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const4.png?raw=true)

synthesis

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const4_synth.png?raw=true)

---
**dff_const5.v**

open gtkwave

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const5.png?raw=true)

synthesis

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/dff_const5_synth.png?raw=true)

---

## 🟢 Unused Output Optimization

open counter_opt.v file by

``` bash
   gvim counter_opt.v
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/counter_opt_code.png?raw=true)


This file looks like a 3 bit up counter.

### 🔎 Observation

* The counter is **3-bit (count[2:0])** → counts like `000, 001, 010, ... 111`.
* But output `q = count[0]` → **only the LSB is used**.
* This means `count[1]` and `count[2]` are **unused**.

👉 Since they do not affect the output, synthesis tools (like **Yosys**) will **optimize away** the unused flip-flops.

* Final optimized design = **just a single D flip-flop toggling** (`0,1,0,1,...`).

---

### ⚡ Run Synthesis

```sh
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter_opt.v
synth -top counter_opt
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/counter_opt_synthesis.png?raw=true)

---

### ✅ Result

* Original RTL → 3-bit counter.
* After optimization → **1 flip-flop** remains, because only `count[0]` impacts output.

This is called **Unused Output Optimization** 🎯.
It removes registers/gates that **do not contribute to primary outputs**.

---

## 🟢 Unused Output Optimization – Example 2

### Step 1: Copy the Original File

```sh
cp counter_opt.v counter_opt2.v
```

* Makes a copy of the original counter file.
* Now you can edit safely without touching the original.

---

### Step 2: Edit the File

```sh
gvim counter_opt2.v
```

Inside the file, change the output assignment to:

 The changes can be done by pressing i (in insert mode).

```verilog
assign q = (count[2:0] == 3'b100);
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/counter_opt2_code.png?raw=true)


* Here `q` will be `1` **only when counter value = `100` (decimal 4)**.
* Otherwise `q = 0`.

So output sequence will look like:

```
000 → q=0
001 → q=0
010 → q=0
011 → q=0
100 → q=1
101 → q=0
110 → q=0
111 → q=0
000 → q=0
...
```

Save and quit in GVim:

* Press `Shift + :` → type `wa!` → Enter

---

### Step 3: Run Synthesis

```sh
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter_opt2.v
synth -top counter_opt2
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_3/counter_opt2.png?raw=true)

---

### 🔎 What Happens After Optimization?

* Now all **3 bits of counter** (`count[2:0]`) are **needed** to check if the value = `100`.
* So **no optimization of unused states is possible**.
* Final circuit will keep **all 3 flip-flops** + comparator logic.

👉 Compare this with `counter_opt.v` (only 1 flip-flop left after optimization).

---

✅ This shows how **output logic determines whether extra flip-flops are removed or kept** in synthesis.

---

# 📌 Day 3 Summary – Logic Optimisation

1. **Combinational Optimisation** → Constant propagation & Boolean reduction minimise gates, saving **area + power**.  
2. **Constant Propagation** → Fixed input values simplify circuits (e.g., complex gate → **inverter**).  
3. **Boolean Optimisation** → K-map / Boolean algebra reduces complex mux logic to **simple gates** (e.g., XNOR).  
4. **Sequential Optimisation** → Includes constant propagation, state optimisation, logic cloning, and retiming for **better performance**.  
5. **Unused Output Optimisation** → Synthesis removes flip-flops/logic not affecting outputs (e.g., 3-bit counter → **1 flip-flop** if only LSB is used).  











