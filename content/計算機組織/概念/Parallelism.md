
---
tags: [概念, 計算機組織]

# Parallelism（平行處理）


---

## Subword Parallelism（SIMD）

來源：§3.6 Parallelism and Computer Arithmetic

多媒體與音訊應用常需對**向量資料**執行相同操作。透過**分割寬 adder**，單一硬體可同時平行處理多個窄運算元。

### 128-bit Adder 分割範例

|分割方式|平行處理單元數|
|---|---|
|8-bit|16 個運算元|
|16-bit|8 個運算元|
|32-bit|4 個運算元|
|64-bit|2 個運算元|

成本小，但 speedup 可觀。

### 名稱對照

- Subword Parallelism
- Data-level Parallelism
- Vector Parallelism
- **SIMD**（Single Instruction, Multiple Data）

> SIMD 詳細內容見 §6.6；§3.7–3.8 可提前閱讀。

---

## FP 的平行執行注意事項

來源：§3.9 Fallacies and Pitfalls

- FP addition **不滿足結合律**，平行程式的運算順序不固定
- OS scheduler 可能在不同次執行時分配不同數量的 processors → FP 加法順序改變 → 結果不同
- 撰寫含 FP 的平行程式：驗證結果的**合理性**，而非要求完全一致


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
- [[Floating Point]] — FP 平行的精度問題
- [[ALU]] — Subword parallelism 的硬體基礎
- [[第三章 Arithmetic for Computers]] — §3.6 完整說明