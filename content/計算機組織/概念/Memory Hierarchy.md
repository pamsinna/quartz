---
tags:
  - 概念
  - 計算機組織
---

# Memory Hierarchy（記憶體層次結構）

## 核心概念

沒有任何一種記憶體能同時做到「快速、大容量、便宜」，因此電腦使用**分層結構**：越靠近 CPU 越快越小越貴，越遠越慢越大越便宜。

## 層次結構（由快到慢）

```
CPU Registers（暫存器）      ← 1 cycle
    ↓
Cache L1 (SRAM, ~KB)        ← 數個 cycles
    ↓
Cache L2/L3 (SRAM, ~MB)     ← 數十 cycles
    ↓
Primary memory - DRAM (~GB) ← 數百 cycles
    ↓
Secondary storage - SSD/HDD ← 數百萬 cycles
```

## 揮發性分類

|類型|斷電後|範例|
|---|---|---|
|**Volatile**|資料消失|SRAM（Cache）、DRAM（主記憶體）|
|**Non-volatile**|資料保留|Flash（SSD）、磁碟、光碟|

## 關鍵原理：局部性（Locality）

- **Temporal locality**：最近存取的資料很快會再被存取
- **Spatial locality**：存取某位址後，鄰近位址也很快會被存取

這使得 Cache 命中率高，彌補速度差距。

## RISC-V 的 Load/Store 架構

只有 `ld`/`sd` 等指令能存取記憶體，其他運算只能操作 [[Register|暫存器]]。

## 相關概念

- [[Register]] — 記憶體層次的最頂層
- [[Performance]] — 記憶體存取延遲直接影響實際 CPI
- [[第一章 Computer Abstractions and Technology]]
- [[第二章 Instructions Language of the Computer]]