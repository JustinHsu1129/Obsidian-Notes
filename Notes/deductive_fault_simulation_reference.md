# Deductive Fault Simulation Equations (2-Input and 3-Input Gates)

In deductive fault simulation, the output fault list $L_z$ of a gate is determined using set operations ($\cup$, $\cap$, $-$, $\Delta$) on the input fault lists ($L_a, L_b, L_c$), combined with the local stuck-at fault at the gate output ($z / \bar{z}$, where $\bar{z}$ represents the complement of the fault-free output value).

| Gate Type | Input State / Condition | 2-Input Equation ($L_z$) | 3-Input Equation ($L_z$) |
| :--- | :--- | :--- | :--- |
| **AND** | All Non-Controlling ($1$) | $L_a \cup L_b \cup \{z/0\}$ | $L_a \cup L_b \cup L_c \cup \{z/0\}$ |
| | All Controlling ($0$) | $(L_a \cap L_b) \cup \{z/1\}$ | $(L_a \cap L_b \cap L_c) \cup \{z/1\}$ |
| | Mixed ($a=0, b=1$) / ($a=0, b=1, c=1$) | $(L_a - L_b) \cup \{z/1\}$ | $(L_a - (L_b \cup L_c)) \cup \{z/1\}$ |
| | Mixed ($a=0, b=0, c=1$) | — | $((L_a \cap L_b) - L_c) \cup \{z/1\}$ |
| **NAND** | All $1$ (Output $0$) | $L_a \cup L_b \cup \{z/1\}$ | $L_a \cup L_b \cup L_c \cup \{z/1\}$ |
| | All $0$ (Output $1$) | $(L_a \cap L_b) \cup \{z/0\}$ | $(L_a \cap L_b \cap L_c) \cup \{z/0\}$ |
| | Mixed ($a=0, b=1$) / ($a=0, b=1, c=1$) | $(L_a - L_b) \cup \{z/0\}$ | $(L_a - (L_b \cup L_c)) \cup \{z/0\}$ |
| | Mixed ($a=0, b=0, c=1$) | — | $((L_a \cap L_b) - L_c) \cup \{z/0\}$ |
| **OR** | All Non-Controlling ($0$) | $L_a \cup L_b \cup \{z/1\}$ | $L_a \cup L_b \cup L_c \cup \{z/1\}$ |
| | All Controlling ($1$) | $(L_a \cap L_b) \cup \{z/0\}$ | $(L_a \cap L_b \cap L_c) \cup \{z/0\}$ |
| | Mixed ($a=1, b=0$) / ($a=1, b=0, c=0$) | $(L_a - L_b) \cup \{z/0\}$ | $(L_a - (L_b \cup L_c)) \cup \{z/0\}$ |
| | Mixed ($a=1, b=1, c=0$) | — | $((L_a \cap L_b) - L_c) \cup \{z/0\}$ |
| **NOR** | All $0$ (Output $1$) | $L_a \cup L_b \cup \{z/0\}$ | $L_a \cup L_b \cup L_c \cup \{z/0\}$ |
| | All $1$ (Output $0$) | $(L_a \cap L_b) \cup \{z/1\}$ | $(L_a \cap L_b \cap L_c) \cup \{z/1\}$ |
| | Mixed ($a=1, b=0$) / ($a=1, b=0, c=0$) | $(L_a - L_b) \cup \{z/1\}$ | $(L_a - (L_b \cup L_c)) \cup \{z/1\}$ |
| | Mixed ($a=1, b=1, c=0$) | — | $((L_a \cap L_b) - L_c) \cup \{z/1\}$ |
| **XOR** | Any Input Combination (Parity check) | $(L_a \Delta L_b) \cup \{z/\bar{z}\}$ | $(L_a \Delta L_b \Delta L_c) \cup \{z/\bar{z}\}$ |
| **XNOR** | Any Input Combination (Parity check) | $(L_a \Delta L_b) \cup \{z/\bar{z}\}$ | $(L_a \Delta L_b \Delta L_c) \cup \{z/\bar{z}\}$ |

---

### Key Rules of Thumb:
1. **And / Or Gates (Controlling Values):** If inputs match the controlling value, intersect their fault lists. Subtract any non-controlling input fault lists. If all inputs are non-controlling, union all input fault lists.
2. **Nand / Nor Gates:** Follow the exact same input set rules as their base gates, but invert the local output stuck-at fault ($z/0$ vs $z/1$) to reflect the inverted output polarity.
3. **Xor / Xnor Gates:** Use symmetric difference ($\Delta$) to propagate faults appearing on an odd number of input lines.