---
tags:
  - 計算機組織
---

## tags: [計算機組織, RISC-V, 第二章]

## 📌 本章大綱

1. **[[ISA|Instruction Set & RISC-V Introduction]]**（指令集與 RISC-V 簡介）
2. **Operands of Computer Hardware**（運算元：[[Register|暫存器]]、記憶體、立即數）
3. **Signed vs. Unsigned Numbers**（有號數與無號數：[[Two's Complement|二補數]]）
4. **Representing Instructions in the Computer**（[[Instruction Format|指令格式]]：R, I, S, B, U, J）
5. **Logical Operations**（邏輯運算）
6. **Instructions for Making Decisions**（流程控制：分支與跳轉）
7. **Supporting Procedures**（程序調用：Stack、[[Register|暫存器]]規範）
8. **Addressing Modes**（定址模式總結）

---

### 1. Introduction & Operations

- **[[ISA|Instruction Set（指令集）]]**：電腦硬體所能理解的語言，不同的電腦有不同的指令集，但基本原理相似。
- **RISC-V**：本課程使用的 [[ISA]]，特色是開放（Open-source）且精簡。
- **設計原則 1：Simplicity favors regularity（簡單源於規則）**
    - 算術運算固定有三個運算元（例如 `add a, b, c` 代表 a=b+c）。
    - 規律的 [[Instruction Format|指令格式]] 能讓硬體實作更簡單、效能更高。

---

### 2. Operands of Computer Hardware（運算元）

- **[[Register|Registers（暫存器）]]**：
    
    - RISC-V 有 **32 個 64-bit [[Register|暫存器]]**（x0 至 x31），在 RV64 中每個暫存器稱為一個 **Doubleword**。
    - **x0 永遠為 0**：寫入 x0 的值會被丟棄，讀取永遠得到 0。
    - **設計原則 2：Smaller is faster（小即是快）**：[[Register|暫存器]] 數量不宜過多，以維持存取速度。
- **Memory Operands（記憶體運算元）**：
    
    - 資料存放在記憶體中，運算前須先用 `ld` (Load doubleword) 載入 [[Register|暫存器]]，運算後用 `sd`(Store doubleword) 存回。
    - **Byte Addressable（位元組定址）**：一個 Doubleword (64-bit) 佔 8 位元組，地址間隔為 8。
    - **Endianness（端序）**：RISC-V 採用 **Little-Endian**（低位元組存放在低地址）。
- **Immediate Operands（立即數）**：
    
    - 常數直接放在指令中（如 `addi x22, x22, 4`）。
    - **設計原則 3：Make the common case fast**：使用立即數可避免從記憶體讀取常數。

---

### 3. Signed & Unsigned Numbers（補數運算）

- **Binary Representation**：n bits 可表示 $2^n$ 個數值。
- **[[Two's Complement|Two's Complement（二補數）]]**：

$$x = -x_{n-1}2^{n-1} + x_{n-2}2^{n-2} + \cdots + x_0 2^0$$

- **Sign Bit**：最高位為 0 代表正數，1 代表負數。
- **Negation 捷徑**：取反（Invert）再加 1。
- **Sign Extension**：將 12-bit 立即數擴展到 64-bit 時，重複填補 Sign Bit 以保持數值不變。

---

### 4. Representing Instructions（[[Instruction Format|指令格式]]）

指令在電腦內以 32-bit 二進位碼存放（Machine Code）。

|格式|說明|主要欄位|
|---|---|---|
|**R-type**|[[Register\|暫存器]] 運算|`opcode`, `rd`, `funct3`, `rs1`, `rs2`, `funct7`|
|**I-type**|立即數 / Load|`opcode`, `rd`, `funct3`, `rs1`, `immediate`|
|**S-type**|Store 指令|`opcode`, `funct3`, `rs1`, `rs2`, `imm[11:5]`, `imm[4:0]`|
|**B-type**|條件分支|`opcode`, `funct3`, `rs1`, `rs2`, `imm`|
|**U-type**|長立即數|`opcode`, `rd`, `imm[31:12]`|
|**J-type**|無條件跳轉|`opcode`, `rd`, `imm`|

- **設計原則 4：Good design demands good compromises**：為了維持指令長度固定（32-bit），[[Instruction Format|指令格式]] 必須有所區分但盡可能保持一致。

---

### 5. Logical Operations（邏輯運算）

|指令|名稱|用途|
|---|---|---|
|`slli`|Shift left logical|左移（補 0），相當於乘以 $2^n$|
|`srli`|Shift right logical|右移（補 0），相當於除以 $2^n$|
|`and` / `andi`|AND|位元遮罩（Masking bits）|
|`or` / `ori`|OR|合併位元|
|`xor` / `xori`|XOR|位元反轉（RISC-V 沒有 NOT 指令，通常用 XORI 達成）|

---

### 6. Instructions for Making Decisions（流程控制）

- **Conditional Branches（有條件分支）**：
    
    - `beq rs1, rs2, Label`：相等跳轉。
    - `bne rs1, rs2, Label`：不相等跳轉。
    - `blt` / `bge`：小於跳轉 / 大於等於跳轉。
- **Unconditional Branches（無條件跳轉）**：
    
    - `jal rd, Label`：跳至標籤並將下一條指令地址存入 `rd`。
    - `jalr rd, offset(rs1)`：跳至 [[Register|暫存器]] 指定的地址。

---

### 7. Supporting Procedures（程序支持）

- **[[Register]] Usage 規範**：
    
    - `a0`–`a7` (x10–x17)：傳遞參數與回傳結果。
    - `ra` (x1)：存放返回地址。
- **Stack（堆疊）**：
    
    - 當 [[Register|暫存器]] 不夠用時，將資料暫存至記憶體。
    - **生長方向**：由高位往低位生長，`sp` (Stack Pointer, x2) 遞減。
- **Leaf vs. Non-Leaf**：Non-leaf procedure 必須將返回地址存入 Stack 避免被覆蓋。
    

---

### 8. Addressing Modes（定址模式總結）

1. **Immediate Addressing**：運算元就在指令中（I-type）。
2. **Register Addressing**：運算元在 [[Register|暫存器]] 中（R-type）。
3. **Base (Displacement) Addressing**：地址 = [[Register|暫存器]] 內容 + 偏移量（Load/Store）。
4. **PC-relative Addressing**：地址 = PC + 偏移量（B-type/J-type）。

---

## 📝 重要公式 & 指令整理

|類別|指令範例|功能說明|
|---|---|---|
|**Arithmetic**|`add`, `sub`, `addi`|加、減、立即值加|
|**Data Transfer**|`ld`, `sd`, `lw`, `sw`|載入/儲存 Doubleword (64b), Word (32b)|
|**Logical**|`sll`, `srl`, `and`, `or`, `xor`|位元邏輯運算|
|**Branch**|`beq`, `bne`, `blt`, `bge`|有條件跳轉|
|**Jump**|`jal`, `jalr`|呼叫函式、回傳、間接跳轉|

---

## 🔗 相關概念

- [[ISA]] — 硬體與軟體間的合約，本章詳述其語法。
- [[Instruction Format]] — 本章核心，R / I / S / B / U / J 六種格式。
- [[Register]] — RISC-V 共 32 個，命名規則與使用慣例。
- [[Two's Complement]] — Sign Extension 的基礎。
- [[Memory Hierarchy]] — Load/Store 是存取記憶體的唯一方式，與快取息息相關。

---

## 🧪 練習題

**Q1.** 將十進位 `-5` 以 8-bit [[Two's Complement|二補數]] 表示。

> 提示：先寫出 +5 的二進位，再取反加 1。

**Q2.** 以下 RISC-V 程式碼執行後，x5 的值為何？

```
addi x5, x0, 10
addi x6, x0, 3
add  x5, x5, x6
slli x5, x5, 2
```

**Q3.** R-type 與 I-type 最大的差異是什麼？為什麼 `addi` 不能用 R-type？

**Q4.** 為什麼 RISC-V 的 Stack 是向低位址生長？`sp` 在 push 時是加還是減？

**Q5.** `ld x10, 8(x5)` 使用了哪種定址模式？地址如何計算？

**Q6.** RISC-V 沒有 NOT 指令，如何用 `xori` 達成對 x5 全位元取反？




> [!note]- 參考答案
> 
> **A1.** +5 = `0000 0101` → 取反 = `1111 1010` → 加 1 = **`1111 1011`**
> 
> **A2.** x5 = 10 + 3 = 13 → slli 2 = 13 × 4 = **52**
> 
> **A3.** R-type 的運算元全為暫存器；I-type 含 12-bit 立即數。`addi` 需要加上常數，必須用 I-type。
>  
> **A4.** 程式碼從低位址往高位址，Stack 從高往低以避免衝突。`sp` 在 push 時**減少**（`addi sp, sp, -8`）。
> 
> **A5.** **Base (Displacement) Addressing**。地址 = x5 的值 + 8
> 
> **A6.** `xori x5, x5, -1`（-1 在二補數為全 1，XOR 全 1 即取反）
 
 
