---
tags:
  - 概念
  - 計算機組織
---

# Register（暫存器）

## 定義

暫存器是 CPU **內部最快的儲存單元**，是算術與邏輯運算的直接操作對象。

## RISC-V 暫存器規格

- 共 **32 個**，編號 x0–x31
- 每個暫存器寬度：**64-bit**（RV64，稱為 Doubleword）
- **x0 永遠為 0**：寫入被丟棄，讀取永遠得 0

## 暫存器使用慣例（ABI 名稱）

|暫存器|ABI 名稱|用途|
|---|---|---|
|x0|zero|常數 0|
|x1|ra|Return Address|
|x2|sp|Stack Pointer|
|x5–x7, x28–x31|t0–t6|Temporaries（caller-saved）|
|x8–x9, x18–x27|s0–s11|Saved registers（callee-saved）|
|x10–x17|a0–a7|函式參數 / 回傳值|

## 設計原則

**Smaller is faster**：暫存器數量不宜過多，32 個是 RISC-V 的設計折衷點。

## 相關概念

- [[ISA]] — ISA 定義暫存器數量與規格
- [[Instruction Format]] — rd, rs1, rs2 欄位對應暫存器編號
- [[Memory Hierarchy]] — 暫存器是記憶體層次的最頂層
- [[第二章 Instructions Language of the Computer]]