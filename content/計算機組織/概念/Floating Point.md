

#計算機組織 #概念

用於表示**非整數**的電腦數字系統，涵蓋極小與極大的實數。

---

## 表示形式

科學記號：$(-1)^S \times (1 + \text{Fraction}) \times 2^{\text{Exponent} - \text{Bias}}$

|欄位|Single Precision|Double Precision|
|---|---|---|
|Sign|1 bit|1 bit|
|Exponent|8 bits（Bias = 127）|11 bits（Bias = 1023）|
|Fraction|23 bits|52 bits|
|總長|32 bits|64 bits|

**Hidden bit**：Significand 永遠有隱含的 leading 1，不需明確儲存。 實際 significand 精度：single = 24 bits，double = 53 bits。

---

## IEEE 754 Special Values

|Exponent|Fraction|意義|
|---|---|---|
|`000...0`|`000...0`|±0|
|`000...0`|≠ 0|±Denormalized（非正規化數）|
|`111...1`|`000...0`|±∞|
|`111...1`|≠ 0|NaN（Not a Number）|

- **∞**：可繼續參與運算（如 $F + \infty = \infty$，$F / \infty = 0$）
- **NaN**：非法操作結果（如 $0/0$），可傳播至後續運算

---

## 精度與 Rounding

四種 rounding modes（IEEE 754）：

1. 向 $+\infty$（round up）
2. 向 $-\infty$（round down）
3. 截斷（truncate）
4. 取最近偶數（round to nearest even）— 預設模式，Java 唯一支援

Extra bits 提升精度：**Guard**、**Round**、**Sticky bit**

精度量測：**ulp**（units in the last place）— IEEE 754 保證 ½ ulp 誤差

---

## 重要特性

- FP addition **不滿足結合律**：$(x+y)+z \neq x+(y+z)$（近似值 + 精度有限）
- 平行程式中運算順序不同 → FP 結果可能不同，需驗證合理性而非一致性

---

## 相關

- [[Overflow & Underflow]] — FP 的 overflow/underflow 定義
- [[ISA]] — RISC-V FP 指令（`fadd`、`fmul`、`flw` 等）
- [[第三章 Arithmetic for Computers]] — 完整推導與範例