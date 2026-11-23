# SkyFusion (GAN-Transformer Physics Pipeline)

## 1. 概述 (Overview)
**SkyFusion** 是由德國比勒費爾德應用科技大學（HSBI）與羅森海姆技術大學（TH Rosenheim）於 2025 年在 EnviroInfo 提出的模組化物理感知生成預測藍圖，隸屬於德國聯邦 Helios 計畫。

### 四階段規劃架構：
1. **Stage 1 (SkyBuilder)**：ConvLSTM-UNet 結合 PatchGAN 判別器與多尺度 SSIM 損失，生成清晰的 15 分鐘天空視訊；
2. **Stage 2 (SkyCompressor)**：3D VQ-VAE 將視訊壓縮至離散碼本潛空間（$16 \times 16 \times T$）；
3. **Stage 3 (SkyForecaster)**：結合 GPT 自迴歸 Transformer 與 PhyCell 偏微分方程約束，並加入 TemporalGAN 保證時間連貫性；
4. **Stage 4 (SkyNowcaster)**：以 RMSE 與 CRPS 損失聯合訓練 CNN 功率回歸頭。

---

## 2. 特點與啟示
SkyFusion 展示了歐洲工業界在太陽能專案中對「PatchGAN 紋理細化」與「PVLib 太陽幾何 4 通道嵌入」的實踐，為後續高精度生成模型提供了幾何引導借鑒。

---

## 3. 關聯論文 (Associated Papers)
- [[2025-atmioui-skyfusion]]：SkyFusion 專案 Milestone 1 成果報告。
