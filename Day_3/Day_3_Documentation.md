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
