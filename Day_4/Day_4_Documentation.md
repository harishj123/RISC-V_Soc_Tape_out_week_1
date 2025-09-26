Got it! Here's a polished, detailed, and **GitHub-ready version** for Day 4, with a clear welcome intro, detailed explanation, and attractive formatting:

````markdown
# 🚀 Day 4: GLS, Blocking vs Non-Blocking, and Synthesis-Simulation Mismatch

Welcome to **Day 4** of our digital design journey! Today we dive into **Gate Level Simulation (GLS)**, understand **blocking vs non-blocking assignments**, and learn why **synthesis vs simulation mismatches** occur. By the end of this day, you’ll understand how to validate your design **after synthesis** and ensure it works correctly with timing.

---

## 🔹 What is GLS?

**GLS** stands for **Gate Level Simulation**.  

- In **RTL simulation**, the **testbench runs with the RTL code** as the Design Under Test (DUT).  
- In **GLS**, the **testbench runs with the synthesized netlist** as the DUT.  
- The **netlist** is logically the same as the RTL but expressed using **standard cell gates and flip-flops** from the target technology library.  

### Example Workflow:

```text
RTL simulation:      Testbench + RTL code
GLS simulation:      Testbench + Synthesized netlist
````

---

## ❓ Why GLS is Needed?

1. **Verify logical correctness after synthesis**

   * Synthesis tools convert RTL into gate-level netlists.
   * GLS ensures that the **functional behavior of the netlist matches the RTL**.

2. **Timing-aware verification**

   * RTL simulation **ignores gate delays**.
   * Netlist simulation includes **timing information** if delay annotations are present.
   * GLS checks whether the design can **meet target clock frequencies**.

3. **Delay-annotated GLS**

   * For **realistic timing validation**, GLS should be run with **annotated delays** from the standard cell library.
   * Helps catch setup/hold violations and other timing issues **before tapeout**.

4. **Detect synthesis-related mismatches**

   * Sometimes, coding style issues in RTL (e.g., improper blocking/non-blocking usage, incomplete sensitivity lists) may cause the **netlist to behave differently** from RTL.
   * GLS identifies these issues early.

5. **Final verification confidence**

   * GLS acts as a **last step of functional and timing validation** before fabrication.

---
```

