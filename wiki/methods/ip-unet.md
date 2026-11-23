# IP-U-Net (Image-to-Power U-Net with Attention Gates)

## 1. 概述與設計動機 (Overview & Motivation)
**IP-U-Net** 是專為「天空影像至光電功率單點回歸（Image-to-Power）」設計的改良型卷積神經網路架構，由 Pan & Lim (2025) 提出。
其繼承了 [[modified-unet-pv|Nie et al. (2024) 改良 U-Net]] 的可解釋性空間加權回歸思路，但在跳躍連接中增加了**注意力門控（Attention Gate, AG）**。

---

## 2. 注意力門控機制 (Attention Gate Mechanism)
在全天空影像中，相機周圍通常包含無效的外框、建築物天際線邊緣與雜散光噪聲；傳統 U-Net 的跳躍連接（Skip Connections）會直接將低層編碼器的空間噪聲傳至解碼層。
IP-U-Net 在跳躍連接處加入 AG 模組：
- 利用深層解碼器的語意信號作為門控（Gating Vector），對淺層編碼器的特徵圖進行逐像素加權：
  $$\alpha = \sigma\left( W_\psi^T (\text{ReLU}(W_x^T x_l + W_g^T g + b_g)) + b_\psi \right)$$
- 此機制能主動壓制天空邊界噪聲，自動聚焦於**太陽盤周圍區域（Circumsolar Area）與高對比度的迎風面雲邊緣**。

---

## 3. 性能影響 (Impact)
消融實驗證實，去除 Attention Gate 後，無論輸入哪一種生成視訊，下游 PV 預測的 CRPS 與 Winkler Score 皆會全面惡化，證明空間雜訊過濾對光電功率精準度極為關鍵。

---

## 4. 關聯論文 (Associated Papers)
- [[2025-pan-igpt-two-stage]]：提出 IP-U-Net 並驗證 Attention Gate 的有效性。
