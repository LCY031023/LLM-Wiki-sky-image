# Winkler Score (WS)

## 1. 概念定義與數學原理 (Definition & Formulation)
**溫克勒評分 (Winkler Score, WS)** 是一種專門評定「區間預測（Prediction Intervals, PI）」品質的評分規則（Scoring Rule）。

給定名義置信水平 $(1 - \alpha) \times 100\%$（常見為 90% 置信區間，即 $\alpha = 0.1$），模型預測的區間下界為 $L_k$、上界為 $U_k$，區間寬度 $\delta_k = U_k - L_k$。真實觀測值為 $x_k$。單點 WS 定義為：
$$\text{WS}_k = \begin{cases} 
\delta_k & \text{若 } L_k \le x_k \le U_k \\
\delta_k + \frac{2}{\alpha}(L_k - x_k) & \text{若 } x_k < L_k \\
\delta_k + \frac{2}{\alpha}(x_k - U_k) & \text{若 } x_k > U_k 
\end{cases}$$

全數據集平均 Winkler Score 為：
$$\text{WS} = \frac{1}{N} \sum_{k=1}^N \text{WS}_k$$

---

## 2. 評分特點與物理意涵
1. **獎勵窄區間（銳利度）**：當觀測值落在預測區間內時，懲罰即為區間寬度 $\delta_k$ 本身。區間越窄，分數越好（越低越好）。
2. **重罰漏網（失真懲罰）**：一旦觀測值跳出區間，將依據超出距離施加 $\frac{2}{\alpha}$（例如 90% 區間時倍率高達 20 倍）的嚴格線性懲罰。
3. **電網風險管理價值**：在太陽光電領域，區間過寬會增加電網備載容量成本；而脫靶（超出區間）則會引發未預期的電壓失穩，WS 能完美權衡這兩者。

---

## 3. 關聯文獻 (Associated Papers)
- [[2024-nie-skygpt|Nie et al. (2024)]]：SkyGPT 達成 26.72 kW，相較經典 CNN 基準 SUNSET（56.95 kW）取得超過 53% 的巨大降幅。
