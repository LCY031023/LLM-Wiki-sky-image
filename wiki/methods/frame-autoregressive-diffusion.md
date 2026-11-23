# Frame AutoRegressive (FAR) Latent Diffusion

## 1. 概述 (Overview)
**FAR (Frame AutoRegressive Diffusion)** 是由 Gu et al. (2025) 提出的一種基於**流匹配（Flow-Matching）**的長程自迴歸潛空間視訊擴散架構。
在 [[2026-fabel-farsky|FarSky (2026)]] 中，FAR 被選用為在任務感知潛空間（DCAEIrr）中外推未來天空演化情境的生成骨幹。

---

## 2. 演算法機制
1. **流匹配訓練目標 (Flow-Matching Objective)**：
   在潛表示 $z$ 上逐步加入高斯噪聲，神經網路僅對未來 16 幀的潛狀態計算流匹配回歸損失，保留乾淨的歷史上下文幀作為自迴歸條件。
2. **跨步自迴歸外推**：
   訓練時設定 16 幀上下文 $\to$ 16 幀目標（覆蓋 16 分鐘）；推論時，將預測出的未來潛狀態反饋至上下文窗口，循環推展至 30 分鐘超短期全時程。
3. **隨機機率採樣 (Stochastic Sampling)**：
   採樣 $S = 4$ 條不同的潛軌跡，經由 Student-$t$ 分佈擬合導出完整的機率置信區間，在保持極高爬坡事件（Ramp Events）捕捉率（F1 > 60%）的同時提供可靠的區間不確定性。

---

## 3. 關聯論文 (Associated Papers)
- [[2026-fabel-farsky]]：將 FAR 流匹配擴散模型應用於西班牙 PSA 案場進行 0~30 分鐘機率預測。
