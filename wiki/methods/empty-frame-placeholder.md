# Empty-Frame Placeholder Strategy

## 1. 概念定義與動機 (Definition & Motivation)
在光電電廠實際工程運維中，不同場站的感測硬體配備極不均勻：
- 有的場站配備高畫質天空攝影機但缺乏即時日射計/功率回傳；
- 有的場站僅有歷史發電電表數據但無相機；
- 攝影機在暴雨或硬體維護時可能暫時離線。

**空幀佔位策略 (Empty-Frame Placeholder Strategy)** 是由 Huang et al. (2026, PV-MM-Diffusion) 提出的一種訓練與推論機制，**使單一神經網路能在完全不改變架構的情況下，自適應支援三種輸入模態**。

---

## 2. 演算法運作機制
- **佔位符注入 (Zero-Tensor Placeholder)**：
  - 若僅輸入天空圖：將歷史功率輸入 $P^{0:7}$ 設為全零張量（“SI $\to$ SI PV”）；
  - 若僅輸入功率時序：將歷史天空影像輸入 $S^{0:7}$ 設為全零張量（“PV $\to$ PV”）；
  - 全模態輸入：維持真實數值輸入（“SI PV $\to$ SI PV”）。
- **混合模式損失函數 (Mixed-Mode Objective)**：
  $$\mathcal{L} = w_{SP} \mathcal{L}_{SP} + w_{SI} \mathcal{L}_{SI} + w_{PV} \mathcal{L}_{PV}$$
  作者在消融實驗中證實，採用平衡權重 $w_{SP} = w_{SI} = w_{PV} = \frac{1}{3}$ 能使網路同時保持全模態的高精度與單模態缺漏時的極佳容災能力。

---

## 3. 關聯論文 (Associated Papers)
- [[2026-huang-pv-mm-diffusion]]：提出空幀佔位策略並驗證其在缺模態情境下的魯棒性。
