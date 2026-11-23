---
title: "SkyGPT: Probabilistic ultra-short-term solar forecasting using synthetic sky images from physics-constrained VideoGPT"
authors: ["Yuhao Nie", "Eric Zelikman", "Andea Scott", "Quentin Paletta", "Adam Brandt"]
year: 2024
venue: "Advances in Applied Energy"
arxiv: "https://doi.org/10.1016/j.adapen.2024.100172"
code: "https://github.com/yuhao-nie/SkyGPT"
forecast_horizon: "15 min (ultra-short-term nowcasting)"
target_variable: "PV Power (kW)"
input_modalities: ["RGB Sky Image Sequences (Historical 15 mins)"]
method_type: "VideoGPT / PhyCell / VQ-VAE / Physics-Informed Generative Transformer"
probabilistic: true
metrics:
  crps: 2.81
  rmse: null
  skill_score: 0.23
tags:
  - paper
  - sky-image
  - generative-model
  - probabilistic-forecasting
  - video-prediction
  - physics-informed-dl
---

# SkyGPT: Probabilistic ultra-short-term solar forecasting using synthetic sky images from physics-constrained VideoGPT

- **作者**：Yuhao Nie, Eric Zelikman, Andea Scott, Quentin Paletta, Adam Brandt (Stanford University, MIT, Cambridge, ESA)
- **年份 / 發表場合**：2024 / *Advances in Applied Energy* (Vol. 14, 100172)
- **論文來源**：[DOI: 10.1016/j.adapen.2024.100172](https://doi.org/10.1016/j.adapen.2024.100172) | **程式碼**：[GitHub - yuhao-nie/SkyGPT](https://github.com/yuhao-nie/SkyGPT)
- **主要方法**：[[skygpt|SkyGPT (Physics-Constrained VideoGPT)]] + [[modified-unet-pv|Modified U-Net (Image-to-Power)]]
- **評測資料集**：[[stanford-rooftop-dataset|Stanford 30-kW Rooftop PV & Sky Image Dataset]]

---

## 1. 核心摘要與主要貢獻 (Executive Summary & Contributions)

本論文針對天空影像光電預測中的兩大瓶頸——**「雲層動態物理難以建模（易產生時延與模糊）」**與**「缺乏機率不確定性量化」**，提出了一套兩階段（Two-stage）深度生成預報架構 **SkyGPT**。
第一階段利用結合偏微分方程（PDE）物理先驗模組（PhyCell）與離散量化自編碼器（VQ-VAE + Transformer）的隨機視頻預測模型，生成多個高逼真度且具物理約束的未來天空影像（Future Sky Scenarios）；第二階段透過特徵圖加權求和的改良型 U-Net 回歸模型，將生成的未來影像映射至未來 15 分鐘的發電功率區間。

### 主要學術貢獻：
1. **首創隨機視頻生成應用於光電機率預報**：打破傳統僅使用單一確定性影像外推或時間序列分位數回歸的局限，利用生成式 AI 採樣多個可能的天空演化情境（Plausible Futures）。
2. **物理約束 Phy-Transformer 架構**：在 VideoGPT 的潛空間中解耦「物理動力學（由近似線性 PDE 的 PhyCell 負責）」與「細粒度雲外觀紋理（由 Transformer 負責）」，大幅解決傳統 VideoGPT 泛化性不足與 ConvLSTM 長期預報嚴重模糊的問題。
3. **可解釋的影像-功率映射架構**：改良 U-Net 結構，產生與原圖同解析度的單通道特徵圖，以加權求和回歸發電量，實現像素級貢獻的可解釋性，並搭配 10-Fold 交叉驗證捕捉模型結構不確定性。
4. **顯著提升機率指標表現**：在 Stanford 實際案場測試集上達成 **CRPS 2.81 kW** 與 **Winkler Score (WS) 26.72 kW**，相較經典 CNN 基準（SUNSET）提升 13% CRPS 與 53% WS，預測技能分數（FS）達 23%。

---

## 2. 問題定義與硬體配置 (Problem Formulation & Hardware Setup)

- **預測目標變數**：未來第 15 分鐘（$t+15$）之光電發電功率 $\mathcal{P}_{t+15}$（kW）。
- **預測時間尺度 (Forecast Horizon)**：超短期臨近預報（Nowcasting），$\Delta t = 15$ 分鐘。
  - 輸入：過去 15 分鐘歷史影像序列，步長 2 分鐘（$t-15, t-13, \dots, t-1$，共 8 幀）。
  - 生成：未來 15 分鐘影像序列（$t+1, t+3, \dots, t+15$，共 8 幀），僅取最後一幀 $t+15$ 進入功率預測階段。
- **全天空攝影機規格**：
  - 型號：海康威視魚眼鏡頭（Hikvision DS-2CD6362F-IV），安裝於史丹佛大學 Green Earth Sciences 大樓屋頂。
  - 取樣頻率：1 分鐘 1 幀（後續以 2 分鐘採樣以平衡訓練成本）。
  - 影像解析度：原始 $2048 \times 2048$ 像素，降採樣至 $64 \times 64$ 像素。外圍無效區域採用二值掩碼（Binary Mask）濾除。
- **光電案場規格**：
  - 距離相機約 125 公尺的 30-kW 屋頂光電系統，傾角（Elevation）22.5°，方位角（Azimuth）195°。
- **輸入特徵組合**：
  - 視頻預測階段：僅輸入純歷史天空影像序列（無歷史功率，防止模型過度依賴數值時延）。
  - 功率預測階段：輸入 $t+15$ 預測天空影像。

---

## 3. 方法論與架構細節 (Methodology & Architecture)

整體架構分為兩階段循序 Pipeline：

```text
[歷史天空影像序列 (t-15 ~ t-1)] 
            │
            ▼
┌──────────────────────────────────────────────┐
│  Stage 1: SkyGPT (Stochastic Video Predictor) │
│  - VQ-VAE: 影像離散 Codebook 量化             │
│  - PhyCell: 偏微分方程 (PDE) 空間導數擬合 (物理分支)  │
│  - Transformer: 局部 Patch 注意力自迴歸 (細節分支)│
│  - 採樣 Nf=10 個潛變量路徑                     │
└──────────────────────────────────────────────┘
            │ 產生 10 種未來的 t+15 天空影像
            ▼
┌──────────────────────────────────────────────┐
│  Stage 2: Modified U-Net (PV Output Model)   │
│  - 2D Conv Encoder -> ResBlock -> Decoder    │
│  - 輸出 64x64x1 像素功率特徵圖 -> 空間加權求和   │
│  - 10-Fold 交叉驗證模型 (共 10 個次模型)       │
└──────────────────────────────────────────────┘
            │ 10 張影像 × 10 個模型 = 100 個預測點
            ▼
[未來 15 分鐘 PV 機率分佈與置信區間 (5%-95%, 25%-75%)]
```

### 3.1 SkyGPT 生成機制
1. **VQ-VAE 離散潛空間表示**：
   - 3D-CNN 編碼器將連續視頻特徵映射至離散 Codebook $C = \{e_i | 1 \le i \le K\}$。離散表示極適合歸納晴天、多雲、陰天等離散天氣型態。
   - 損失函數：$\mathcal{L}_{VQ-VAE} = \mathcal{L}_{recon} + \mathcal{L}_{codebook} + \beta \mathcal{L}_{commit}$。
2. **Phy-Transformer 解耦潛建模**：
   - **PhyCell 分支**：透過二維卷積近似二階線性偏微分方程（如對流擴散方程 Advection-Diffusion、波動方程式），學習大尺度雲層位移物理趨勢。
   - **Transformer 分支**：利用 Patch-level 自注意力機制建模局部雲層變形、凝結與消散細節。
   - 透過 3D ResNet 編碼歷史條件幀，搭配 Cross-Attention 引導生成。
   - 聯合損失：$\mathcal{L}_{Phy-transformer} = \mathcal{L}_{moment} + \mathcal{L}_{cross-entropy}$。

### 3.2 改良型 U-Net 功率映射
- 取代標準分類/分割輸出，保留 Decoder 輸出尺寸為 $64 \times 64 \times 1$，物理意涵為「每個天空像素對光電板受光遮蔽的貢獻權重」，經加權求和輸出單一功率值 $\hat{\mathcal{P}}$。
- 損失函數為確定性 MSE。利用 10-fold 次模型（Sub-models）搭配 $N_f=10$ 個生成情境，在推論時得到 $10 \times 10 = 100$ 個取樣點，構築出經驗累積分佈函數（ECDF）。

---

## 4. 關鍵實驗結果與基準比對 (Key Results & Benchmarks)

測試集選取跨年度獨立 5 個多雲日（2,582 筆樣本），評估指標包含 [[continuous-ranked-probability-score|CRPS]]、[[winkler-score|Winkler Score (WS, 90% 區間)]] 與預測技能分數（Forecast Skill, FS）：

### 4.1 光電機率預測性能對比（測試集 Test Set）
| 模型架構 | 預測類型 | CRPS (kW) ↓ | FS (基於 CRPS) ↑ | WS (kW) ↓ | 核心特點 / 評語 |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Smart Persistence (SPM)** | 確定性基準 | 3.67 | 0% | — | 無法預測雲層動態遮蔽突變 |
| **SUNSET (AlexNet-like CNN)** | 深度學習基準 | 3.31 | 10% | 56.95 | 依賴歷史功率，爬坡事件有明顯時間滯後 |
| **ConvLSTM $\to$ U-Net** | 確定性生成 | 3.09 | 16% | 43.82 | 長時預報畫面模糊，低估極端值 |
| **PhyDNet $\to$ U-Net** | 確定性物理 | 3.12 | 15% | 45.20 | 捕捉大尺度位移，但缺乏多模態多樣性 |
| **PhyDNet+GAN $\to$ U-Net** | 確定性對抗 | 3.02 | 18% | 39.54 | 畫面清晰度高，但無法輸出隨機區間 |
| **VideoGPT $\to$ U-Net** | 隨機生成 | 3.34 | 9% | 36.22 | 驗證集良好但在測試集過擬合（泛化弱） |
| **SkyGPT $\to$ U-Net (本文方法)** | **隨機物理生成** | **2.81** | **23%** | **26.72** | **綜合 Reliability 與 Sharpness 最優** |
| *Real Future SI $\to$ U-Net (Oracle)* | *理想上限* | *2.34* | *36%* | *27.13* | *視訊預報 100% 正確下的理論上限* |

### 4.2 核心實驗發現：
1. **生成多樣性樣本數的邊際效應**：實驗發現 $N_f$ 從 1 增加至 10 時，CRPS 與 WS 呈現斷崖式下降；但超過 10 樣本後指標趨於平緩（收斂）。因此 $N_f = 10$ 為運算成本與預測精度的最佳平衡點。
2. **誤差解耦分析 (Error Decomposition)**：
   - 從 VideoGPT 換為 SkyGPT 提升約 13% CRPS；
   - 即使影像預測 100% 完美（Real Future SI），CRPS 仍有 2.34 kW，證明**第二階段的「天空影像 $\to$ 發電功率」映射誤差才是整個預報系統的主要誤差源**。

---

## 5. 深入剖析與局限性 (Critical Analysis & Limitations)

### 優勢剖析：
1. **消除時滯（No Temporal Lag）**：傳統 CNN（如 SUNSET）過度擬合歷史發電功率數值，在劇烈雲遮突變（Ramp Events）時總是「落後反映」；SkyGPT 直接依據雲層空間位移推導，成功提前捕捉功率階躍。
2. **動態調整不確定性寬度**：天氣平穩時預測區間自發收窄，劇烈雲遮對流時區間變寬，符合物理真實直覺。

### 失效案例與瓶頸：
1. **極值覆蓋不全**：在功率尖峰與深谷處（極端晴空或極暗暴雨雲），即使輸入真實影像，U-Net 偶爾仍無法完全包絡實際數值。
2. **偶發過寬區間（Sharpness 不足）**：在發電量穩定的時段，SkyGPT 偶爾因隨機採樣漂移產生不必要的形變，導致區間偏寬。主因是 Stage 1 的視訊生成與 Stage 2 的功率損失未實施端到端反向傳播反饋。
3. **推論延遲考量**：在單張 Nvidia A100 GPU 上，單一採樣需耗費約 0.64 秒，產生 50 個採樣需 32 秒，雖能滿足 15 分鐘預報需求，但若擴展至更高解析度或更短時程（如 1 分鐘），算力需求顯著。

---

## 6. 與其他文獻之脈絡關聯 (Connections to Other Work)

- **繼承與改進**：
  - 繼承 [[videogpt|VideoGPT]] 的離散 VQ-VAE + Transformer 結構，但引入 [[phydnet|PhyDNet]] 的 PhyCell 模組解決純 Transformer 物理規律缺失與泛化性差的問題。
  - 對比並超越經典基準 [[sunset|SUNSET (Sun et al., 2019)]] 與 [[convlstm|ConvLSTM (Shi et al., 2015)]]。
- **被後續研究突破**：
  - [[2025-xiong-multimodal-diff-transformer|Xiong et al. (2025)]] 指出本架構的兩大核心盲點（僅依賴單一影像模態缺乏歷史功率、忽略下游回歸之認知不確定性），改採 Neural SDE + 條件擴散模型與多模態 Transformer GPA，在相同 Stanford 測試集上將 CRPS 從 2.81 kW 突破至 2.44 kW，Winkler Score 降至 19.62 kW。
- **關聯核心概念**：
  - 機率指標：[[continuous-ranked-probability-score|CRPS]]、[[winkler-score|Winkler Score (WS)]]、[[forecast-skill|Forecast Skill (FS)]]。
  - 氣象與光電物理：[[clear-sky-index|Clear-Sky Index ($k_t$)]]、[[ramp-events|PV Ramp Events]]、[[cloud-motion-dynamics|Cloud Motion Dynamics]]。
- **後續延伸啟示**：
  - 論文提及未來可將 Vision Transformer (ViT) 引入 Stage 2，或結合輔助數值天氣預報與衛星圖資進行跨小時擴展。
