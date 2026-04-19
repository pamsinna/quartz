# Pipelining

#計算機組織 #概念

一種讓多條指令**重疊執行**的實作技術，類似工廠流水線（assembly line）。利用 **instruction-level parallelism（ILP）** 提升 throughput。

---

## 基本概念

- **Throughput 提升**，但單條指令的 latency 不變
- 所有 pipeline stage 共用同一個 clock cycle，長度由**最慢的 stage** 決定
- 理想加速比 = pipeline 的 stage 數（在 stage 均衡時）

Speedup 公式（stage 均衡時）：

$$\text{Time between instructions}_\text{pipelined} = \frac{\text{Time between instructions}_\text{nonpipelined}}{\text{Number of stages}}$$

---

## RISC-V 五階段 Pipeline

|Stage|全名|動作|
|---|---|---|
|**IF**|Instruction Fetch|從 instruction memory 讀取指令|
|**ID**|Instruction Decode / Register Read|解碼指令、讀取 registers|
|**EX**|Execute|ALU 執行運算或計算 address|
|**MEM**|Memory Access|讀寫 data memory（load/store）|
|**WB**|Write Back|將結果寫回 register file|

---

## 效能比較（Fig. 4.28, 4.29）

假設各元件延遲：Memory = 200ps、ALU = 200ps、Register file = 100ps

|指令|Single-cycle 時間|Pipeline clock cycle|
|---|---|---|
|lw（最慢）|800ps|200ps（最慢 stage）|
|beq（最快）|500ps|200ps（同上）|

Single-cycle 必須以最慢指令（800ps）為 clock，三條指令需 2400ps；Pipeline 三條指令只需 600ps（**4 倍加速**）。

---

## RISC-V ISA 對 Pipelining 的支援

1. **Fixed-length instructions**（32 bits）→ 方便一個 cycle 完成 fetch 與 decode
2. **Regular instruction formats**：rs1、rs2、rd 永遠在相同位置 → 可同步 decode 與 register read
3. **Load/Store addressing**：memory operand 只出現在 load/store → address 在 EX 計算，MEM 才存取

（對比 x86：1~15 byte 變長指令，在實作上會先轉成類似 RISC 的 micro-code）

---

## Pipeline Registers

在每兩個 stage 之間加入 **pipeline register**，儲存需要傳遞到後續 stage 的資料與控制信號：

```
IF/ID → ID/EX → EX/MEM → MEM/WB
```

- IF/ID：96 bits（32-bit 指令 + 64-bit PC）
- 每個 register 的寬度需能容納該 stage 所有輸出

---

## Pipeline Hazards

詳見 [[Pipeline Hazards]]

---

## 相關

- [[Pipeline Hazards]] — Structure / Data / Control hazard 與解法
- [[ALU]] — EX stage 的核心元件
- [[Performance]] — Throughput vs. Latency 的取捨
- [[Parallelism]] — ILP 的更進階實作（multiple issue、dynamic scheduling）
- [[第四章 The Processor]] — 完整 datapath 與 pipelined control 設計