

#計算機組織 #概念

讓 [[Pipelining]] 無法在下一個 clock cycle 啟動下一條指令的情況（事件）。共三種類型。

---

## 1. Structure Hazard

**原因**：硬體無法支援同時執行的指令組合——所需資源正在被佔用。

**範例**：若只有一個 memory（instruction 與 data 共用），第一條指令在 MEM stage 存取資料時，第四條指令要 fetch 就會衝突。

**解法**：分開 instruction memory 與 data memory（或分開 instruction cache / data cache）。

---

## 2. Data Hazard

**原因**：某條指令必須等待前一條指令完成資料的讀寫才能繼續。

### Sol 1：Forwarding（Bypassing）

不等結果寫回 register file，直接從 pipeline 內部將結果送往需要的 stage。

偵測條件（EX hazard）：

```
if (EX/MEM.RegWrite
    and (EX/MEM.RegisterRd ≠ 0)
    and (EX/MEM.RegisterRd = ID/EX.RegisterRs1))
→ ForwardA = 10
```

偵測條件（MEM hazard）：

```
if (MEM/WB.RegWrite
    and (MEM/WB.RegisterRd ≠ 0)
    and (MEM/WB.RegisterRd = ID/EX.RegisterRs1))
→ ForwardA = 01
```

**Double data hazard**（同一 register 連續兩次被寫入）：優先使用 EX hazard forwarding，MEM hazard 只在 EX 條件不成立時才觸發。

### Load-Use Data Hazard

Forwarding 無法解決：load 的資料在 MEM stage 結束才可用，但下一條指令在 EX stage 就需要 → **無法往回傳遞**。

**偵測條件**：

```
if (ID/EX.MemRead
    and ((ID/EX.RegisterRd = IF/ID.RegisterRs1)
      or (ID/EX.RegisterRd = IF/ID.RegisterRs2)))
→ stall the pipeline
```

**Stall 做法**（插入 bubble）：

1. 阻止 PC 與 IF/ID pipeline register 更新（freeze）
2. 將 EX、MEM、WB 的七條 control signal 全設為 0（NOP）

### Sol 2：Code Scheduling（SW 解法）

由 compiler reorder 指令，讓 load 與使用其結果的指令之間插入不相依的指令，避免 stall。

---

## 3. Control Hazard（Branch Hazard）

**原因**：branch 決定的 next PC 在 pipeline 中途才知道，導致 fetch 到錯誤指令。

|解法|說明|
|---|---|
|**Stall on Branch**|Fetch branch 後立即 stall，等 branch 結果確定再繼續|
|**Branch Prediction**|預測 branch 結果，預測錯誤則 flush pipeline（詳下）|
|**Reduce Branch Delay**|將 branch 判斷移到 ID stage（原在 MEM），只需 flush 1 條指令（而非 3 條）|
|**Delayed Branch**|永遠執行 branch 後的那條指令（delay slot），由 compiler 安排一條無害指令填入|

### Branch Prediction

**Static branch prediction**：

- Backward branch → 預測 taken（迴圈通常往回跳）
- Forward branch → 預測 not taken
- 命中率約 80%

**Dynamic branch prediction**：

- 用 **branch prediction buffer**（branch history table）記錄 recent branch 的結果
- 以指令地址的低位 bits 為 index
- **1-bit predictor**：預測錯一次就立刻改變預測
- **2-bit predictor**：必須連續錯兩次才改變預測（對強傾向的 branch 更穩定）
- 預測錯誤時 flush 已 fetch 的指令並更新 history

### Flush 機制

- `IF.Flush`：將 IF/ID pipeline register 中的指令清零（→ NOP）
- `ID.Flush`：對 ID stage stall 時使用
- `EX.Flush`：Exception 或 branch misprediction 時，清除 EX stage 的 control signals

---

## 小結

|類型|原因|主要解法|
|---|---|---|
|Structure|資源衝突|分離 instruction/data memory|
|Data|資料相依|Forwarding；load-use → stall；code scheduling|
|Control|Branch 結果不確定|Prediction；reduce branch delay；delayed branch|

---

## 相關

- [[Pipelining]] — Pipeline 架構總覽
- [[ALU]] — Forwarding 的資料來源
- [[Performance]] — Hazard 造成的 CPI 增加
- [[第四章 The Processor]] — 完整 pipeline datapath 與 hazard 偵測硬體