
---

# 🚀 Day 5: Optimization in Synthesis – `if` vs `case` Statements

👋 **Welcome back!**
In RTL design, how you describe logic (`if` or `case`) has a **direct impact on synthesis results, hardware structure, and performance**. A small mistake—like missing an `else` or `default`—can introduce **unwanted latches** and hurt optimization.

Today, we’ll explore:

* ✅ How `if` creates **priority logic**
* ✅ How `case` creates **parallel logic**
* ⚠️ Common pitfalls that cause **latch inference**
* 🔑 The key differences and best practices

---

## 🔹 `if` Statement – Priority Logic

* `if` and `else if` create **priority logic**.
* Highest condition comes first, and synthesis gives priority accordingly.
* If conditions are incomplete, it may lead to **latch inference**.

### ✅ Syntax

```verilog
if (condition1) begin
    // highest priority
end
else if (condition2) begin
    // next priority
end
else begin
    // lowest priority
end
```

### ⚠️ Danger – Inferred Latches

If no `else` is given, synthesis **remembers the previous value** → infers a **latch**.

#### Example – Incomplete `if`

```verilog
always @(*) begin
    if (cond1)
        y = a;
    else if (cond2)
        y = b;
    // ❌ No "else" -> y holds previous value -> latch inferred
end
```

#### Example – Counter with Missing `else`

```verilog
always @(posedge clk or posedge reset) begin
    if (reset)
        count <= 3'b000;
    else if (enable)
        count <= count + 1;
    // ❌ No "else" -> count holds old value -> latch inferred
end
```

---

## 🔹 `case` Statement – Parallel Logic

* `case` creates **parallel logic** (no priority).
* Cleaner for **multi-way selection**.
* **Incomplete cases** also infer latches.

### ✅ Syntax

```verilog
always @(*) begin
    case (sel)
        2'b00: y = a;
        2'b01: y = b;
        2'b10: y = c;
        2'b11: y = d;
        default: y = 0;  // avoids latches
    endcase
end
```

---

## ⚠️ Caveats in `case`

### 1️⃣ Incomplete Case → Latch Inference

```verilog
always @(*) begin
    case (sel)
        2'b00: y = a;
        2'b01: y = b;
        // ❌ Missing 10 and 11 -> latch inferred
    endcase
end
```

✅ Fix: Add **`default`**

```verilog
always @(*) begin
    case (sel)
        2'b00: y = a;
        2'b01: y = b;
        default: y = 0; // prevents latch
    endcase
end
```

---

### 2️⃣ Partial Assignment

When not **all signals** are assigned in a branch, synthesis **remembers old values**.

#### Example – Partial Assignment

```verilog
reg [1:0] sel;
reg x, y;

always @(*) begin
    case (sel)
        2'b00: begin
            x = a;
            y = b;
        end
        2'b01: begin
            x = c;
            // ❌ y not assigned here -> latch inferred
        end
        default: begin
            x = d;
            y = b;
        end
    endcase
end
```

✅ Fix: Assign all outputs in every branch.

---

## 🔑 Key Differences: `if` vs `case`

| Feature         | `if` / `else if` / `else` | `case`                                 |
| --------------- | ------------------------- | -------------------------------------- |
| **Logic Type**  | Priority logic            | Parallel logic                         |
| **Execution**   | Top → bottom              | One selected branch                    |
| **Best for**    | Comparisons, priority mux | Multi-way selection                    |
| **Latch Risk**  | Missing `else`            | Missing `default` / partial assignment |
| **Readability** | Good for few conditions   | Cleaner for multi-bit signals          |

---

# 🧪 Day 5 Lab – Incomplete `if` and Latch Inference

In this lab, we test **incomplete `if` statements** and observe how they infer **latches**. We’ll simulate the design, view waveforms in GTKWave, and then synthesize using Yosys with the Sky130 library.

---

## 🔹 Step 1: Open Incomplete Files

```bash
ls *incomp*
gvim *incomp* -o
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/opening%20all%20incomp%20files.png?raw=true)

* This opens all files containing `incomp` (incomplete logic examples).
* `-o` option in `gvim` opens multiple files side by side.

---

## 🔹 Step 2: Simulate with Icarus Verilog + GTKWave

Compile and run the simulation:

```bash
iverilog incomp_if.v tb_incomp_if.v
./a.out
```

View waveforms:

```bash
gtkwave tb_incomp_if.vcd
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_if_gtkwave.png?raw=true)


👉 In GTKWave, you’ll notice that **when no condition matches, the output holds its previous value** → this is exactly the **inferred latch behavior**.

---

## 🔹 Step 3: Synthesis with Yosys

Run Yosys:

```bash
yosys
```

Inside Yosys shell, run the following commands:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_if_Synth.png?raw=true)

---

# 🧪 Day 5 Lab – Incomplete `if` (Version 2) and Latch Inference

In this lab, we test another example of an **incomplete `if` statement** using `incomp_if2.v`. The goal is to observe how synthesis infers **latches** when not all conditions are covered.

---

## 🔹 Step 1: Open Incomplete Files

```bash
gvim incomp_if2.v
```
![iamge alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_if2_code.png?raw=true)


* Opens `incomp_if2.v` and its testbench in split view.

---

## 🔹 Step 2: Simulate with Icarus Verilog + GTKWave

Compile and run the simulation:

```bash
iverilog incomp_if2.v tb_incomp_if2.v
./a.out
```

View waveforms:

```bash
gtkwave tb_incomp_if2.vcd
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_if2_gtkwave.png?raw=true)

👉 In GTKWave, you’ll notice the output **retains old values when no branch is executed**, which is the **inferred latch**.

---

## 🔹 Step 3: Synthesis with Yosys

Run Yosys:

```bash
yosys
```

Inside Yosys shell, execute:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if2.v
synth -top incomp_if2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_if2_synth.png?raw=true)

---
