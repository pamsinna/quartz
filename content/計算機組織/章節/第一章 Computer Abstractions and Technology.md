---
tags:
  - 計算機組織
---

## tags: [計算機組織, 第一章]

## 📌 本章大綱

- Introduction & Classes of Computers
- Seven Great Ideas
- Below Your Program（程式底層）
- Under the Covers（硬體內部）
- Technologies for Building Processors/Memory
- [[Performance]]
- The Power Wall
- Switch to Multiprocessors
- Benchmarking

---

## 1. Introduction

### 電腦革命的驅動力

- 進步由 **domain-specific accelerators（領域專用加速器）** 推動
- 讓許多新應用成為可能：汽車電腦、手機、人類基因組計畫、WWW、搜尋引擎

### 電腦分類

|類型|特點|
|---|---|
|**Personal Computers**|通用用途，cost/performance tradeoff|
|**Embedded Computers**|隱藏在系統中，嚴格的 power/cost/performance 限制|
|**Server Computers**|網路為基礎，高容量、高可靠性|
|**Supercomputers**|伺服器的一種，高階科學計算，市占率小|

### PostPC 時代

- **PMD（Personal Mobile Device）**：電池供電、連網、數百美元（智慧型手機、平板）
- **Cloud Computing**：Warehouse Scale Computers (WSC)、SaaS、Amazon/Google 等提供

### 本課程學習目標

- 程式如何翻譯成機器語言，硬體如何執行
- 硬體/軟體介面 → [[ISA]]
- 決定程式效能的因素 → [[Performance]]
- 硬體設計者如何提升效能
- 平行處理概念 → [[Parallelism]]

---

## 2. Seven Great Ideas

|Idea|說明|
|---|---|
|**Use abstraction to simplify design**|用抽象化隱藏低層細節|
|**Make the common case fast**|優化最常見的情況|
|**Performance via parallelism**|透過並行提升效能|
|**Performance via pipelining**|透過流水線提升效能（一種平行的模式）|
|**Performance via prediction**|透過預測提升效能|
|**Hierarchy of memories**|記憶體層次結構，速度 vs. 大小 vs. 成本|
|**Dependability via redundancy**|透過冗餘提升可靠性|

---

## 3. Below Your Program（程式底層）

### 軟硬體層次

```
Applications software（高階語言）
    ↓ Compiler
System software
    - OS：管理 I/O、記憶體、排程
    ↓ Assembler
Hardware：Processor, Memory, I/O controllers
```

### 程式碼層次

|層次|說明|
|---|---|
|**High-level language**|接近問題領域，高生產力與可移植性|
|**Assembly language**|指令的符號表示|
|**Machine language**|二進位 bits，硬體直接執行|

---

## 4. Under the Covers（硬體內部）

### 電腦五大古典元件

1. **Input**
2. **Output**
3. **Memory**
4. **Datapath** ← 與 Control 合稱 Processor
5. **Control**

> Control 決定 Datapath、Memory、Input、Output 的操作

### I/O 裝置

- **Input（PostPC）**：觸控螢幕（電容式 > 電阻式，支援多點觸控）
- **Output**：LCD 螢幕（pixels），內容來自 frame buffer memory

### 記憶體 → 詳見 [[Memory Hierarchy]]

|類型|特性|
|---|---|
|**Volatile（揮發性）**|斷電即失（DRAM/SRAM）|
|**Non-volatile（非揮發性）**|斷電不失（Flash, 磁碟, 光碟）|

**記憶體層次（由快到慢、由小到大）：**

```
Cache memory (SRAM)
    ↓
Primary memory (DRAM)
    ↓
Secondary memory (Flash memory / Magnetic disk)
```

### Abstractions（抽象化）→ 詳見 [[ISA]]

- **[[ISA]]（Instruction Set Architecture）**：硬體/軟體介面，例如 RISC-V 指令集
- **ABI（Application Binary Interface）**：ISA + 系統軟體介面，跨電腦二進位可移植性
- 抽象化讓不同實作（cost/performance 各異）可執行相同軟體

---

## 5. Technologies for Building Processors and Memory

### 半導體製程

- 矽半導體 → 加入材料改變特性：導體、絕緣體、開關（電晶體）
- **Yield（良率）** = 晶圓中可用 die 的比例

### IC 成本公式

$$\text{Cost per die} = \frac{\text{Cost per wafer}}{\text{Dies per wafer} \times \text{Yield}}$$

$$\text{Dies per wafer} \approx \frac{\text{Wafer area}}{\text{Die area}}$$

$$\text{Yield} = \frac{1}{(1 + \text{Defects per area} \times \text{Die area})^N}$$

> Die 面積越大 → Yield 越低 → Cost 非線性增加

---

## 6. Performance → [[Performance]]

### 效能指標

|指標|定義|
|---|---|
|**Response time（Execution time）**|完成一個任務所需總時間|
|**Throughput（Bandwidth）**|單位時間完成的工作量|

$$\text{Performance} = \frac{1}{\text{Execution Time}}$$

「X 比 Y 快 n 倍」：

$$n = \frac{\text{Performance}_X}{\text{Performance}_Y} = \frac{\text{Execution Time}_Y}{\text{Execution Time}_X}$$

