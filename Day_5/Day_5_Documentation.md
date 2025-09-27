
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

In this lab, we test another example of an **incomplete `if` statement** using `incomp_if2.v`. The goal is to observe how synthesis infers **latches** when not all conditions are covered.

---

## 🔹 Step 1: Open Incomplete Files

```bash
gvim incomp_if2.v
```
![iamge alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_if2_code.png?raw=true)


* Opens `incomp_if2.v` 

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


In this lab, we test an **incomplete `case` statement** using `incomp_case.v`. Just like with `if`, missing branches in a `case` statement can lead to **inferred latches**.

---

## 🔹 Step 1: Open Incomplete Files

```bash
gvim incomp_case.v 
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_case_code.png?raw=true)

* Opens `incomp_case.v`

---

## 🔹 Step 2: Simulate with Icarus Verilog + GTKWave

Compile and run the simulation:

```bash
iverilog incomp_case.v tb_incomp_case.v
./a.out
```

View waveforms:

```bash
gtkwave tb_incomp_case.vcd
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_case_gtkwave.png?raw=true)

👉 In GTKWave, you’ll notice that **when `sel` is not covered by the case items, the output holds its old value** → latch inference.

---

## 🔹 Step 3: Synthesis with Yosys

Run Yosys:

```bash
yosys
```

Inside Yosys shell, execute:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_case.v
synth -top incomp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/incomp_case_synth.png?raw=true)

---

Got it 👍 Let’s document the **complete flow for `comp_case.v`** (open, simulate with GTKWave, and synthesize with Yosys) in the same GitHub-friendly style you’ve been using.

---

In this lab, we implement a **multiplexer** using a `case` statement. We’ll simulate the design, observe waveforms in GTKWave, and then synthesize it using Yosys.

---

## 🔹 Step 1: Open Files

```bash
ls *case*
gvim comp_case.v tb_comp_case.v -o
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/comp_case_code.png?raw=true)

* Opens both the design (`comp_case.v`) and the testbench (`tb_comp_case.v`) in split view.

---

## 🔹 Step 2: Simulation with Icarus Verilog + GTKWave

Compile and run the simulation:

```bash
iverilog comp_case.v tb_comp_case.v
./a.out
```

Open the waveform:

```bash
gtkwave tb_comp_case.vcd
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/comp_case_gtkwave.png?raw=true)

👉 In GTKWave, you will observe **`sel` selecting one of the inputs (i0, i1, i2, or default value)**. This confirms the **MUX behavior**.

---

## 🔹 Step 3: Synthesis with Yosys

Start Yosys:

```bash
yosys
```

Inside Yosys shell, run:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog comp_case.v
synth -top comp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/comp_case_synth.png?raw=true)

👉 The schematic displayed by `show` will now be a **multiplexer structure** (not a latch, since we added all cases/default).

---

Practice output

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/practice_code.png?raw=true)

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/practice_gtkwave.png?raw=true)

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/practice_synth.png?raw=true)

---

### **1️⃣ partial_case_assign.v**

**Open file:**

```bash
gvim partial_case_assign.v
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/partial_code.png?raw=true)


**Synthesis using Yosys:**

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog partial_case_assign.v
synth -top partial_case_assign
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/partial_synth.png?raw=true)

* `read_liberty` → loads the standard cell library.
* `read_verilog` → reads your RTL file.
* `synth -top` → synthesizes with the top module.
* `abc` → technology mapping to the library.
* `show` → opens schematic viewer for verification.

---

### **2️⃣ bad_case.v**

**Open RTL file:**

```bash
gvim bad_case.v
```

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/bad_code.png?raw=true)


**Run simulation:**

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd__tt_025C_1v80.lib bad_case.v tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/bad_gtk.png?raw=true)

* Include any library files needed (`primitives.v`, cell library).
* Compile with `iverilog`.
* Run simulation (`./a.out`) to generate VCD.
* Open waveform in GTKWave.

**Synthesis using Yosys:**

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_case.v
synth -top bad_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr tb_bad_case_net.v
show
```
![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/bad_synth.png?raw=true)

* `write_verilog -noattr` → generates netlist without attributes.
* `show` → opens schematic viewer.

---

## **1️⃣ 32:1 MUX using `for` loop inside always**

Here, we select 1 out of 32 inputs. Writing 32 `if` or `case` statements is tedious. We use a `for` loop to evaluate which input to select.

```verilog
module mux32to1(
    input  [31:0] i,  // 32 input bits
    input  [4:0]  sel, // select line
    output reg    y
);
    integer k;

    always @(*) begin
        y = 0;
        for(k = 0; k < 32; k = k + 1) begin
            if(sel == k)
                y = i[k];
        end
    end
endmodule
```

✅ **Notes:**

* `for` loop evaluates logic but does **not instantiate hardware multiple times**.
* `always @(*)` ensures combinational logic.

---

## **2️⃣ Replicating hardware with `generate`**

Here, hardware is **instantiated multiple times**. Example: **AND gates replicated 4 times**.

```verilog
module and_array(
    input  [3:0] a, b,
    output [3:0] y
);
    genvar i;

    generate
        for(i = 0; i < 4; i = i + 1) begin : gen_and
            and u_and(y[i], a[i], b[i]);
        end
    endgenerate
endmodule
```

✅ **Notes:**

* `generate` creates **multiple hardware blocks**, not just evaluating values.
* `genvar` is **required** for generate loops.

---

## **3️⃣ Ripple Carry Adder using `generate`**

We can build an N-bit ripple carry adder by replicating full adders.

```verilog
module full_adder(
    input  a, b, cin,
    output sum, cout
);
    assign sum  = a ^ b ^ cin;
    assign cout = (a & b) | (b & cin) | (a & cin);
endmodule

module ripple_carry_adder #(parameter N = 8)(
    input  [N-1:0] a, b,
    input  cin,
    output [N-1:0] sum,
    output cout
);
    wire [N:0] c;
    assign c[0] = cin;
    assign cout = c[N];

    genvar i;
    generate
        for(i = 0; i < N; i = i + 1) begin : adder_gen
            full_adder u_fa(
                .a(a[i]),
                .b(b[i]),
                .cin(c[i]),
                .sum(sum[i]),
                .cout(c[i+1])
            );
        end
    endgenerate
endmodule
```

---

open ***mux_generate.v***

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/mux_generator_code.png?raw=true
)

GTKWave

![image alt](https://github.com/harishj123/RISC-V_Soc_Tape_out_week_1/blob/main/Day_5/mux_generator_code.png?raw=true
)

