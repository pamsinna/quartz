---
tags:
  - 概念
  - 計算機組織
---

# Parallelism（平行處理）

## 定義

平行處理是讓多個運算**同時進行**以提升效能，是突破 Power Wall 後的主要效能提升手段。

## 兩種平行層次

|層次|說明|對程式設計師可見？|
|---|---|---|
|**ILP（Instruction-Level Parallelism）**|硬體自動重疊執行多條指令（Pipelining, Out-of-order）|否，硬體透明|
|**Thread/Process Level**|多核心同時執行不同執行緒|**是**，需明確撰寫|

## 為何從單核轉向多核？

$$\text{Power} = C \times V^2 \times f$$

頻率持續提升 → 散熱無法負荷（Power Wall）→ 約 2004 年後改為增加核心數。

## Amdahl's Law

程式中不可平行化的部分限制了加速上限：

$$\text{Speedup} = \frac{1}{(1-p) + \dfrac{p}{n}}$$

其中 $p$ 為可平行部分比例，$n$ 為核心數。

## 平行程式設計挑戰

1. **Load balancing**：讓各核心工作量均衡
2. **Synchronization**：共享資料的存取順序控制
3. **Communication overhead**：核心間資料傳輸成本

## 相關概念

- [[Performance]] — 平行是提升 throughput 的主要手段
- [[Memory Hierarchy]] — 多核心共享 Cache，造成一致性問題
- [[第一章 Computer Abstractions and Technology]]