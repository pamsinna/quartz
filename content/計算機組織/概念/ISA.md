---
tags:
  - 概念
  - 計算機組織
---

# ISA（Instruction Set Architecture）

## 定義

ISA 是**硬體與軟體之間的介面合約**，定義了程式設計師（或編譯器）看得到的機器行為：指令集、暫存器、記憶體定址方式、資料類型等。

不同的 ISA 實作可以有不同的效能與成本，但對上層軟體呈現相同的行為。

## 本課程使用的 ISA：RISC-V

- **開放標準**（Open-source），不需授權費
- **精簡指令集（RISC）**：指令數量少、格式規則、每條指令 32-bit 固定長度
- 對比：x86 為 CISC，指令長度可變、格式複雜

## ISA 的組成要素

|要素|RISC-V 範例|
|---|---|
|**Registers**|32 個 64-bit 暫存器（x0–x31）|
|**Instruction formats**|R / I / S / B / U / J 六種格式|
|**Memory model**|Byte-addressable，Little-Endian|
|**Addressing modes**|Immediate / Register / Base / PC-relative|
|**Data types**|Byte, Halfword, Word, Doubleword|

## ISA vs ABI

- **ISA**：硬體/軟體介面（指令層級）
- **ABI（Application Binary Interface）**：ISA + OS 呼叫慣例，讓二進位程式可跨電腦執行

## 相關概念

- [[Instruction Format]] — ISA 規定指令如何編碼
- [[Register]] — ISA 定義暫存器數量與用途
- [[Performance]] — ISA 的選擇影響 IC 與 CPI
- [[第一章 Computer Abstractions and Technology]]
- [[第二章 Instructions Language of the Computer]]