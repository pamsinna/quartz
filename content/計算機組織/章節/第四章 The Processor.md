

#計算機組織 #章節

---

## Chapter Goals

- 理解 processor 的 **datapath** 與 **control unit** 設計
- 比較 **single-cycle** 與 **pipelined** 兩種實作
- 理解 [[Pipeline Hazards]] 及其解決方案
- 了解 **ILP**（Instruction-Level Parallelism）的提升方式

本章以下列 RISC-V 指令子集為範例：

- Memory reference：`lw`、`sw`
- Arithmetic/logical：`add`、`sub`、`and`、`or`
- Control transfer：`beq`

---

## 4.1 Introduction

效能由三個因素決定（詳見第一章）：

1. **Clock cycle time** — 由 processor 實作決定
2. **CPI** — 由 processor 實作決定
3. **Instruction count** — 由 [[ISA]] 與 compiler 決定

### Instruction Execution 概覽

每條指令前兩步相同：

1. 將 **PC** 送到 instruction memory，fetch 指令
2. 根據指令欄位讀一或兩個 **registers**

之後依指令類別分歧：

|指令類別|ALU 用途|後續動作|
|---|---|---|
|Memory-reference|Address calculation|讀（load）或寫（store）memory|
|Arithmetic-logical|執行運算|ALU result 寫回 register|
|Conditional branch|Equality test（subtraction）|更新 PC → target 或 PC+4|

RISC-V 指令集的簡單規律性（simplicity and regularity）大幅簡化了 implementation。

### Instruction Execution Flow（Fig. 4.1, 4.2）

```
PC → Instruction Memory → Register File → ALU → (Data Memory) → Register File
```

需要加入的元件：

- **三個 Multiplexors**（data selector）：在多個來源中選一個送往目的地
    - Top mux：選擇下一個 PC（PC+4 或 branch target）
    - Middle mux：選擇寫回 register 的資料（ALU result 或 memory data）
    - Bottom mux：選擇 ALU 第二輸入（register 或 sign-extended immediate）
- **Control unit**：根據指令的 opcode 欄位，決定所有 control line 的設定

---

## 4.2 Logic Design Conventions

### Combinational vs. Sequential Elements

|類型|說明|範例|
|---|---|---|
|**Combinational element**|輸出只取決於當前輸入；相同輸入永遠產生相同輸出|AND gate、[[ALU]]、Multiplexer、Adder|
|**State element**|含有內部儲存（nonvolatile）；輸出與輸入和當前狀態都有關|Register、Memory|

State element 至少需要兩個輸入：

- 要寫入的 **data value**
- 決定「何時」寫入的 **clock**

### Edge-Triggered Clocking

- 採用 **edge-triggered clocking**：所有 state changes 發生在 clock edge
- 本章假設所有 state element 皆為 **positive edge-triggered**（上升緣觸發）
- 輸入是前一個 clock cycle 寫入的值；輸出可在下一個 cycle 使用
- 資訊以二進位編碼：Low = 0、High = 1；多 bit 用 multi-wire **bus**

### Register with Write Enable

- 只有在 clock edge 且 `Write == 1` 時才更新
- 用於需要選擇性寫入的場合（如 register file）

---

## 4.3 Building a Datapath

**Datapath element**：processor 內用來操作或儲存資料的單元，包含 instruction memory、data memory、register file、[[ALU]]、adders。以下逐步建構 RISC-V datapath。

### Instruction Fetch（Fig. 4.5, 4.6）

需要的元件：

|元件|說明|
|---|---|
|**Instruction memory**|根據地址輸出指令，唯讀，視為 combinational logic|
|**Program Counter（PC）**|32-bit register，儲存當前指令地址，每個 clock cycle 結束時寫入（不需 write control）|
|**Adder**|計算 PC+4，產生下一條指令地址|

### R-Format ALU Operations（Fig. 4.7）

**Register file**：儲存 32 個 general-purpose registers

- 兩個 read port + 一個 write port（multiported）
- Register number 輸入為 **5 bits**（$2^5 = 32$）
- 讀取永遠輸出（combinational）；寫入需 `RegWrite` control signal（edge-triggered）
- 合法：同一 clock cycle 內讀寫同一個 register（讀到的是前一 cycle 寫入的值）
- ALU control signal 為 **4 bits**；**Zero output** 用於 conditional branch

### Load/Store（Fig. 4.8）

額外需要：

|元件|說明|
|---|---|
|**ImmGen**（Immediate Generation Unit）|將指令中的 12-bit offset sign-extend 為 32 bits|
|**Data memory**|有獨立的 `MemRead` / `MemWrite` control；讀取 invalid address 可能造成問題（見第五章）|

Load/Store 流程：

1. 讀 register operand（base register）
2. 用 ALU 計算 address（base + sign-extended offset）
3. Load：從 memory 讀資料 → 寫回 register
4. Store：將 register 值寫入 memory

### Branch Instruction（Fig. 4.9）

步驟：

1. 讀兩個 register operands
2. 用 ALU 做**減法**，檢查 **Zero output**（Zero=1 代表兩者相等）
3. 計算 branch target：sign-extend 12-bit offset → shift left 1（halfword displacement）→ add to PC+4
4. 根據 Zero output 選擇下一個 PC

> 注意：執行 branch 指令時，IF stage 已將 PC 更新為 PC+4

### Compose the Elements（Fig. 4.10, 4.11）

讓 datapath 在一個 clock cycle 完成一條指令，需要：

- **Separate instruction/data memories**（每個元件一次只能做一件事）
- **Multiplexors**（不同指令使用不同資料來源）

加入的三個 mux：