### Execution Time 的兩種定義

|類型|定義|
|---|---|
|**Elapsed time（Wall clock time）**|包含所有：CPU、I/O、OS overhead、idle time|
|**CPU time**|只計算 CPU 處理時間（又分 user CPU time / system CPU time）|

### CPU Time 公式

$$\text{CPU Time} = \text{CPU Clock Cycles} \times \text{Clock Cycle Time} = \frac{\text{CPU Clock Cycles}}{\text{Clock Rate}}$$

$$\text{CPU Time} = \text{Instruction Count} \times \text{CPI} \times \text{Clock Cycle Time} = \frac{IC \times CPI}{\text{Clock Rate}}$$

### CPI（Cycles Per Instruction）

$$\text{Clock Cycles} = \sum_{i=1}^{n} (\text{CPI}_i \times \text{IC}_i)$$

$$\text{CPI} = \frac{\text{Clock Cycles}}{\text{Instruction Count}} = \sum_{i=1}^{n} \left(\text{CPI}_i \times \frac{\text{IC}_i}{\text{IC}}\right)$$

### 效能影響因素總覽

|因素|影響 IC|影響 CPI|影響 Clock Rate|
|---|---|---|---|
|Algorithm|✓|可能||
|Programming language|✓|✓||
|Compiler|✓|✓||
|ISA|✓|✓|✓|

> **Time 是唯一完整可靠的效能衡量指標**

---

## 7. The Power Wall

### CMOS 功耗公式

$$\text{Power} = \text{Capacitive load} \times \text{Voltage}^2 \times \text{Frequency}$$

- 電壓從 5V 降至 1V → 功耗降低 25 倍
- **Dynamic energy**：電晶體切換 0↔1 時消耗
- **Static energy（leakage）**：電晶體關閉時仍有漏電流，伺服器中佔約 40%
- Clock rate 提升 → 功耗增加 → 散熱問題 → 轉向多核心設計

---

## 8. Switch from Uniprocessors to Multiprocessors → [[Parallelism]]

|項目|轉換前|轉換後|
|---|---|---|
|平行方式|Instruction-level parallelism（ILP），對程式透明|需明確撰寫平行程式|
|難度|低（硬體自動處理）|高（需考慮 load balancing、同步等）|

---

## 9. Benchmarking

$$\text{SPECratio} = \frac{\text{Reference Time}}{\text{Measured Time}}$$

$$\text{SPEC score} = \sqrt[n]{\prod_{i=1}^{n} \text{Execution time ratio}_i}$$

$$\text{Overall ssj\_ops per Watt} = \frac{\sum_{i=0}^{10} \text{ssj\_ops}_i}{\sum_{i=0}^{10} \text{power}_i}$$

---

## 📝 重要公式整理

|公式|說明|
|---|---|
|$\text{Performance} = 1/\text{Execution Time}$|效能定義|
|$\text{CPU Time} = IC \times CPI \times \text{Clock Cycle Time}$|CPU 時間|
|$\text{CPU Time} = IC \times CPI / \text{Clock Rate}$|CPU 時間（另一種寫法）|
|$\text{Power} = C \times V^2 \times f$|CMOS 功耗|
|$\text{Cost per die} = \text{Cost per wafer}/(\text{Dies per wafer} \times \text{Yield})$|IC 成本|

---

## 🔗 相關概念

- [[ISA]] — 硬體/軟體介面，RISC-V 指令集
- [[Performance]] — 效能指標、CPI、CPU Time
- [[Memory Hierarchy]] — Cache/DRAM/Flash 層次結構
- [[Parallelism]] — 多核心、平行程式設計
- [[Register]] — 暫存器（第二章詳述）
- [[Instruction Format]] — 指令格式（第二章詳述）
- [[Two's Complement]] — 補數（第二章詳述）

---

## 🧪 練習題

**Q1.** 若 CPU A 的 Clock Rate 為 2 GHz、CPI 為 4；CPU B 的 Clock Rate 為 3 GHz、CPI 為 3，兩者執行相同程式（相同 IC），哪一個較快？快多少？

> 提示：CPU Time = IC × CPI / Clock Rate

**Q2.** 某程式有 40% 的指令為整數運算（CPI=1）、60% 為浮點運算（CPI=4），整體 CPI 為何？

**Q3.** 解釋 Response time 與 Throughput 的差異，並各舉一個優化範例。

**Q4.** 為什麼單純提高 Clock Rate 無法無限提升效能？（提示：Power Wall）

**Q5.** Seven Great Ideas 中哪三個與效能最直接相關？各用一句話說明。

> [!note]- 參考答案 
> A1. CPU A time = IC × 4 / 2G = 2IC ns；CPU B time = IC × 3 / 3G = 1IC ns → CPU B 快 2 倍
> 
> A2. CPI = 0.4 × 1 + 0.6 × 4 = **2.8**
> 
> A3. Response time：完成單一任務的時間，優化方式如加快 CPU。Throughput：單位時間完成的工作量，優化方式如增加多核心或流水線。
> 
> A4. Power = C × V² × f，頻率提高導致功耗線性增加、散熱極限，約 2004 年後改為多核心路線。
> 
> A5. Make the common case fast / Performance via parallelism / Performance via pipelining