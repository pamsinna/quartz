

#計算機組織 #概念

**Arithmetic Logic Unit**：執行算術（加、減、乘、除）與邏輯運算（AND、OR）的硬體單元。

---

## 基本架構

### 1-bit ALU

- **Logical unit**（Fig. A.5.1）：執行 AND / OR
- **Full adder**（Fig. A.5.2）：3 inputs（a, b, CarryIn）→ 2 outputs（Sum, CarryOut）
    - 又稱 (3,2) adder
    - Half adder：只有 a, b 兩個 input，又稱 (2,2) adder

CarryOut 邏輯： $$\text{CarryOut} = (b \cdot \text{CarryIn}) + (a \cdot \text{CarryIn}) + (a \cdot b)$$

### 32-bit ALU（Ripple Carry）

- 32 個 1-bit ALU 串接（Fig. A.5.7）
- 前一個 ALU 的 CarryOut 接到下一個的 CarryIn
- `Operation` signal 控制執行 AND / OR / Add / Subtract

Subtraction 實作：對第二個 operand 取 two's complement（negate + 1），再相加。

---

## Integer 乘除法硬體

### Multiplication（第一版，Fig. 3.3）

- 64-bit Multiplicand register + 64-bit Product register + 32-bit Multiplier register
- 每次迭代：有條件加法 → Multiplicand 左 shift → Multiplier 右 shift（共 32 次）

### Multiplication（優化版，Fig. 3.5）

- ALU 縮減至 32-bit，Product 右 shift 取代 Multiplicand 左 shift
- Multiplier 合併入 Product register 右半部

### Faster Multiplication（Fig. 3.7）

- 31 個 32-bit adder 組成 parallel tree
- 延遲從 32 次加法 → $\log_2 32 = 5$ 次

### Division（第一版，Fig. 3.8）

- 64-bit Divisor / ALU / Remainder + 32-bit Quotient
- 每次迭代：減法 → 視 remainder 正負決定 quotient bit → Divisor 右 shift（共 33 次）

### Division（優化版，Fig. 3.11）

- ALU / Divisor / Quotient 縮減至 32-bit，Remainder 改為左 shift

---

## FP 算術硬體

- FP Adder（Fig. 3.15）：步驟為 exponent 對齊 → significand 加法 → normalize → round，通常需多個 clock cycles，可 pipelined
- FP Multiplier：複雜度相近，significand 改為乘法
- RISC-V 使用獨立的 FP register file（`f0`–`f31`）

---

## 相關

- [[Two's Complement]] — Subtraction 的實作基礎
- [[Overflow & Underflow]] — ALU 結果的邊界情況
- [[Floating Point]] — FP 算術硬體
- [[ISA]] — RISC-V 的 ALU 相關指令
- [[第三章 Arithmetic for Computers]] — 完整硬體設計說明