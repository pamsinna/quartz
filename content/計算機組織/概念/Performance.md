---
tags:
  - 概念
  - 計算機組織
---

# Performance（效能）

## 核心定義

$$\text{Performance} = \frac{1}{\text{Execution Time}}$$

「X 比 Y 快 n 倍」：

$$n = \frac{\text{Execution Time}_Y}{\text{Execution Time}_X}$$

## Execution Time 的兩種定義

|類型|內容|
|---|---|
|**Elapsed time（Wall clock）**|含 CPU、I/O、OS、idle 的總時間|
|**CPU time**|僅計算 CPU 執行時間（常用於效能比較）|

## CPU Time 公式

$$\text{CPU Time} = IC \times CPI \times \text{Clock Cycle Time} = \frac{IC \times CPI}{\text{Clock Rate}}$$

|符號|意義|決定因素|
|---|---|---|
|**IC**（Instruction Count）|程式執行的指令數|演算法、語言、編譯器、[[ISA]]|
|**CPI**（Cycles Per Instruction）|平均每條指令的週期數|CPU 硬體設計|
|**Clock Rate**|每秒幾個 clock cycle（Hz）|硬體實作|

## 混合 CPI 計算

$$\text{CPI} = \sum_{i=1}^{n} \left(\text{CPI}_i \times \frac{\text{IC}_i}{\text{IC}}\right)$$

## Power Wall 與效能瓶頸

$$\text{Power} = C \times V^2 \times f$$

頻率無法無限提升 → 轉向 [[Parallelism|多核心]] 設計

## 相關概念

- [[ISA]] — 影響 IC 與 CPI
- [[Parallelism]] — 多核心提升 throughput
- [[Memory Hierarchy]] — 記憶體存取影響實際 CPI
- [[第一章 Computer Abstractions and Technology]]