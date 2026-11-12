# Mathematical Theory, Architecture & VLSI Implementation of Carry Look-Ahead Adders

This document provides a rigorous architectural, mathematical, and circuit-level breakdown of the **8-Bit and 16-Bit Carry Look-Ahead Adders (CLA)** implemented in **Cadence Virtuoso 90nm GPDK CMOS**.

---

## 1. Fundamentals of Binary Addition

In a conventional Full Adder (FA), the $i$-th sum bit $S_i$ and the output carry $C_{i+1}$ are functions of the inputs $A_i$, $B_i$, and the incoming carry $C_i$:

$$S_i = A_i \oplus B_i \oplus C_i$$
$$C_{i+1} = A_i B_i + C_i (A_i \oplus B_i)$$

### 1.1 The Ripple Carry Bottleneck
In a standard **Ripple Carry Adder (RCA)**, the carry must ripple through each 1-bit full adder stage sequentially from the Least Significant Bit (LSB) to the Most Significant Bit (MSB).

- **Worst-case delay:**
  $$t_{\text{RCA}} = t_{\text{setup}} + (n - 1) t_{\text{carry\_propagation}} + t_{\text{sum}}$$
- For an $n$-bit adder, the latency scales linearly:
  $$\text{Delay}_{\text{RCA}} = \mathcal{O}(n)$$
For 8-bit, 16-bit, and 32-bit arithmetic datapath units in high-speed microprocessors, the ripple carry delay becomes the primary critical path limiter.

---

## 2. Carry Look-Ahead (CLA) Principle

The Carry Look-Ahead Adder overcomes the $\mathcal{O}(n)$ delay bottleneck by computing all carries in parallel using two auxiliary Boolean signals defined at each bit position $i$:

### 2.1 Carry Generate ($G_i$)
A carry is **generated** when both input bits are high, independent of the incoming carry:
$$G_i = A_i \cdot B_i$$

### 2.2 Carry Propagate ($P_i$)
An incoming carry is **propagated** through the stage when at least one input bit is high (or exactly one for the exclusive form):
$$P_i = A_i \oplus B_i \quad \text{(or } A_i + B_i\text{)}$$

Using $G_i$ and $P_i$, the next carry bit $C_{i+1}$ can be rewritten as:
$$C_{i+1} = G_i + P_i \cdot C_i$$

The sum output is:
$$S_i = P_i \oplus C_i$$

---

## 3. Carry Recurrence Expansion (Look-Ahead Generator)

By recursively substituting $C_i$ into the recurrence relation, every carry bit can be expressed strictly in terms of the primary inputs ($A, B$) and the initial carry-in $C_0$:

$$\begin{aligned}
C_1 &= G_0 + P_0 C_0 \\
C_2 &= G_1 + P_1 C_1 \\
    &= G_1 + P_1 G_0 + P_1 P_0 C_0 \\
C_3 &= G_2 + P_2 C_2 \\
    &= G_2 + P_2 G_1 + P_2 P_1 G_0 + P_2 P_1 P_0 C_0 \\
C_4 &= G_3 + P_3 C_3 \\
    &= G_3 + P_3 G_2 + P_3 P_2 G_1 + P_3 P_2 P_1 G_0 + P_3 P_2 P_1 P_0 C_0
\end{aligned}$$

### 3.1 Fan-In Limits and Block Partitioning
Notice that generating $C_4$ requires:
- An AND gate with fan-in = 5 ($P_3 P_2 P_1 P_0 C_0$)
- An OR gate with fan-in = 5

In static CMOS logic, connecting more than 4 PMOS transistors in series causes quadratic rise-time degradation ($\tau \propto N^2$) due to substrate body effect and parasitic junction capacitance.

Therefore, practical industrial CLA architectures partition adders into **4-bit blocks** (slices) and use a hierarchical look-ahead carry generator:

$$\begin{aligned}
P_{\text{group}} &= P_3 \cdot P_2 \cdot P_1 \cdot P_0 \\
G_{\text{group}} &= G_3 + P_3 G_2 + P_3 P_2 G_1 + P_3 P_2 P_1 G_0
\end{aligned}$$

