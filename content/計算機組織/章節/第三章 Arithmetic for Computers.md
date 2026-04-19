

#計算機組織 #章節

---

## Chapter Goals

- 理解實數的**表示方式**
- 理解**算術演算法**
- 理解實作這些演算法的**硬體**，以及這些設計對指令集的影響
- 學會如何利用這些知識讓**算術密集型程式**加速執行

---

## 3.1 Introduction

> 目標：理解 real numbers 的表示法、算術演算法、相對應硬體，以及對 instruction sets 的影響。

---

## 3.2 Addition and Subtraction

### Integer Addition

- [[Overflow & Underflow|Overflow]] 發生在結果超出範圍時
- 不同符號（一正一負）相加 → **不會 overflow**（結果不可能大於任一個 operand）
- 兩個正數相加：結果 sign bit 為 1 → overflow
- 兩個負數相加：結果 sign bit 為 0 → overflow

### Integer Subtraction

- 概念：negate 第二個 operand，然後做加法
    
    - $c - b = c + (-b)$
    - 例：$7 + (-6)$，其中 $-6$ 用 [[Two's Complement]] 表示
- Overflow 偵測規則：
    

|運算|情況|判定|
|---|---|---|
|同號相加 / 同號相減|無 overflow|✓|
|從負數減正數（如 $-8 - 2$）|結果 sign 為 1|overflow|
|從正數減負數|結果 sign 為 0|overflow|

- **Unsigned integers** 的 overflow：由 compiler 以 branch instruction 偵測
    
    - 加法：sum < 任一 addend → overflow
    - 減法：difference > minuend → overflow
- **Saturating operations**（多媒體應用）：overflow 時 clamp 到最大可表示值（如 audio clipping、video saturation）
    

---

## 3.3 Multiplication

### 基本概念

二進位乘法步驟：

1. 從右到左取 multiplier 的每一位
2. 將 multiplicand 與該位相乘
3. 將中間積向左 shift 一位

乘積長度：$n$-bit × $m$-bit = $(n+m)$-bit（忽略 sign bit）→ 可能 overflow

### 第一版 Multiplication Hardware（Fig. 3.3）

- 64-bit Multiplicand register（每次左 shift）
- 64-bit Product register（初始化為 0）
- 32-bit Multiplier register（每次右 shift）

演算法（每次迭代三步，共 32 次）：

1. Multiplier$_0$ = 1 → Product += Multiplicand
2. Multiplicand 左 shift 1 bit
3. Multiplier 右 shift 1 bit

### 優化版 Multiplication Hardware（Fig. 3.5）

- Multiplicand register 與 [[ALU]] 縮減至 **32-bit**
- Multiplier 放入 Product register 右半部（Product 擴展為 65-bit 以容納 carry-out）
- Product 改為**右 shift**（而非 Multiplicand 左 shift）
- 節省 register 資源

### Faster Multiplier（Fig. 3.7）

- 提供 **31 個 32-bit adder**（每個 multiplier bit 對應一個）
- 組成 **parallel tree**
- 等待時間從 32 次加法縮短為 $\log_2(32) = 5$ 次

### RISC-V Multiplication 指令

|指令|全名|功能|
|---|---|---|
|`mul`|multiply|64-bit 乘積的低 32 bits（整數乘積）|
|`mulh`|multiply high|64-bit 乘積的高 32 bits（兩個 signed）|
|`mulhu`|multiply high unsigned|高 32 bits（兩個 unsigned）|
|`mulhsu`|multiply high signed/unsigned|高 32 bits（一 signed 一 unsigned）|

用 `mulh` 的結果來確認是否 64-bit overflow。

範例（unsigned 乘法）：

```
mulh rdh, rs1, rs2   # 高 32 bits → rdh
mul  rdl, rs1, rs2   # 低 32 bits → rdl
```

---

## 3.4 Division

### 基本概念

- 先檢查 divisor 是否為 0
    
- Long division approach：
    
    - `len(divisor) ≤ len(dividend)` → quotient 為 1，執行減法
    - 否則 → quotient 為 0，繼續下一位
- **Restoring division**（HW 實作用）：先減，若 remainder < 0 → 加回 divisor
    
- **Signed division**：先取絕對值計算，再調整 quotient 與 remainder 的符號
    

$n$-bit 運算元 → $n$-bit quotient + $n$-bit remainder

### Division Algorithm（Fig. 3.9）

假設 dividend 與 divisor 皆為正 32-bit 值，共迭代 **33 次**：

1. Remainder -= Divisor 2a. Remainder ≥ 0 → quotient bit = 1 2b. Remainder < 0 → 加回 Divisor，quotient bit = 0
2. Divisor 右 shift 1 bit

### Division Hardware（Fig. 3.8）

- 64-bit Divisor、[[ALU]]、Remainder register
- 32-bit Quotient register
- Divisor 初始放左半部，每次右 shift

### 優化版 Division Hardware（Fig. 3.11）

- Divisor、ALU、Quotient register 皆縮減至 **32-bit**
- Remainder 改為左 shift
- Quotient register 合併至 Remainder register 右半部

### 更快的 Division：SRT Division

- 乘法的多 adder trick **不適用於除法**（需先知道 difference 的符號才能進行下一步）
- SRT division：每步預測**數個 quotient bits**，用 **table lookup** 實作
    - 例：取 remainder 高 6 bits + divisor 高 4 bits → 查表決定猜測值

### RISC-V Division 指令

|指令|功能|
|---|---|
|`div`|signed 除法（商）|
|`rem`|signed 除法（餘數）|
|`divu`|unsigned 除法（商）|
|`remu`|unsigned 除法（餘數）|

> 延伸閱讀：Signed Division (p. 203–205)、Hardware/Software Interface (p. 206)（overflow 與除以 0 不由硬體報錯）

---

## 3.5 [[Floating Point]]

### 基本概念

- 用於表示**非整數**，包含極小或極大的數
- 採用**科學記號**：小數點左方只有一位非零數字
    - 十進位：$-2.34 \times 10^6$
    - 二進位（normalized）：$\pm 1.xxxxxxx_2 \times 2^{yyyy}$
- C 的 `float`（single precision）與 `double`（double precision）

### Floating-Point Representation

表示形式：$(-1)^S \times F \times 2^E$

|欄位|說明|
|---|---|
|Sign ($S$)|0 = 正，1 = 負|
|Exponent|指數值（含 sign）|
|Fraction|小數部分（mantissa）|

- **[[Overflow & Underflow|Overflow]]**：正 exponent 超出 exponent field 範圍
- **[[Overflow & Underflow|Underflow]]**：負 exponent 超出 exponent field 範圍

### IEEE 754 Standard

|精度|Sign|Exponent|Fraction|Bias|
|---|---|---|---|---|
|Single|1 bit|8 bits|23 bits|127|
|Double|1 bit|11 bits|52 bits|1023|

重要概念：

- **Hidden bit**：significand 永遠有隱含的 leading 1（不需明確儲存）
    - Significand = 1 + Fraction（實際為 24-bit / 53-bit）
- **Biased exponent**：Actual Exponent = Exponent − Bias
    - 確保 Exponent 欄位為 unsigned
    - 例：actual exponent = −1 → 儲存值 = −1 + 127 = 126 = `0111 1110`

### Single Precision 範圍

保留值：Exponent = `00000000` 與 `11111111`

||Exponent|Fraction|值|
|---|---|---|---|
|最小|`00000001`（=1, actual=−126）|`000...0`（sig=1.0）|$\pm 1.0 \times 2^{-126} \approx \pm 1.2 \times 10^{-38}$|
|最大|`11111110`（=254, actual=127）|`111...1`（sig≈2.0）|$\pm 2.0 \times 2^{127} \approx \pm 3.4 \times 10^{38}$|

### Double Precision 範圍

||Actual Exponent|值|
|---|---|---|
|最小|−1022|$\pm 1.0 \times 2^{-1022} \approx \pm 2.2 \times 10^{-308}$|
|最大|+1023|$\pm 2.0 \times 2^{1023} \approx \pm 1.8 \times 10^{308}$|

### Special Values（IEEE 754 Encoding）

|Exponent|Fraction|意義|
|---|---|---|
|`000...0`|`000...0`|±0|
|`000...0`|≠ 0|±Denormalized numbers|
|`111...1`|`000...0`|±∞（Infinity）|
|`111...1`|≠ 0|NaN（Not a Number）|

- **∞**：可繼續參與運算，避免需要 overflow check（如 $F + \infty = \infty$，$F / \infty = 0$）
- **NaN**：非法或未定義操作（如 $0.0 / 0.0$）

### Floating-Point 表示範例

$-0.75_{10}$ 的 single precision 表示：

$$-0.75 = -1.1_2 \times 2^{-1}$$

- $S = 1$
- Exponent = $-1 + 127 = 126 = \texttt{01111110}_2$
- Fraction = $\texttt{1000...00}_2$

```
1 | 01111110 | 10000000000000000000000
```

### IEEE 754-2008 更新

在 IEEE 754-1985 基礎上新增：

- **Half precision**（16-bit）：1-bit sign、5-bit exponent（bias=15）、10-bit fraction
- **Quadruple precision**（128-bit）：1-bit sign、15-bit exponent（bias=262,143）、112-bit fraction

---

## 3.5 Floating-Point Arithmetic

### Floating-Point Addition 步驟

1. **對齊 exponent**：將較小 exponent 的數的 significand 右 shift，直到兩者 exponent 相同
2. **相加 significands**
3. **Normalize 結果**，並檢查 overflow / underflow
    - Single precision：$-126 \leq \text{Exponent} \leq 127$
4. **Rounding**

Rounding 規則：

- 右方數字 0–4 → 捨去（round down）
- 右方數字 5–9 → 進位（round up）
- 若 rounding 後不再 normalized → 重複 Step 3

**FP Adder**（Fig. 3.15）：比 integer adder 複雜得多，通常需要**數個 clock cycles**，可 pipelined。

#### 範例（二進位，4 digits）

$0.5 + (-0.4375)$：

|步驟|計算|
|---|---|
|對齊 exponent|$1.000_2 \times 2^{-1}$ 和 $-0.111_2 \times 2^{-1}$|
|相加|$0.001_2 \times 2^{-1}$|
|Normalize|$1.000_2 \times 2^{-4}$|
|結果|$0.0625_{10}$|

#### 範例（十進位，4 decimal digits）

$9.999 \times 10^1 + 1.610 \times 10^{-1}$：

|步驟|計算|
|---|---|
|對齊|$9.999 \times 10^1 + 0.016 \times 10^1$|
|相加|$10.015 \times 10^1$|
|Normalize|$1.0015 \times 10^2$|
|Round|$1.002 \times 10^2$|

---

### Floating-Point Multiplication 步驟

1. **計算 exponent**：直接相加兩個 exponent
    - Biased exponents：$(E_1 + \text{Bias}) + (E_2 + \text{Bias}) - \text{Bias}$
2. **相乘 significands**
3. **Normalize 結果**，並檢查 overflow / underflow
4. **Rounding**（符合位數長度）
5. **決定 sign**：同號 → 正；異號 → 負

#### 範例（二進位，4 digits）

$0.5 \times (-0.4375)$：

|步驟|計算|
|---|---|
|加 exponent|$-1 + (-2) = -3$（biased: $124$）|
|乘 significands|$1.000_2 \times 1.110_2 = 1.110_2$|
|Normalize|$1.110_2 \times 2^{-3}$（無 overflow）|
|Round|不需調整|
|Sign|正 × 負 = 負|
|結果|$-1.110_2 \times 2^{-3} = -0.21875_{10}$|

---

### FP Hardware

- FP Multiplier 複雜度約等於 FP Adder
- 通常支援：加、減、乘、除、倒數、平方根、FP⇔integer 轉換
- RISC-V 使用**獨立的 FP registers**（32 個）

---

### RISC-V FP Registers 與指令

FP registers：`f0` … `f31`

- Single precision 值存在 double-precision register 的低 32 bits
- **`f0` 不是 hardwired 為 0**（與整數 `x0` 不同）

**Load/Store：**

|指令|說明|
|---|---|
|`flw` / `fsw`|Single-precision load/store|
|`fld` / `fsd`|Double-precision load/store|

**算術運算：**

|指令|說明|
|---|---|
|`fadd.s/d`, `fsub.s/d`|加、減|
|`fmul.s/d`, `fdiv.s/d`|乘、除|
|`fsqrt.s/d`|平方根|

**比較：**

|指令|說明|
|---|---|
|`feq.s/d`, `flt.s/d`, `fle.s/d`|比較，結果寫入整數 register（0 或 1）|

搭配 `beq` / `bne` 使用 FP 比較結果做 branch。

**指令格式：**

- Load：I-type
- Store：S-type
- 算術：R-type

#### C ↔ RISC-V 範例（Fahrenheit to Celsius）

```c
float f2c(float fahr) {
    return ((5.0/9.0) * (fahr - 32.0));
}
```

```asm
f2c:
    flw    f0, const5(x3)   # f0 = 5.0
    flw    f1, const9(x3)   # f1 = 9.0
    fdiv.s f0, f0, f1       # f0 = 5.0/9.0
    flw    f1, const32(x3)  # f1 = 32.0
    fsub.s f10, f10, f1     # f10 = fahr - 32.0
    fmul.s f10, f0, f10     # f10 = (5/9) * (fahr-32)
    jalr   x0, 0(x1)        # return
```

---

### Accurate Arithmetic

IEEE 754 提供額外精度控制：

**Extra bits：**

- **Guard**：中間計算時右方第 1 個額外 bit
- **Round**：中間計算時右方第 2 個額外 bit
- **Sticky bit**：若 round bit 右方有任何非零 bit 則設為 1（區分如 $0.50...00$ 與 $0.50...01$）

**四種 Rounding modes：**

1. 向 $+\infty$ rounding（always round up）
2. 向 $-\infty$ rounding（always round down）
3. 截斷（Truncate）
4. 取最近偶數（Round to nearest even）—— Java 只支援此模式

**精度量測：ulp（units in the last place）**

- 實際值與可表示值之間，最低有效位的誤差 bits 數
- IEEE 754 保證 ½ ulp

#### Guard/Round 範例

計算 $2.34 + 0.0256$（3 significant digits）：

|方式|結果|
|---|---|
|有 guard/round|$2.3656 \rightarrow 2.37$（正確）|
|無 guard/round|$2.34 + 0.02 = 2.36$（差 1 ulp）|

> 延伸閱讀：C procedure with 2D matrices (p. 226–229)、ulp for IEEE 754 (p. 230)

---

## 3.6 [[Parallelism]] and Computer Arithmetic: Subword Parallelism

多媒體與音訊應用常對**向量資料**執行相同操作。

透過**分割 128-bit adder**，可同時平行處理：

- 16 個 8-bit 運算元
- 8 個 16-bit 運算元
- 4 個 32-bit 運算元
- 2 個 64-bit 運算元

此類平行性稱為 **Subword Parallelism**，也稱為：

- Data-level parallelism
- Vector parallelism
- **SIMD**（Single Instruction, Multiple Data）

---

## 3.9 Fallacies and Pitfalls

### Fallacy：右移 = 整數除以 2 的次方？

|情況|是否成立|
|---|---|
|Unsigned integers|✓ 成立|
|Signed integers|✗ 不成立|

Signed integer 範例（$-5 / 4 = -1$）：

- Logical right shift（補 0）：`11111011 >>> 2 = 00111110` = $+62$（❌）
- Arithmetic right shift（補 sign bit）：`11111011 >> 2 = 11111110` = $-2$（❌，應為 $-1$）

### Pitfall：Floating-point addition is NOT associative

因為 FP 是實數的**近似值**，且電腦精度有限：

$$(\text{x} + \text{y}) + \text{z} \neq \text{x} + (\text{y} + \text{z})$$

範例：

- $(x + y) + z = 1.0$
- $x + (y + z) = 0.0$

平行程式可能以非預期順序交錯運算 → 結果可能不一致，需要驗證。

### Fallacy：整數的平行執行策略同樣適用於 FP？

不一定。原因：OS scheduler 在不同次執行時可能分配不同數量的 processors → FP 加法順序不同 → 結果不同（但不一定是 bug）。

撰寫含 FP 的平行程式時，需驗證結果的**合理性**，而非要求結果完全一致。

---

## 3.10 Concluding Remarks

- **Bit patterns 沒有固有意義**，意義取決於執行的 instructions
- 電腦數字有**有限的 range 與 precision**
    - 程式設計時必須考慮這些限制
- ISA 支援的算術：
    - Signed / unsigned integers
    - Floating-point（實數的近似值）
    - 運算可能發生 **[[Overflow & Underflow|overflow]]** 與 **[[Overflow & Underflow|underflow]]**

---

## 概念索引

- [[Two's Complement]] — 負數表示法、加減法 overflow 偵測
- [[ALU]] — 1-bit 到 32-bit 硬體架構、乘除法硬體演進
- [[Overflow & Underflow]] — Integer 與 FP 的偵測規則與處理
- [[Floating Point]] — IEEE 754、special values、rounding
- [[Parallelism]] — Subword parallelism、SIMD
- [[ISA]] — RISC-V 乘除法指令（`mul`、`div`）、FP 指令（`fadd`、`flw`）
- [[Instruction Format]] — I-type, S-type, R-type（FP）