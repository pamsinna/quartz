# Register

#計算機組織 #概念

processor 內部用來暫存資料的**高速儲存單元**，屬於 state element（sequential logic）。是算術與邏輯運算的直接操作對象，也是 [[Memory Hierarchy]] 的最頂層。

---

## General-Purpose Registers

RISC-V 有 **32 個** general-purpose registers（`x0`–`x31`），每個 32-bit（RV32）或 64-bit（RV64，稱為 Doubleword）。

特殊規則：

- `x0`（zero register）：hardwired 為 0，寫入被丟棄，讀取永遠得 0
- FP registers `f0` 不是 hardwired 為 0（與 `x0` 不同）

### 暫存器使用慣例（ABI 名稱）

|暫存器|ABI 名稱|用途|
|---|---|---|
|x0|zero|常數 0|
|x1|ra|Return Address|
|x2|sp|Stack Pointer|
|x5–x7, x28–x31|t0–t6|Temporaries（caller-saved）|
|x8–x9, x18–x27|s0–s11|Saved registers（callee-saved）|
|x10–x17|a0–a7|函式參數 / 回傳值|

### 設計原則

**Smaller is faster**：暫存器數量不宜過多，32 個是 RISC-V 的設計折衷點。

---

## Register File

儲存所有 32 個 register 的結構，特性：

|特性|說明|
|---|---|
|**Multiported**|2 個 read port + 1 個 write port（可同時讀兩個 register）|
|**Register number**|5-bit address（$2^5 = 32$）|
|**Read**|Combinational（永遠輸出，不需 control signal）|
|**Write**|Edge-triggered，需 `RegWrite` control signal|
|**同 cycle 讀寫**|合法：讀到的是前一 cycle 寫入的值|

`RegWrite`：只有 load 與 arithmetic-logical 指令才會 assert。

---

## Pipeline Registers

在 [[Pipelining]] 中，各 stage 之間的 pipeline registers 也是一種 register：

|Pipeline Register|寬度|儲存內容|
|---|---|---|
|IF/ID|96 bits|32-bit 指令 + 64-bit PC|
|ID/EX|256 bits|control signals + register values + immediate + PC|
|EX/MEM|193 bits|control signals + ALU result + register values|
|MEM/WB|128 bits|control signals + memory/ALU data + register number|

---

## FP Registers

RISC-V 另有 32 個 FP registers（`f0`–`f31`），詳見 [[Floating Point]]。

---

## 相關

- [[ISA]] — register 使用慣例（calling convention）
- [[Instruction Format]] — rd, rs1, rs2 欄位對應暫存器編號
- [[Two's Complement]] — register 內整數的表示方式
- [[Pipelining]] — pipeline register 的角色
- [[Memory Hierarchy]] — 暫存器是記憶體層次的最頂層
- [[第二章 Instructions Language of the Computer]] — ABI 與暫存器慣例
- [[第四章 The Processor]] — register file 在 datapath 中的位置