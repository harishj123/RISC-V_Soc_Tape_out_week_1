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
4. **Tip:** If the text is hard to read:

   * Press `Esc` in gvim.
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
   cell("sky130_fd_sc_hd__a2111o_1") { ... }
   ```

   * `a2111o_1` → Contains 2-input AND and OR logic gates.

3. **To see the corresponding Verilog implementation:**

   ```vim
   :sp ../my_lib/verilog_model/sky130_fd_sc_hd.v
   /sky130_fd_sc_hd__a2111o
   ```

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

