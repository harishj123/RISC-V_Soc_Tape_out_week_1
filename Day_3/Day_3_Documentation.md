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

$Y = \overline{(AB + C)}$

* If **A = 0** (grounded), then:
  $Y = \overline{C}$
  → Only one **inverter** is needed for the output.

📉 **Transistor Count in CMOS:**

* For $Y = \overline{(AB + C)}$: needs **6 transistors**
* For $Y = \overline{C}$: needs **2 transistors**

✅ This demonstrates **constant propagation optimisation**.

---

### 🔸 Boolean Logic Optimisation

Boolean optimisation uses methods like **K-map** or **Quine–McCluskey** to simplify logic expressions.

**Example:**

```verilog
assign y = a ? (b ? c : (c ? a : 0)) : (!c);
```

Naively, this requires **three multiplexers**. Let’s optimise step by step:

1. First MUX simplifies:
   $(c \times 0) + ac = ac$

2. Second MUX simplifies:
   $bc + a\bar{b}c$

3. Third MUX output:
   $a(bc + a\bar{b}c) + a\bar{c}$
   $= abc + a\bar{b}c + a\bar{c}$
   $= ac[b + \bar{b}] + a\bar{c}$
   $= ac + a\bar{c}$
   $= a \oplus c$

✅ Final Optimised Expression:
$y = a \oplus c$

So, instead of **3 multiplexers**, only an **XOR gate** is required. 🎉

---

## 🔹 Sequential Logic Optimisation

Sequential logic optimisation focuses on improving circuits with storage elements (like flip-flops).

### Techniques:

* **Sequential Constant Propagation**
* **State Optimisation**
* **Retiming**
* **Sequential Logic Cloning (Floorplan-Aware Synthesis)**

---

## 🎯 Key Takeaway

Logic optimisation (both combinational and sequential) is **crucial for VLSI design**, reducing **area, power, and delay** while maintaining correct functionality.

---

✨ This is Day 3 of my Digital VLSI Learning Journey 🚀