The block carry-out is then simply:
$$C_{\text{block\_out}} = G_{\text{group}} + P_{\text{group}} \cdot C_{\text{block\_in}}$$

---

## 4. Architecture of the 8-Bit & 16-Bit Adders

```
                +-------------------------------------------------+
                |                  8-Bit CLA                      |
                +-------------------------------------------------+
                         |                                 |
                         v                                 v
               +--------------------+            +--------------------+
               |  Block 0 (Bits 0-3)|            |  Block 1 (Bits 4-7)|
               |    4-Bit CLA       |-- C4 ----> |    4-Bit CLA       |-- Cout
               +--------------------+            +--------------------+
                 ^        ^   |                    ^        ^   |
                 |        |   v                    |        |   v
                A[3:0]  B[3:0] S[3:0]             A[7:4]  B[7:4] S[7:4]
                 |
                Cin
```

### 4.1 8-Bit Integration
- **Inputs:** Operand $A[7:0]$, Operand $B[7:0]$, Carry-in $C_{in}$
- **Outputs:** Sum vector $S[7:0]$, Carry-out $C_{out}$
- **Internal Stages:**
  1. **Bit-Level $P$ & $G$ Generation:** 8 parallel XOR gates ($P_i$) and 8 parallel AND gates ($G_i$).
  2. **Look-Ahead Carry Generator:** Computes internal carries $C_1, C_2, C_3, C_4, C_5, C_6, C_7$ and $C_{out}$.
  3. **Sum Generation:** 8 parallel XOR gates computing $S_i = P_i \oplus C_i$.

### 4.2 16-Bit Extension
By cascading two 8-bit CLA macros (or four 4-bit blocks) through inter-block look-ahead logic, the architecture scales to 16 bits with minimal latency penalty ($\mathcal{O}(\log_4 n)$ prefix delay).

---

## 5. Circuit Comparison: RCA vs. CLA vs. CSA

| Metric | Ripple Carry Adder (RCA) | Carry Look-Ahead (CLA) | Carry Select Adder (CSA) |
| :--- | :--- | :--- | :--- |
| **Delay Complexity** | $\mathcal{O}(n)$ | $\mathcal{O}(\log n)$ / $\mathcal{O}(1)$ local | $\mathcal{O}(\sqrt{n})$ |
| **Area / Transistor Count** | Minimal ($\approx 28 \times n$) | Moderate ($\approx 40-50 \times n$) | High ($\approx 2 \times \text{RCA} + \text{MUX}$) |
| **Critical Path for 8-Bit** | 8 full adder stages | Setup + 2-level Carry + Sum | Setup + Add + MUX select |
| **Power Dissipation** | Low static / dynamic | Moderate switching activity | Higher due to redundant paths |
| **Typical Use Case** | Low-power / IoT sensors | High-speed ALUs / DSP units | Critical-path pipelined units |

---

## 6. Transistor-Level CMOS Sizing (90nm Process)

In the Cadence Generic 90nm Process (`gpdk090`):
- **Minimum Gate Length ($L_{min}$):** $90\,\text{nm} = 0.09\,\mu\text{m}$
- **Supply Voltage ($V_{DD}$):** $1.8\,\text{V}$ (for IO / thick oxide standard cells) or $1.0-1.2\,\text{V}$ (core logic).
- **Mobility Matching ($W_p / W_n$):**
  Due to hole mobility being approximately half of electron mobility in silicon ($\mu_n \approx 2.5 \times \mu_p$), PMOS pull-up transistors are sized wider:
  $$\left(\frac{W}{L}\right)_{\text{PMOS}} \approx 2.0 - 2.5 \times \left(\frac{W}{L}\right)_{\text{NMOS}}$$
- **Series Stacking Sizing:**
  - 2-input NAND: Series NMOS pull-down transistors sized to $2 \times W_n$ to maintain equivalent resistance $R_{on}$.
  - 2-input NOR: Series PMOS pull-up transistors sized to $2 \times W_p$.
  - XOR Gate: Implemented using complementary transmission gates or pass-transistor logic to minimize transistor count and reduce parasitic input capacitance.
