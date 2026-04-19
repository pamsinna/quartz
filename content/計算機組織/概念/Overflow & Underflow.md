

#計算機組織 #概念

---

## Integer Overflow

結果超出 $n$-bit 可表示範圍時發生。

### Two's Complement 偵測規則

|運算|Overflow 條件|
|---|---|
|兩正數相加|結果 sign bit = 1|
|兩負數相加|結果 sign bit = 0|
|正數 − 負數|結果 sign bit = 1（< 0）|
|負數 − 正數|結果 sign bit = 0（≥ 0）|
|不同符號相加 / 相同符號相減|**不會** overflow|

### Unsigned Integer 偵測（由 compiler 處理）

- 加法：`sum < addend` → overflow
- 減法：`difference > minuend` → overflow

### 處理方式

- **Exception**（一般情況）：硬體觸發例外
- **Saturating arithmetic**（多媒體應用）：clamp 到最大可表示值
    - 例：audio clipping、video saturation

---

## Floating-Point Overflow & Underflow

|情況|定義|結果|
|---|---|---|
|**Overflow**|正 exponent 超出 exponent field 上限|±∞|
|**Underflow**|負 exponent 超出 exponent field 下限|±Denormalized / ±0|

### Single Precision 邊界

- Overflow threshold：$\approx \pm 3.4 \times 10^{38}$
- Underflow threshold：$\approx \pm 1.2 \times 10^{-38}$

### Double Precision 邊界

- Overflow threshold：$\approx \pm 1.8 \times 10^{308}$
- Underflow threshold：$\approx \pm 2.2 \times 10^{-308}$

### 注意

- FP overflow / underflow **不由硬體報錯**，需由軟體處理
- RISC-V 除以 0 同樣不產生硬體 exception，需程式自行處理
- 詳見 Hardware/Software Interface（教材 p. 206）

---

## 與 Integer Overflow 的差異

||Integer|Floating-Point|
|---|---|---|
|Overflow 意義|超出 bit 範圍|Exponent 過大|
|Underflow|不適用|Exponent 過小（絕對值太小）|
|硬體處理|可觸發 exception|通常不報錯，結果為 ±∞ 或 denorm|
|軟體責任|Unsigned 由 compiler 偵測|程式必須自行驗證|

---

## 相關

- [[Two's Complement]] — Integer overflow 的數學基礎
- [[Floating Point]] — FP overflow/underflow 的 exponent 機制
- [[ALU]] — 硬體偵測與處理
- [[第三章 Arithmetic for Computers]] — 各類範例與規則推導