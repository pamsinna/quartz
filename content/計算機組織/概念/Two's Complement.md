---
tags:
  - 概念
  - 計算機組織
---

# Two's Complement（二補數）

## 定義

二補數是現代電腦表示**有號整數**的標準方式。

$$x = -x_{n-1}2^{n-1} + x_{n-2}2^{n-2} + \cdots + x_0 2^0$$

最高位（MSB）為 **sign bit**：0 為正數，1 為負數。

## n-bit 二補數的範圍

$$-2^{n-1} \leq x \leq 2^{n-1} - 1$$

例：8-bit → 範圍 −128 ~ +127

## 求負數的捷徑

**取反（bitwise NOT）再加 1**

例：+5 = `0000 0101` → 取反 = `1111 1010` → 加 1 = `1111 1011` = −5 ✓

## Sign Extension（符號擴展）

將較少位數擴展到較多位數時，**重複填補 sign bit**：

- +3（4-bit）= `0011` → 擴展到 8-bit = `0000 0011` ✓
- −3（4-bit）= `1101` → 擴展到 8-bit = `1111 1101` ✓

RISC-V 的 12-bit 立即數在使用前都會 sign-extend 到 64-bit。

## 優點

- 加法硬體對有號/無號數通用，不需額外電路
- 只有一個 0（避免 sign-magnitude 的 +0/−0 問題）

## 相關概念

- [[Register]] — 暫存器儲存二補數整數
- [[Instruction Format]] — 立即數欄位以二補數表示，使用前 sign-extend
- [[第二章 Instructions Language of the Computer]]