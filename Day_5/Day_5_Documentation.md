
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
