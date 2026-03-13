---
tags:
  - 概念
  - 計算機組織
---

# Instruction Format（指令格式）

## 定義

指令格式規定了一條機器指令（32-bit）中各欄位的**位置與意義**。RISC-V 有六種格式，設計目標是在固定長度內盡量保持一致性。

## RISC-V 六種格式總覽

|格式|用途|特點|
|---|---|---|
|**R-type**|暫存器–暫存器運算（add, sub, and…）|含 rs1, rs2, rd, funct3, funct7|
|**I-type**|立即數運算 / Load（addi, ld…）|含 12-bit 立即數|
|**S-type**|Store（sd, sw…）|立即數拆成兩段（imm[11:5] + imm[4:0]）|
|**B-type**|條件分支（beq, bne…）|PC-relative，立即數為偏移量|
|**U-type**|長立即數（lui, auipc）|20-bit 立即數，放高位|
|**J-type**|無條件跳轉（jal）|PC-relative，21-bit 範圍|

## 共同欄位

所有格式都有 **opcode**（bits 6:0）；[[Register|暫存器]] 編號欄位（rs1, rs2, rd）位置在各格式中盡量保持一致，以簡化硬體解碼。

## 設計原則

> Good design demands good compromises：固定 32-bit 長度 + 六種格式，兼顧規律性與彈性。

## 相關概念

- [[ISA]] — ISA 定義合法的指令格式
- [[Register]] — rd, rs1, rs2 對應暫存器編號
- [[Two's Complement]] — 立即數欄位以二補數表示，使用前 sign-extend
- [[第二章 Instructions Language of the Computer]]