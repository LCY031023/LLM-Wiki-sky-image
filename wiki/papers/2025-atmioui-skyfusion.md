---
title: "SkyFusion: A Physics-Informed GAN-Transformer Framework for Intra-Hour Sky Forecasting and PV Prediction"
authors: ["Naoufal El Atmioui", "Grit Behrens", "Andreas Boschert", "Michael Zehner"]
year: 2025
venue: "EnviroInfo 2025, Lecture Notes in Informatics (LNI)"
arxiv: null
code: null
forecast_horizon: "15 min (30 frames at 30-sec intervals)"
target_variable: "PV Power Output & Sky Image Sequences"
input_modalities: ["RGB Sky Images + 4th Channel Binary Sun Mask (4, 128, 128)"]
method_type: "ConvLSTM-UNet + PatchGAN / Planned 4-Stage GAN-Transformer (SkyFusion)"
probabilistic: true
metrics:
  crps: null
  rmse: null
  skill_score: null
tags:
  - paper
  - sky-image
  - generative-model
  - probabilistic-forecasting
  - gan
  - convlstm
  - sun-position-masking
  - german-pv-testfield
---

# SkyFusion: A Physics-Informed GAN-Transformer Framework for Intra-Hour Sky Forecasting and PV Prediction

- **作者**：Naoufal El Atmioui, Grit Behrens, Andreas Boschert, Michael Zehner (比勒費爾德應用科技大學 HSBI Solar Computing Lab, 羅森海姆技術大學 TH Rosenheim 太陽能系統實驗室, 德國)
- **年份 / 發表場合**：2025 / *EnviroInfo 2025*, Lecture Notes in Informatics (LNI), Gesellschaft für Informatik, Bonn.
- **論文來源**：[DOI: 10.18420/env2025_01](https://doi.org/10.18420/env2025_01)
- **主要方法**：[[skyfusion|SkyFusion 4-Stage Pipeline]] + [[sun-position-masking|Sun Position Masking (4th Channel via PVLib)]]
- **評測資料集**：[[german-pv-testfield-dataset|German PV Test Field ASI Dataset (asi_16030 & asi_16515)]]

---

## 1. 核心摘要與主要貢獻 (Executive Summary & Contributions)

本論文介紹了德國聯邦資助的 **Helios** 專案成果，提出了一套模組化的物理感知對抗生成框架 **SkyFusion**，旨在實現 15 分鐘超短期全天空影像與光電發電功率預報。
本篇為該專案的 **Milestone 1（第一階段里程碑）報告**，重點聚焦於：
1. **太陽幾何先驗物理注入**：利用 PVLib 依據地理座標精確計算太陽高度角與方位角，生成二值化日盤遮罩（Binary Sun Mask），將其作為**第四通道（4th Channel: RGB + Sun Mask）**輸入，協助神經網路在雲層遮蔽或環日眩光下精確鎖定太陽幾何位置。
2. **PatchGAN 對抗性細化基線**：針對經典 ConvLSTM-UNet 因最小化像素級均方誤差而產生的嚴重畫面模糊問題，引入 PatchGAN 局部判別器進行對抗訓練，顯著增強雲層微觀邊緣與紋理的銳利度。
3. **規劃四階段模組化演進藍圖**：
   - **Stage 1 (SkyBuilder)**：ConvLSTM-UNet + PatchGAN（生成 30 幀，步長 30 秒）；
   - **Stage 2 (SkyCompressor)**：3D VQ-VAE 潛空間離散化（$16 \times 16 \times T$，碼本 $K \approx 1024$）；
   - **Stage 3 (SkyForecaster)**：GPT 式自迴歸 Transformer + PhyCell 物理約束 + TemporalGAN；
   - **Stage 4 (SkyNowcaster)**：以 RMSE 與 CRPS 損失聯合訓練 CNN 功率回歸器。

---

## 2. 問題定義與硬體配置 (Problem Formulation & Hardware Setup)

- **預測時間尺度 (Forecast Horizon)**：15 分鐘（$t+1 \dots t+30$ 幀，步長 30 秒，共 30 幀）。
  - 歷史輸入：過去 6 分鐘（12 幀，步長 30 秒）。
- **特徵輸入格式 (4 通道張量)**：
  - 將 RGB 3 通道影像與太陽位置二值遮罩拼接，組成形狀為 `(Batch, 4, 128, 128)` 的輸入張量。
- **全天空攝影機與資料集 (German PV Test Field)**：
  - 兩台商用全天空攝影機（`asi_16030` 與 `asi_16515`），原解析度擷取間隔 15 秒（原始資料超 40 萬張）。
  - 清洗與下採樣：剔除受損幀，保留微小雨滴等真實噪聲，降採樣至 30 秒間隔，最終獲得 178,000 張影像。
  - 影像降採樣至 $128 \times 128$ 像素。
- **訓練硬體**：單張 NVIDIA A100 (80GB) GPU，基於 PyTorch Lightning 框架。

---

## 3. 方法論與架構細節 (Methodology & Architecture)

### 3.1 太陽位置遮罩生成 ([[sun-position-masking|Sun Position Masking]])
利用 `pvlib-python` 依據即時時間戳、緯度與經度推導太陽天頂角與方位角，在全黑背景上繪製對應大小的白色日盤圓盤：
$$\mathbf{X}_{input} = \text{Concat}\left(\mathbf{I}_{RGB}, \mathbf{M}_{sun}\right) \in \mathbb{R}^{4 \times 128 \times 128}$$
在多雲甚至全陰天條件下，相機圖像中肉眼已無法分辨太陽確切位置，此輔助通道能提供關鍵的幾何物理錨點，防止生成模型將雲團運動誤認為日盤漂移。

### 3.2 複合損失函數 (Composite Loss with PatchGAN)
為平衡數值強度準確度與視覺銳利度，ConvLSTM-UNet 採用混合損失：
$$\mathcal{L} = \mathcal{L}_{L1} + \lambda_{ssim} \mathcal{L}_{SSIM} + \lambda_{adv} \mathcal{L}_{PatchGAN}$$
- $L_1$ 損失保障整體像素光通量強度不漂移；
- 多尺度 SSIM 損失保證結構拓撲完整；
- PatchGAN 判別器迫使生成器產出真實的高頻雲彩紋理，克服 ConvLSTM 固有的模糊瓶頸。

---

## 4. 初步實驗結果與發現 (Preliminary Results)

1. **定性視覺對比（Fig. 4 vs. Fig. 5）**：
   - 未添加 PatchGAN 的純 ConvLSTM-UNet：在預測 5 分鐘後迅速退化為均勻平滑的灰色漸層，完全丟失雲團邊緣；
   - 整合 PatchGAN 後：生成的雲體結構維持高度清晰的邊緣與光暗陰影，顯著抑制了視訊模糊。
2. **物理數據增強策略**：
   - 實驗證實僅能採用物理合理增強（水平翻轉 $p=0.3$、輕微旋轉、高斯模糊與色彩抖動）；
   - **嚴禁使用 MixUp 與 CutMix**，因為跨幀與拼貼會嚴重破壞雲層流體力學的時間與空間連續性。

---

## 5. 深入剖析與局限性 (Critical Analysis & Limitations)

### 優勢剖析：
1. **幾何引導創新**：直接將太陽天文幾何轉化為 2D 空間通道（4th Channel），相比傳統直接把天頂角作為 1D 數值傳入全連接層，4D 卷積能更直觀地學習「雲層何時跨越日盤邊界」。

### 局限性與後續規劃：
1. **本篇為 Milestone 1 階段性報告**：文中僅展示了視訊預測的視覺質量改善與架構藍圖，尚未完整回報測試集上的 CRPS 與 Winkler Score 數值（數值回歸規劃於後續階段開展）。
2. **GAN 訓練穩定性問題**：論文坦承在多尺度與長時間外推時，對抗訓練面臨穩定性挑戰，這也是後來主流文獻全面轉向擴散模型（Diffusion）的重要原因。

---

## 6. 與其他文獻之脈絡關聯 (Connections to Other Work)

- **與 [[2024-nie-skygpt|SkyGPT]] 之關聯**：
  - SkyFusion 旨在吸收 SkyGPT 的 PhyCell 概念，但試圖透過 PatchGAN 與 SSIM 損失克服 SkyGPT 紋理模糊的問題。
- **與其他生成預報文獻對比**：
  - 提供了罕見的 **RGB+Sun 4 通道空間幾何輸入** 範例，值得後續 [[2026-suwandee-single-stage-diffusion|單階段擴散]] 或 [[2026-fabel-farsky|FarSky]] 借鑒。
- **涉及核心概念**：
  - [[sun-position-masking|Sun Position Masking]]
  - [[german-pv-testfield-dataset|German PV Test Field Dataset]]
