---
title: "FarSky: Task-Aware Latent-Space Coupling for Generative Intra-Hour Solar Forecasting"
authors: ["Yann Fabel", "Bijan Nouri", "Milon Miah", "Niklas Blum", "Luis F. Zarzalejo", "Julia Kowalski", "Robert Pitz-Paal"]
year: 2026
venue: "arXiv preprint (arXiv:2608.11254v1 [cs.LG])"
arxiv: "https://arxiv.org/abs/2608.11254"
code: "https://doi.org/10.5281/zenodo.21682388"
forecast_horizon: "0-30 min (intra-hour nowcasting, 1-min resolution)"
target_variable: "Global Horizontal Irradiance (GHI, W/m^2)"
input_modalities: ["RGB All-Sky Images (16 historical frames at 1-min cadence)"]
method_type: "FarSky / Task-Aware DCAEIrr / Latent Flow-Matching Diffusion (FAR) / Direct Latent Irradiance Decoding"
probabilistic: true
metrics:
  rmse: 50.0 # W/m^2 on PSA-Bench2019 (15-min avg); 98.0 on SVA-Test
  skill_score: 0.320 # 32.0% FS over Scaled Persistence (PSA-Bench); 0.241 on SVA-Test
  crps: null # CRPSS: 0.715 on PSA-Bench2019; 0.540 on SVA-Test
tags:
  - paper
  - sky-image
  - generative-model
  - probabilistic-forecasting
  - diffusion-model
  - latent-diffusion
  - task-aware-latent-space
  - ramp-events
  - psa-almeria-dataset
---

# FarSky: Task-Aware Latent-Space Coupling for Generative Intra-Hour Solar Forecasting