|Mux|選擇|控制信號|
|---|---|---|
|ALUSrc|Register 值 vs. Sign-extended immediate|`ALUSrc`|
|MemtoReg|ALU result vs. Memory data|`MemtoReg`|
|PCSrc|PC+4 vs. Branch target|`Branch AND Zero`|

---

## 4.4 A Simple Implementation Scheme（Single-Cycle）

### ALU Control（Fig. 4.12, 4.13）

ALU control 採**兩層解碼**（減少 latency → 有助於縮短 clock cycle time）：

**第一層：ALUOp**（2-bit，由 main control unit 產生）

|ALUOp|意義|適用指令|
|---|---|---|
|`00`|add|load / store|
|`01`|subtract & test zero|beq|
|`10`|由 funct7 + funct3 決定|R-format|

**第二層：4-bit ALU control input**（最終控制 ALU 操作）

|指令|funct7 bit 30|funct3|ALU control|
|---|---|---|---|
|`add` / `lw` / `sw`|X|X|0010（add）|
|`sub` / `beq`|1 / X|000 / X|0110（subtract）|
|`and`|0|111|0000（AND）|
|`or`|0|110|0001（OR）|

**Don't-care term（X）**：輸出不依賴於該 input bit，可用於化簡邏輯。

### Instruction Formats 回顧（Fig. 4.14）

|格式|指令|opcode|特點|
|---|---|---|---|
|R-type|add, sub, and, or|`0110011`|rs1, rs2, rd；funct3+funct7 決定操作|
|I-type|lw|`0000011`|rs1, rd；12-bit immediate（offset）|
|S-type|sw|`0100011`|rs1, rs2；12-bit immediate 分兩段|
|SB-type|beq|`1100011`|rs1, rs2；12-bit immediate（branch offset）|

所有格式共同點（RISC-V 簡化硬體設計的關鍵）：

- opcode 永遠在 bits **6:0**
- rs1 永遠在 bits **19:15**
- rs2 永遠在 bits **24:20**
- rd 永遠在 bits **11:7**

> RISC-V 與 MIPS 比較：MIPS 的 load 指令目的 register 是 rt，與 R-format 的 rd 位置不同，需要額外一個 mux。RISC-V 的設計統一了 rd 位置，簡化了硬體。

### Main Control Unit（Fig. 4.20, 4.21, 4.22）

Control unit 輸入為 **7-bit opcode**，輸出：

|控制信號|位元數|功能|
|---|---|---|
|`ALUSrc`|1|ALU 第二輸入來源（0=register, 1=immediate）|
|`MemtoReg`|1|寫回 register 的資料來源（0=ALU, 1=memory）|
|`RegWrite`|1|是否寫入 register file|
|`MemRead`|1|是否讀取 data memory|
|`MemWrite`|1|是否寫入 data memory|
|`Branch`|1|是否為 branch 指令|
|`ALUOp`|2|ALU 操作類型（送往 ALU control）|

`PCSrc` 為 **derived signal**：`Branch AND Zero`（不直接由 control unit 輸出）

各指令的 control signal 設定：

|指令|ALUSrc|MemtoReg|RegWrite|MemRead|MemWrite|Branch|ALUOp|
|---|---|---|---|---|---|---|---|
|R-format|0|0|1|0|0|0|10|
|lw|1|1|1|1|0|0|00|
|sw|1|X|0|0|1|0|00|
|beq|0|X|0|0|0|1|01|

> `MemtoReg = X`：當 `RegWrite = 0` 時，register file 不寫入，`MemtoReg` 的值無關緊要。

### Execution Flow（各指令類別）

#### R-Format（Fig. 4.23）— 4 steps

1. Fetch 指令，PC += 4
2. 讀兩個 source registers（x2, x3）；main control unit 計算 control signals
3. ALU 根據 funct 欄位執行運算
4. ALU result 寫回 destination register（x1）

#### I-Format / Load（Fig. 4.24）— 5 steps

1. Fetch 指令，PC += 4
2. 讀 base register（x2）；計算 control signals
3. ALU 計算 address（x2 + sign-extended offset）
4. 以 ALU result 為地址讀取 data memory
5. Memory data 寫回 destination register（x1）

> Store 與 Load 類似，差別在 memory 為寫入（不寫回 register）。

#### Branch（Fig. 4.25）— 4 steps

1. Fetch 指令，PC += 4
2. 讀兩個 register（x1, x2）；計算 control signals
3. ALU 做減法，產生 Zero output；同時計算 branch target（PC+4 + sign-extended offset << 1）
4. Zero AND Branch → 選擇 branch target 或 PC+4 寫入 PC

### Control Unit Truth Table（Fig. 4.26）

Control function 完全由 opcode bits（I[0:6]）決定：

|Instruction|I[6:0]|
|---|---|
|R-format|`0110011`|
|lw|`0000011`|
|sw|`0100011`|
|beq|`1100011`|

---

## 4.5 Performance Issue of Single-Cycle Design

**問題**：clock cycle 由最慢的指令決定（critical path = load instruction）

Load 的 critical path： $$\text{Instruction Memory} \to \text{Register File} \to \text{ALU} \to \text{Data Memory} \to \text{Register File}$$

- 每條指令都必須使用 800ps 的 clock cycle（即使只需 200ps）
- 違反設計原則：**Making the common case fast**

**解決方案**：[[Pipelining]]

---

## 概念索引

- [[ISA]] — RISC-V 指令格式、opcode、[[Instruction Format]]
- [[ALU]] — ALU control 兩層解碼、combinational element
- [[Performance]] — single-cycle critical path、clock cycle time
- [[Pipelining]] — 解決 single-cycle 效能問題
- [[Pipeline Hazards]] — 第四章後半主題
- [[Register]] — Register file 設計、multiported、write enable