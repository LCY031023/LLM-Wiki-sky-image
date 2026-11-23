# Modified U-Net for PV Power Prediction

## 1. 概述與架構改進 (Overview & Architectural Adaptations)
在天空影像光電預測領域，直接利用 CNN 進行多標籤或單值回歸時常面臨空間訊息丟失與缺乏可解釋性的問題。
[[2024-nie-skygpt|Nie et al. (2024)]] 改良了原本用於生物醫學語意分割的 **U-Net** 架構，將其轉化為高解釋性的影像至功率回歸器（Image-to-Power Regressor）。

### 關鍵架構修改：
1. **空間加權特徵圖輸出 (Spatial Heatmap Regression)**：
   - Decoder 最終輸出的不是分割圖或單一全連接層數值，而是一個維持原圖解析度（$64 \times 64 \times 1$）的二維特徵圖。
   - 預測發電量為特徵圖所有像素之加權求和（Weighted Sum）：
     $$\hat{\mathcal{P}} = \sum_{h=1}^H \sum_{w=1}^W \omega_{h,w} \cdot \mathbf{F}_{h,w}$$
   - 該特徵圖可直觀視為「每個天空區域對地面光電板產電的正面或負面光通量貢獻權重」。
2. **殘差瓶頸塊 (Residual Bottleneck)**：
   - 將 U-Net 中間瓶頸層的傳統 Conv-BN-ReLU 結構替換為殘差模組（ResBlock），穩定梯度傳遞。
3. **輕量化剪枝**：適度精簡卷積層數以防在小規模案場數據上過擬合。

---

## 2. 不確定性建模搭配 (Ensemble & Probabilistic Inference)
- 該模型本體以確定性 MSE 進行訓練。
- 為了量化預測不確定性，作者採用 **10-Fold 交叉驗證（10-Fold Cross-Validation）** 保存 10 個次模型（Sub-models）。
- 推論時將隨機視訊生成器（如 [[skygpt|SkyGPT]]）產生的 $N_f=10$ 張未來影像，分別送入 10 個次模型，合計輸出 $10 \times 10 = 100$ 個點預測值，進而構建出機率分佈與預測區間。

---

## 3. 關聯論文 (Associated Papers)
- [[2024-nie-skygpt|Nie et al. (2024)]]：首次將此改良結構作為生成未來影像的下游功率預報頭。
