# Continuous Ranked Probability Score (CRPS)

## 1. 概念定義與數學原理 (Definition & Mathematical Formulation)
**連續分級機率評分 (CRPS)** 是機率預測（Probabilistic Forecasting）領域中最具代表性且被視為黃金標準的綜合評估指標。它評估預測的累積機率分佈函數（CDF）與真實觀測值之間的距離。

對單一樣本 $k$，若模型輸出的預測累積分佈為 $F_k(x)$，真實觀測值為 $y_k$（其真實分佈等同於在 $y_k$ 處躍升的 Heaviside 階躍函數 $H(x - y_k)$），則 CRPS 定義為：
$$\text{CRPS}(F_k, y_k) = \int_{-\infty}^{+\infty} \left[ F_k(x) - H(x - y_k) \right]^2 dx$$

總體評分為 $N$ 筆樣本之均值：
$$\text{CRPS} = \frac{1}{N} \sum_{k=1}^N \text{CRPS}(F_k, y_k)$$

---

## 2. 在太陽光電與氣象預報中的核心優勢
1. **維度與物理量一致 (Dimensionally Consistent)**：
   - CRPS 的單位與被預測物理量完全相同（例如在光電預測中為 $\text{kW}$ 或 $\text{W/m}^2$）。
2. **兼顧可靠性與銳利度 (Reliability & Sharpness)**：
   - 同時考核預測區間是否覆蓋真實值（Reliability），以及機率分佈是否足夠集中、不產生過寬的無效預測（Sharpness）。
3. **向下相容確定性預報**：
   - 當預測退化為確定性單點預測時，CRPS 數值嚴格等價於平均絕對誤差（MAE）：
     $$\text{CRPS}_{\text{deterministic}} = \text{MAE} = \frac{1}{N}\sum |y_k - \hat{y}_k|$$
   - 這使得隨機機率預報（如 [[skygpt|SkyGPT]]）能與傳統確定性基準（如 Persistence、SUNSET）在相同基準下嚴格對比。

---

## 3. 關聯文獻 (Associated Papers)
- [[2024-nie-skygpt|Nie et al. (2024)]]：在 15 分鐘預報時程中取得 2.81 kW 的 SOTA 成績。