- **作者**：Yann Fabel, Bijan Nouri, Milon Miah, Niklas Blum, Luis F. Zarzalejo, Julia Kowalski, Robert Pitz-Paal (DLR 德國航空太空中心太陽能研究所, 亞琛工業大學 RWTH Aachen, 西班牙能源環境與技術研究中心 CIEMAT)
- **年份 / 發表場合**：2026 / *arXiv:2608.11254v1*
- **論文來源**：[arXiv:2608.11254](https://arxiv.org/abs/2608.11254) | **評測資料集**：[Zenodo (10.5281/zenodo.21682388)](https://doi.org/10.5281/zenodo.21682388)
- **主要方法**：[[farsky-latent-coupling|FarSky (Task-Aware Latent-Space Coupling)]] + [[frame-autoregressive-diffusion|FAR (Frame AutoRegressive Latent Diffusion)]]
- **評測資料集**：[[psa-almeria-dataset|PSA-Bench2019 & SolarVision Almería (SVA-Test) Datasets (Spain)]]

---

## 1. 核心摘要與主要貢獻 (Executive Summary & Contributions)

本論文針對先前兩階段生成預報（如 [[2024-nie-skygpt|SkyGPT]]）以及端到端回歸模型（E2E-TF）的固有缺陷，提出了 **FarSky**——一套基於**「任務感知潛空間耦合（Task-Aware Latent-Space Coupling）」**的擴散生成預測架構。

### 現有方法的痛點：
1. **端到端回歸（如 MSE 訓練的 Transformer）**：為了最小化均方誤差，輸出高度平滑化，在面對劇烈光電爬坡事件（Ramp Events）時幾乎完全失效（檢測召回率 Recall 與 F1-score 接近 0！）。
2. **傳統解耦兩階段（如 SkyGPT）**：自編碼器（VAE）僅針對視覺重建訓練，潛空間未編碼任何輻射特徵；當下游獨立回歸器讀取生成影像時，生成影像的微小形變引發嚴重的**「介面級分佈偏移（Interface Domain Shift）」**，導致顯著的系統性低估偏差（MBE 達到 -35 至 -44 $W/m^2$）。

### FarSky 解決方案：
FarSky 設計了多任務自編碼器 **DCAEIrr**：在影像編碼器輸出潛向量 $z = E(x)$ 後，同時訓練一個影像解碼器 $\hat{x} = D_x(z)$ 與一個**日照預測 MLP 頭 $\hat{y} = D_y(z)$**。
在潛空間中強制將日照資訊（太陽盤光度、雲光學厚度）與影像特徵耦合；接著利用潛空間流匹配擴散模型（FAR）直接預測未來的潛狀態 $z_{t+1 \dots t+h}$，最後**直接透過潛空間 MLP 頭 $D_y$ 解碼出未來 GHI 數值**，徹底繞過像素級下游回歸！

### 主要學術貢獻：
1. **任務感知潛空間多任務學習 (Task-Aware Multi-Task Autoencoding)**：
   在 Deep Compression Autoencoder (DCAE) 中加入日照預測損失 $\mathcal{L}_{irr} = \|\hat{y} - y\|_1$，使潛表示兼具高保真視覺特徵與光通量物理特徵。零時步真實影像評估顯示，DCAEIrr 的日照回歸 RMSE 比獨立 CNN 降低 50%（25.9 vs. 52.0 $W/m^2$），且幾乎消除了系統性偏差（MBE 從 -24.9 縮小至 -5.7 $W/m^2$）。
2. **潛空間直接解碼預測 (Direct Latent Decoding)**：
   未來潛狀態由 FAR 擴散模型生成後，直接經由潛空間 MLP 頭映射至 GHI，完全消除「生成像素影像 $\to$ 獨立回歸器」之間引發的 30~40 $W/m^2$ 系統性負偏差。
3. **優異的極端爬坡（Ramp Events）捕捉能力**：
   在極端雲遮突變檢測中，相較於端到端模型的 F1 接近零（0.004），FarSky 的爬坡 F1-score 超過 **60%**（達到 0.618 與 0.673），Recall 高達 80% 以上。
4. **雙資料集全面領先**：在西班牙 PSA 的 PSA-Bench2019 與 SVA-Test 兩大基準測試集上，FarSky 15 分鐘預報技能分數（FS）分別達到 **32.0%** 與 **24.1%**，機率技能分數（CRPSS）達 **0.715** 與 **0.540**，全面優於 SkyGPT 與端到端深度模型。

---

## 2. 問題定義與硬體配置 (Problem Formulation & Hardware Setup)

- **預測目標變數**：未來 0~30 分鐘（1 分鐘高頻時間解析度）的水平面總日照強度 $GHI(t+1 \dots t+30)$。
- **預測時間尺度 (Forecast Horizon)**：超短期/小時前預報（0–30 分鐘，主評估為前 15 分鐘平均）。
- **全天空攝影機規格**：
  - 德國 Mobotix Q25 與 Q26 全球面魚眼攝影機（180° FOV），CMOS 感測器固定曝光 160 $\mu\text{s}$，採樣間隔 30 秒。
  - 影像幾何校正：以太陽/月亮天頂角定位幾何中心，裁剪為天頂居中之正方形，降採樣至 $128 \times 128$ 像素。
- **輻射感測設備**：
  - ISO 9060:2018 Class A Kipp & Zonen 高精度熱電堆日射計（1 Hz 取樣，1 分鐘平均）。
- **資料站點 (Plataforma Solar de Almería, PSA, Spain)**：
  - 位於西班牙阿爾梅里亞沙漠高原（BWk 冷沙漠氣候，海拔 500m，周圍受內華達山脈地形雲強烈擾動）。
  - 涵蓋 4 個子測站（HP, KONTAS, METAS, PVOT）2019–2024 年近 5 年的多機位巨量資料。

---

## 3. 方法論與架構細節 (Methodology & Architecture)

整體架構分為兩大訓練階段：

```text
階段一：任務感知潛空間表徵學習 (Representation Learning)
┌────────────────────────────────────────────────────────┐
│ 天空影像 X_t ──► DCAEIrr Encoder E ──► 潛變量 z = E(x) │
│                                         ├──► 解碼器 D_x(z) ──► 重建影像 \hat{X}_t
│                                         └──► MLP 頭 D_y(z)  ──► 預測日照 \hat{y}_t (GHI)
│ 聯合損失: L = L_DCAE + \lambda_irr * ||\hat{y}_t - y_t||_1
└────────────────────────────────────────────────────────┘

階段二：潛空間自迴歸流匹配擴散 (Generative Forecasting in Latent Space)
┌────────────────────────────────────────────────────────┐
│ 歷史潛狀態序列 [z_{t-15}, ..., z_{t_0}]                 │
│                          │                             │
│                          ▼ (FAR 流匹配潛擴散)           │
│ 預測未來潛狀態序列 [z_{t+1}, ..., z_{t+h}]             │
│                          │                             │
│      ┌───────────────────┴───────────────────┐         │
│      ▼                                       ▼         │
│ 解碼器 D_x(z)                             MLP 頭 D_y(z)│
│ 未來天空影像序列 (可視化副產物)           未來 GHI 預測時序 (主任務)│
└────────────────────────────────────────────────────────┘
```

### 3.1 DCAEIrr 核心架構
- 骨幹採用 Deep Compression Autoencoder (DCAE)，具備極高的空間壓縮比與特徵保持力；
- 日照預測頭：全域平均池化（Global Average Pooling）+ 雙層 MLP（隱藏維度 128，SiLU 激活函數）；
- 損失權重設定：$\mathcal{L}_{recon}=1.0, \mathcal{L}_{perceptual}=1.0, \mathcal{L}_{discriminator}=0.5, \mathcal{L}_{irr}=0.1$。

### 3.2 潛空間 FAR (Frame AutoRegressive) 擴散預測
- 在凍結的 DCAEIrr 潛空間中執行流匹配（Flow-Matching）生成，預測未來 16 幀潛狀態；
- 推論時以自迴歸方式循環推展至 30 分鐘；
- 透過隨機噪聲採樣 $S = 4$ 條潛軌跡，擬合自由度 $\nu = S-1$ 的 Student-$t$ 分佈，直接解析導出任意置信區間（如 68% 與 95% PI）。

---

## 4. 關鍵實驗結果與基準比對 (Key Results & Benchmarks)

前 15 分鐘平均預測指標（對比 Scaled Persistence、端到端模型 E2E-TF、解耦基準 FAR、以及文獻基準 SkyGPT）：

### 4.1 雙測試集確定性表現（前 15 分鐘平均）
| 測試集 | 模型架構 | RMSE ($W/m^2$) ↓ | MAE ($W/m^2$) ↓ | MBE ($W/m^2$) [系統偏差] | 技能分數 FS (%) ↑ |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **PSA-Bench2019** | Scaled Persistence (SP) | 73.5 | 49.0 | 0.5 | 0.0% |
| (28 天公開基準) | SkyGPT (Nie et al., 2024) | 86.8 | 63.4 | +1.3 | -29.2% |
| | FAR (解耦兩階段級聯) | 69.7 | 49.1 | -34.9 | -0.5% |
| | E2E-TF (端到端 Transformer) | 57.7 | 32.1 | -4.0 | +20.9% |
| | **FarSky (本文方法)** | **50.0** | **25.8** | **+3.6** | **+32.0%** |
| **SVA-Test** | Scaled Persistence (SP) | 129.2 | 70.0 | 0.2 | 0.0% |
| (60 天獨立測試) | SkyGPT (Nie et al., 2024) | 146.7 | 108.2 | -20.9 | -21.4% |
| | FAR (解耦兩階段級聯) | 122.8 | 80.2 | -44.4 | +1.8% |
| | E2E-TF (端到端 Transformer) | 100.8 | 55.3 | -7.6 | +20.9% |
| | **FarSky (本文方法)** | **98.0** | **50.6** | **+15.8** | **+24.1%** |

### 4.2 機率評估與極端爬坡事件（Ramp Events）對比
| 測試集 | 模型 | 機率技能 CRPSS ↑ | 68% 區間評分 IS68 ↓ | 95% 區間評分 IS95 ↓ | 爬坡檢測 Precision ↑ | 爬坡檢測 Recall ↑ | **爬坡 F1-Score ↑** |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **PSA-Bench2019** | E2E-TF (端到端) | 0.677 | 130.3 | 266.6 | 0.340 | 0.002 | **0.004 (失效)** |
| | SkyGPT | 0.275 | 276.9 | 636.3 | 0.298 | 0.542 | 0.384 |
| | FAR (解耦) | 0.396 | 242.3 | 838.8 | 0.452 | 0.862 | 0.592 |
| | **FarSky** | **0.715** | **107.0** | **236.5** | **0.490** | **0.839** | **0.618** |
| **SVA-Test** | E2E-TF (端到端) | 0.532 | 233.6 | 517.2 | 0.757 | 0.004 | **0.021 (失效)** |
| | SkyGPT | -0.070 | 523.9 | 1626.0 | 0.372 | 0.502 | 0.427 |
| | FAR (解耦) | 0.213 | 385.0 | 1308.5 | 0.561 | 0.859 | 0.678 |
| | **FarSky** | **0.540** | **217.0** | **572.0** | **0.578** | **0.805** | **0.673** |

---

## 5. 深入剖析與局限性 (Critical Analysis & Limitations)

### 關鍵洞察：
1. **揭開端到端模型的高精度假象**：
   E2E-TF 的 RMSE 雖然不錯（57.7 $W/m^2$），但其在爬坡檢測上的 F1 幾乎為 0。這是因為純回歸以 MSE 為導向，被迫選擇輸出平滑均值，完全喪失了預警突變的能力。
2. **潛空間耦合消除介面負偏差**：
   解耦模型 FAR 的 MBE 高達 -34.9 至 -44.4 $W/m^2$（嚴重低估日照），而 FarSky 的 MBE 僅為 +3.6 $W/m^2$。這直接佐證了 [[2026-suwandee-single-stage-diffusion]] 的論斷：**傳統兩階段獨立訓練的回歸器對生成視訊存在介面級適應不良**，而 FarSky 透過「潛空間多任務感知」在特徵層面解決了此問題。

### 局限性：
1. **全陰天（Mostly Overcast）長期衰退**：在超過 15 分鐘的全陰天情況下，由於相機視野受限，擴散模型傾向回歸至訓練集中更常見的碎雲型態，產生輕微的高估正偏差。
2. **採樣次數受限**：受限於擴散推論時間，本研究僅採樣 $S = 4$ 條軌跡，導致 95% 寬區間的實證覆蓋率（PICP）略有不足（Undercoverage），需要藉助 Student-$t$ 分佈擬合。

---

## 6. 與其他文獻之脈絡關聯 (Connections to Other Work)

- **與先前兩階段架構的關聯**：
  - 超越 [[2024-nie-skygpt|SkyGPT]]：指出 SkyGPT 潛空間未考慮輻射任務的硬傷，在相同西班牙數據上各項指標大幅領先。
  - 對比 [[2026-suwandee-single-stage-diffusion|Suwandee et al. (2026)]]：Suwandee 透過單階段像素級聯合去噪消除兩階段問題；FarSky 則在**潛空間（Latent Space）**中透過多任務學習與直接潛解碼消除了回歸器接口問題。
- **涉及核心概念**：
  - [[farsky-latent-coupling|Task-Aware Latent-Space Coupling]]
  - [[psa-almeria-dataset|PSA Almería Benchmark]]
  - [[continuous-ranked-probability-score|CRPS]]、[[ramp-events|Ramp Events]]
