---
title: "Multimodal ultra-short-term probabilistic solar power forecasting with generative AI and Transformer"
authors: ["Binyu Xiong", "Yuntian Chen", "Xin Zhao", "Zhongyue Su", "Jun Fu", "Dali Chen", "Dongxiao Zhang"]
year: 2025
venue: "Advances in Applied Energy"
arxiv: "https://doi.org/10.1016/j.adapen.2025.100250"
code: null
forecast_horizon: "15 min, 30 min, 60 min (ultra-short-term)"
target_variable: "PV Power (kW)"
input_modalities: ["RGB Sky Images", "Historical Solar Power Generation", "Temporal Features"]
method_type: "Diffusion Model / Neural SDE / Multimodal Transformer / Gaussian Process Approximation (GPA)"
probabilistic: true
metrics:
  crps: 4.07 # Kuitun 15m; on Stanford dataset: 2.44
  rmse: 7.80 # Kuitun 15m
  skill_score: null
tags:
  - paper
  - sky-image
  - generative-model
  - probabilistic-forecasting
  - diffusion-model
  - multimodal-learning
  - neural-sde
  - gaussian-process
---

# Multimodal ultra-short-term probabilistic solar power forecasting with generative AI and Transformer

- **作者**：Binyu Xiong, Yuntian Chen, Xin Zhao, Zhongyue Su, Jun Fu, Dali Chen, Dongxiao Zhang (Northeastern University, Eastern Institute of Technology Ningbo)
- **年份 / 發表場合**：2025 / *Advances in Applied Energy* (Vol. 20, 100250)
- **論文來源**：[DOI: 10.1016/j.adapen.2025.100250](https://doi.org/10.1016/j.adapen.2025.100250)
- **主要方法**：[[neural-sde-diffusion-video|Neural SDE + Conditional Diffusion Video Predictor]] + [[gaussian-process-approximation|Multimodal Transformer with GPA]]
- **評測資料集**：[[kuitun-dataset|Kuitun Solar & Sky Image Dataset]]、[[stanford-rooftop-dataset|Stanford Rooftop Dataset (SKIPP'D)]]

---

## 1. 核心摘要與主要貢獻 (Executive Summary & Contributions)

本論文針對現有天空影像超短期光電機率預報的兩大核心盲點提出根本性改進：
1. **單一模態局限**：先前如 [[2024-nie-skygpt|SkyGPT]] 僅仰賴預測的天空影像推斷光電輸出。然而研究證實，**兩張視覺相似的天空影像可能因相機視角、大氣氣溶膠及系統運作狀態而對應截然不同的發電功率**，必須納入歷史發電時序。
2. **不確定性建模不完整**：現有方法多只考慮雲層動態的偶發不確定性（Aleatoric Uncertainty），完全忽略了下游回歸模型本身的認知不確定性（Epistemic Model Uncertainty），導致在平穩晴空或陰雨天時嚴重低估不確定區間。

為此，作者提出了一套**結合生成式擴散模型（Diffusion）、神經隨機微分方程（Neural SDE）、多模態 Transformer 與高斯過程近似（GPA）的雙重不確定性量化預測架構**。

### 主要學術貢獻：
1. **Neural SDE + 條件擴散視訊預測**：採用 Conv-GRU 提取時空特徵，以神經隨機微分方程（Neural SDE）建模未來運動特徵漂移，再由條件擴散模型（CDM）生成高保真度未來天空影像，在消融實驗中顯著優於 SkyGPT 的 VideoGPT 生成質量。
2. **多模態特徵早期融合 Transformer**：設計早期拼接（Early-concatenation）機制，透過 ResNet 提取未來影像特徵與歷史功率時序共同送入 Transformer Encoder-Decoder，使影像能直接調整時序趨勢。
3. **高斯過程近似 (GPA) 與雙重不確定性融合**：在多模態 Transformer 輸出端引入譜歸一化（Spectral Normalization）與隨機傅立葉特徵擴展（RFF），結合全方差公式（Law of Total Variance）同時整合「未來天空動態隨機性」與「多模態神經網路模型不確定性」。
4. **全面超越先前 SOTA**：在全新新疆奎屯（Kuitun）資料集上達成領先；在同一個 Stanford 開源基準集上，將 15 分鐘預報 CRPS 從 SkyGPT 的 2.81 kW 進一步壓低至 **2.44 kW**（提升 13.2%），Winkler Score (WS) 從 26.72 kW 降至 **19.62 kW**（改善 26.6%）。

---

## 2. 問題定義與硬體配置 (Problem Formulation & Hardware Setup)

- **預測目標變數**：未來超短期光電輸出功率 $\mathbf{s}_{t_0:T} = [s_{t_0}, s_{t_0+1}, \dots, s_T]$ 的條件機率分佈 $P(\mathbf{s}_{t_0:T} | \mathbf{s}_{1:t_0-1}, \mathbf{w}_{1:t_0-1}, \mathbf{u}_{1:T})$。
- **預測時間尺度 (Forecast Horizons)**：
  - 15 分鐘預報（15-min）：輸入過去 15 分鐘影像與功率（採樣頻率 2 分鐘，8 幀）。
  - 30 分鐘預報（30-min）：輸入過去 15 分鐘資料（採樣頻率 2 分鐘）。
  - 60 分鐘預報（60-min）：輸入過去 30 分鐘資料（採樣頻率 5 分鐘）。
- **全天空相機與資料規格 (Kuitun Dataset)**：
  - 地點：中國新疆奎屯市（Kuitun City, Xinjiang, China）。
  - 時間範圍：2024 年 3 月 1 日至 7 月 31 日，每日 06:00 ~ 22:00，共 112,599 幀。
  - 解析度：原始 $2560 \times 1920$ 裁剪去除無效黑邊並降採樣至 $128 \times 128$。
  - 天氣劃分：依據主要天氣型態嚴格劃分為 Sunny、Cloudy、Overcast、Rainy 4 大類別後分配訓練/驗證/測試集，防止資料分佈偏移（OOD）。

---

## 3. 方法論與架構細節 (Methodology & Architecture)

整體框架可分為視訊生成、多模態融合與不確定性聚合三大模組：

```text
[歷史天空影像 w(1:t0-1)] 
         │ (差分特徵)
         ▼
┌────────────────────────────────────────────────────────┐
│ 模組 1: 隨機視訊預測 (Conv-GRU + Neural SDE + CDM)       │
│ - Conv-GRU: 編碼雲層時空運動特徵                         │
│ - Neural SDE: dX = a(X,t)dt + b(X,t)dE_t (連續運動漂移) │
│ - 條件擴散模型 (CDM): 逐步去噪生成 Nf=10 組未來影像序列  │
└────────────────────────────────────────────────────────┘
         │ 輸出 Nf=10 組預測影像序列 W^i(t0:T)
         ▼
┌────────────────────────────────────────────────────────┐
│ 模組 2: 多模態 Transformer + 高斯過程近似 (GPA)        │
│ - 影像端: ResNet (加入高斯噪聲/模糊預訓練) 提取空間特徵   │
│ - 功率端: 數值 Embedding + 週期時間編碼                 │
│ - 結構: 早期拼接 (Early-Concat) -> Self & Cross Attention│
│ - 輸出層: 隨機傅立葉映射輸出均值與協方差 (μ_mi, σ_mi^2) │
└────────────────────────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────────────────────┐
│ 模組 3: 雙重不確定性全方差聚合 (Total Variance Law)      │
│   \hat{μ}_d = (1/N) \sum \hat{μ}_mi                    │
│   \hat{σ}_d^2 = (1/N) \sum (\hat{σ}_mi^2 + \hat{μ}_mi^2) - \hat{μ}_d^2 │
└────────────────────────────────────────────────────────┘
         │
         ▼
[最終輸出：兼具雲動態多樣性與模型認知不確定性的機率預報]
```

### 3.1 隨機視訊生成：Neural SDE + 條件擴散模型 (CDM)
1. **差分特徵提取**：計算相鄰幀差分矩陣，經 CNN 與 GRU 單元遞歸編碼為動態特徵 $m_{t_0-1}$。
2. **神經隨機微分方程 (Neural SDE)**：
   $$d X_t = a(X_t, t)dt + b(X_t, t) dE_t, \quad a, b = f_\theta(m_{t_{i-1}})$$
   其中 $E_t$ 為維納過程（Wiener Process），連續建模雲層在外力擾動下的隨機傳播。
3. **條件擴散去噪 (CDM)**：
   以 SDE 預測之動態特徵 $m_t$ 為引導條件，U-Net 預測每一步去噪高斯噪聲 $\epsilon$，生成微觀結構銳利的未來天空幀。

### 3.2 多模態 Transformer 與 GPA 不確定性估計
- **ResNet 特徵抽取**：預先對影像注入高斯噪聲與模糊進行預訓練，模擬生成影像瑕疵，並對權重執行**譜歸一化（Spectral Normalization）**以滿足 Lipschitz 連續性條件。
- **高斯過程近似 (GPA)**：取代傳統神經網路 Dense 輸出層，利用 Laplace 近似與隨機傅立葉特徵擴展（RFF）直接估計預測均值 $\hat{\mu}_m$ 與模型認知協方差 $\hat{\sigma}_m^2$。

---

## 4. 關鍵實驗結果與基準比對 (Key Results & Benchmarks)

### 4.1 奎屯資料集（Kuitun Dataset）整體對比
評估指標包含確定性指標（RMSE, MAE, SMAPE）與機率指標（[[continuous-ranked-probability-score|CRPS]], MWIS 95% 區間）：

| 預測時程 | 評估指標 | 純時序模型 (Time Series) | 純影像模型 (Image Only) | 視訊預測模型 (Video Pred) | 傳統多模態 (Multimodal) | **本論文方法 (Ours)** |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **15-min** | RMSE (kW) ↓ | 9.82 | 14.79 | 14.51 | 8.85 | **7.80** (-20.6% vs TS) |
| | MAE (kW) ↓ | 6.59 | 11.09 | 11.03 | 5.65 | **4.96** (-24.7% vs TS) |
| | **CRPS (kW) ↓** | 5.05 | 8.35 | 9.89 | 4.45 | **4.07** (-19.4% vs TS) |
| | **MWIS (kW) ↓** | 68.01 | 100.76 | 295.20 | 64.84 | **48.22** (-29.1% vs TS) |
| **30-min** | RMSE (kW) ↓ | 11.82 | 15.00 | 14.56 | 11.66 | **10.28** |
| | **CRPS (kW) ↓** | 6.30 | 8.50 | 10.04 | 6.11 | **5.46** |
| | **MWIS (kW) ↓** | 78.89 | 103.59 | 298.87 | 74.51 | **59.07** |
| **60-min** | RMSE (kW) ↓ | 14.80 | 16.51 | 15.71 | 13.64 | **12.83** |
| | **CRPS (kW) ↓** | 7.96 | 9.42 | 10.83 | 7.60 | **6.90** |
| | **MWIS (kW) ↓** | 99.33 | 107.79 | 302.36 | 89.80 | **74.11** |

### 4.2 史丹佛資料集（Stanford SKIPP'D）直接橫向對比（見論文 Table C.1）
在與 [[2024-nie-skygpt|SkyGPT]] 相同的測試集與設定下：
- **SkyGPT-UNet**：CRPS = 2.81 kW, WS = 26.72 kW
- **本論文方法 (Ours)**：**CRPS = 2.44 kW**（降低 13.2%）, **WS = 19.62 kW**（降低 26.6%）
- 證明引入歷史功率多模態輸入與雙重不確定性後，能跨站點顯著超越 SkyGPT！

### 4.3 消融實驗分析 (Ablation Study, Table 4)
1. **視訊生成模組替換為 SkyGPT**：15 分鐘 CRPS 從 4.07 退化至 4.80（+15.2%），證明 Neural SDE + Diffusion 生成之未來天空影像質量與物理合理性優於 VideoGPT。
2. **多模態特徵改為後拼接 (Cross-modal Attention)**：15 分鐘 MAE 增加 21.4%，證明先拼接後送入自注意力機制更具表達力。
3. **影像骨幹改為 ViT**：RMSE 增加 4.6%，因天空影像資料量不足以支撐未經大尺度預訓練的 Vision Transformer。

---

## 5. 深入剖析與局限性 (Critical Analysis & Limitations)

### 優勢剖析：
1. **多模態互補優勢**：在晴天以歷史發電趨勢為主；在陰天與雨天雲遮劇烈時，視訊生成影像權重自發提高，成功避免了純影像法的漂移與純時序法的時延。
2. **區間自適應特性**：晴天置信區間緊緻收斂，多雲與陣雨時區間自發擴大；相較傳統視訊法在陰天嚴重低估不確定性（Underestimation），本文區間覆蓋率最優。

### 失效案例與瓶頸：
1. **擴散模型推論耗時巨大**：條件擴散去噪需要多步採樣，模型規模龐大，導致在 60 分鐘預報時受限於算力被迫將採樣時間間隔放大至 5 分鐘，限制了實時邊緣端部署。
2. **純數據驅動，缺乏顯式物理方程約束**：不同於 SkyGPT 明確使用 Navier-Stokes/線性 PDE 矩陣約束，本篇 Diffusion 依賴純數據分佈學習，偶爾在極度罕見極端天氣下缺乏顯式物理邊界約束。

---

## 6. 與其他文獻之脈絡關聯 (Connections to Other Work)

- **繼承與超越**：
  - 直接針對 [[2024-nie-skygpt|SkyGPT (Nie et al., 2024)]] 的兩大缺陷進行突破（補足歷史功率多模態、補足模型認知不確定性），並在同一個 Stanford 案場上達成新 SOTA。
- **關聯核心方法與概念**：
  - [[neural-sde-diffusion-video|Neural SDE + Diffusion Video Predictor]]
  - [[gaussian-process-approximation|Gaussian Process Approximation (GPA)]]
  - [[continuous-ranked-probability-score|Continuous Ranked Probability Score (CRPS)]]
  - [[winkler-score|Winkler Score / MWIS]]
  - [[stanford-rooftop-dataset|Stanford Rooftop Dataset]]
  - [[kuitun-dataset|Kuitun Dataset]]
