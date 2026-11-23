---
title: "A Generative Sky Image-Based Two-Stage Framework for Probabilistic Photovoltaic Power Forecasting"
authors: ["Chen Pan", "ChangGyoon Lim"]
year: 2025
venue: "Computer Modeling in Engineering & Sciences (CMES)"
arxiv: "https://doi.org/10.32604/cmes.2025.073389"
code: null
forecast_horizon: "15 min (ultra-short-term nowcasting)"
target_variable: "PV Power (kW)"
input_modalities: ["RGB Sky Image Sequences (Historical 15 mins)"]
method_type: "I-GPT (Multi-scale PhyCell + Dynamic Weight Fusion Module + Axial VQ-VAE) + IP-U-Net (Attention Gates)"
probabilistic: true
metrics:
  crps: 2.912 # Stanford test set
  rmse: null
  skill_score: 0.3589 # 35.89% over Smart Persistence
tags:
  - paper
  - sky-image
  - generative-model
  - probabilistic-forecasting
  - video-prediction
  - multi-scale-phycell
  - dynamic-weight-fusion
  - attention-gate
---

# A Generative Sky Image-Based Two-Stage Framework for Probabilistic Photovoltaic Power Forecasting

- **作者**：Chen Pan, ChangGyoon Lim (Department of Computer Engineering, Chonnam National University, Republic of Korea)
- **年份 / 發表場合**：2025 / *Computer Modeling in Engineering & Sciences* (CMES), Vol. 145, No. 3.
- **論文來源**：[DOI: 10.32604/cmes.2025.073389](https://doi.org/10.32604/cmes.2025.073389)
- **主要方法**：[[i-gpt|I-GPT (Multi-scale Physics-Constrained Video Predictor)]] + [[ip-unet|IP-U-Net (Attention-Gated Image-to-Power Regressor)]]
- **評測資料集**：[[stanford-rooftop-dataset|Stanford 30-kW Rooftop PV & Sky Image Dataset (SKIPP'D)]]

---

## 1. 核心摘要與主要貢獻 (Executive Summary & Contributions)

本論文針對先前 [[2024-nie-skygpt|SkyGPT]] 兩階段架構中的結構性瓶頸進行深度改良。作者指出：
1. **單尺度物理建模之不足**：雲層動態具有顯著的多尺度特性（如細長快速的雲絲 vs. 寬廣緩慢的中尺度雲團），SkyGPT 採用的單尺度 PhyCell 無法同時兼顧多種受風剪切與擴散尺度的雲層運動。
2. **物理與外觀簡單相加的缺陷**：SkyGPT 在潛空間中直接將 PhyCell 的物理嵌入與 Transformer 的外觀嵌入做「逐元素相加（Element-wise Addition）」，忽略了不同天氣擾動下兩者權重的非線性動態調節。
3. **功率回歸端背景噪聲干擾**：下游 U-Net 的 Skip Connection 傳遞了過多無關的天空背景與邊界噪聲，干擾對環日區（Circumsolar Area）關鍵雲遮的捕捉。

為此，作者提出 **I-GPT** 視訊生成模型搭配 **IP-U-Net** 功率回歸器的升級架構，成功應用於虛擬電廠（VPP）的 15 分鐘超短期機率調度。

### 主要學術貢獻：
1. **多尺度物理約束單元 (Multi-scale PhyCell)**：在潛空間構建具備不同卷積核尺寸（$3\times3, 5\times5, 7\times7$）與擴張率的多尺度濾波器組，同時受偏微分方程（PDE）矩約束（Moment Loss），精準捕捉不同尺度的雲體平流與擴散。
2. **動態加權融合模組 (Dynamic Weight Fusion Module, DWFM)**：採用雙路並行 3D 卷積，分別使用 GELU 提取豐富非線性特徵、Sigmoid 產生空間自適應門控權重，取代單純相加，實現外觀與物理特徵的數據驅動動態分配。
3. **軸向注意力 VQ-VAE (Axial Attention VQ-VAE)**：在自編碼器殘差塊中引入軸向注意力（Axial Attention），在大幅降低時空運算複雜度的同時強化長程時空特徵建模。
4. **注意力門控回歸器 (IP-U-Net)**：在跳躍連接處加入注意力門（Attention Gate, AG），壓制無關邊緣背景雜訊，集中注意力於太陽盤與高對比雲邊緣。
5. **超越 SkyGPT 基準**：在 Stanford 案場多雲測試集上達成 **CRPS 2.912 kW** 與 **WS 33.103 kW**，相較持久模型提升技能分數（FS）達 35.89%，全面優於同架構設定下的 SkyGPT（CRPS 3.183 kW, WS 36.186 kW）。

---

## 2. 問題定義與硬體配置 (Problem Formulation & Hardware Setup)

- **預測目標變數**：未來 15 分鐘（$t+15$）之光電輸出功率 $P(t+15)$。
- **預測時間尺度 (Forecast Horizon)**：超短期預報，$\Delta t = 15$ 分鐘。
  - 輸入：過去 15 分鐘歷史影像序列（$t-15:2:t-1$，共 8 幀）。
  - 生成：未來 15 分鐘影像序列（$t+1:2:t+15$，共 8 幀），取第 8 幀（$t+15$）輸入 IP-U-Net。
- **資料集與硬體設定**：
  - 資料集：史丹佛大學 30.1 kW 屋頂太陽能板與 125 公尺旁魚眼相機（與 [[2024-nie-skygpt|SkyGPT]] 相同案場與切分）。
  - 訓練樣本：53,336 組訓練樣本、4,467 組多雲驗證樣本、2,582 組跨期多雲測試樣本（2019 年 11~12 月）。
  - 運算環境：單張 NVIDIA GeForce RTX 4090 GPU (24 GB) + Intel Core i7-13700K。
  - 訓練成本：約 24 GPU 小時，推論峰值顯存約 6 GB，每組樣本生成 10 個情境並完成功率推論僅需約 0.5 秒。

---

## 3. 方法論與架構細節 (Methodology & Architecture)

```text
[歷史天空影像序列 (8 幀)] 
            │
            ▼
┌────────────────────────────────────────────────────────┐
│ Stage 1: I-GPT (Stochastic Video Predictor)            │
│ 1. 軸向注意力 VQ-VAE: 壓縮至離散碼本 C                  │
│ 2. 多尺度 PhyCell: 3x3, 5x5, 7x7 卷積核近似 PDE 矩約束 │
│ 3. 遮蔽自注意力 Transformer: 學習潛外觀自迴歸先驗      │
│ 4. 動態加權融合模組 (DWFM): GELU/Sigmoid 雙分支自適應融合│
└────────────────────────────────────────────────────────┘
            │ 採樣 Nf=10 組未來 t+15 影像
            ▼
┌────────────────────────────────────────────────────────┐
│ Stage 2: IP-U-Net (Image-to-Power Regressor)           │
│ - 2D Conv 編碼器 + 殘差瓶頸塊                           │
│ - 跨層 Skip Connections 配備 Attention Gates (AG)      │
│ - 抑制無效背景，輸出 64x64x1 空間特徵圖並加權求和      │
│ - 10-Fold 交叉驗證次模型                               │
└────────────────────────────────────────────────────────┘
            │ 10 張影像 × 10 個次模型 = 100 個點預測
            ▼
[未來 15 分鐘 PV 機率預測區間 (5%-95% 與 25%-75%)]
```

### 3.1 I-GPT 生成關鍵設計
1. **多尺度 PhyCell 矩損失**：
   $$\mathcal{L}_{moment} = \sum_{scale} \sum_{i \le k} \sum_{j \le k} \| M(w_{p,i,j}^k) - \Delta_{i,j}^k \|_F$$
   在多個尺度上約束空間導數近似熱傳導、波動與對流擴散方程，使細碎卷雲與大片積雨雲皆能服從流體物理。
2. **DWFM 動態融合公式**：
   $$O_1 = \text{GELU}(\text{Conv}_1(X_1)) \odot X_1$$
   $$O_2 = \text{Sigmoid}(\text{Conv}_2(X_1)) \odot X_1$$
   $$\text{Out}_{DWFM} = \text{Conv}_3(O_1 + O_2)$$

### 3.2 IP-U-Net 注意力門控 (Attention Gate, AG)
在 U-Net 解碼階段，引入門控信號調控編碼層傳來的特徵圖：
透過點卷積與 Sigmoid 輸出空間注意力係數 $\alpha \in [0, 1]$，使網路聚焦於日盤周邊光暈及迎風面雲邊緣，忽略邊緣無效外框。

---

## 4. 關鍵實驗結果與基準比對 (Key Results & Benchmarks)

測試集選取 2,582 筆跨期陰天樣本（嚴格評估陰天突變挑戰），比較指標包含 CRPS、Winkler Score (WS, 90% PI) 與技能分數 (FS)：

### 4.1 測試集（Test Set, 2,582 筆樣本）光電機率預報對比
| 模型名稱 | 模型架構特徵 | CRPS (kW) ↓ | WS (kW) ↓ | FS (%) ↑ | 預測特性評析 |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Smart Persistence (SPM)** | 確定性基準 | 4.542 | — | 0% | 嚴重落後 |
| **ACMCA (Pan et al., 2024)** | 歷史多模態 (影像+功率) | 2.961 | 45.231 | 34.81% | 點預測尚可，但區間過寬（WS 高） |
| **ConvLSTM $\to$ IP-U-Net** | 確定性時空 RNN | 3.400 | 51.054 | 25.15% | 畫面平滑模糊導致低估尖峰 |
| **PhyDNet $\to$ IP-U-Net** | 確定性單尺度物理 | 3.687 | 56.243 | 18.83% | 缺乏隨機多樣性 |
| **PhyDNet+GAN $\to$ IP-U-Net** | 對抗生成物理網絡 | 3.728 | 57.432 | 17.93% | 銳利但缺乏多樣性 |
| **VideoGPT $\to$ IP-U-Net** | 隨機離散視訊生成 | 3.090 | 36.830 | 31.97% | 隨機性強但偶有物理幻覺 |
| **SkyGPT $\to$ IP-U-Net** | 單尺度 PhyCell + 相加 | 3.183 | 36.186 | 29.93% | 泛化性受限於單尺度約束 |
| **I-GPT $\to$ IP-U-Net (本文方法)** | **多尺度 PhyCell + DWFM + AG** | **2.912** | **33.103** | **35.89%** | **純影像兩階段架構之最優表現** |
| *RSI $\to$ IP-U-Net (Oracle)* | *真實未來天空圖 (上限)* | *2.595* | *32.599* | *42.87%* | *生成模型完美時之理論極限* |

*(註：驗證集表現：I-GPT CRPS 達 2.073 kW, WS 達 22.202 kW, FS 達 42.78%)*

### 4.2 模組消融實驗 (Ablation Studies)
1. **移除多尺度 PhyCell**：15 分鐘影像預測 MSE 上升、VGG-CS 下降；下游預測 CRPS 由 2.912 惡化至 2.984 kW（WS 升至 33.417 kW）。
2. **移除 DWFM 改為直接相加**：CRPS 惡化至 2.984 kW，驗證了動態加權非線性融合的必要性。
3. **移除 IP-U-Net 的 Attention Gate (AG)**：測試集 CRPS 由 2.912 退化至 2.984 kW，WS 由 33.103 惡化至 33.417 kW（驗證集 WS 惡化幅度達 0.75 kW）。

---

## 5. 深入剖析與局限性 (Critical Analysis & Limitations)

### 優勢剖析：
1. **堅持純影像兩階段的泛化彈性**：作者刻意不將歷史發電功率餵給模型，是為了避免模型在晴天轉陰天突變時過擬合歷史時序趨勢（避免 Phase Lag），在劇烈 Ramp Events 中更能由雲動態驅動提前響應。
2. **運算與工程實用性極佳**：相較於 Diffusion 需要百步迭代，I-GPT 在 RTX 4090 上單次推論僅需 0.5 秒，峰值顯存 6 GB，完全符合虛擬電廠（VPP）即時通訊調度協議。

### 局限性與觀點衝突：
1. **與多模態流派的衝突**：[[2025-xiong-multimodal-diff-transformer|Xiong et al. (2025)]] 證明在同一測試集上，若加入歷史發電數據作為多模態輸入，CRPS 可進一步降至 2.44 kW。I-GPT 的 2.912 kW 雖優於 SkyGPT，但在絕對數值上仍遜於多模態融合方案。
2. **下游映射誤差依然主導**：論文第 6.3 節明確承認，即使輸入 100% 正確的真實未來影像（RSI），CRPS 仍有 2.595 kW，殘差不確定性主要仍停留在「影像映射到功率」的環節。

---

## 6. 與其他文獻之脈絡關聯 (Connections to Other Work)

- **繼承與改良對象**：
  - 直接針對 [[2024-nie-skygpt|SkyGPT (Nie et al., 2024)]] 提出三大升級（單尺度 $\to$ 多尺度 PhyCell；簡單相加 $\to$ DWFM；標準 U-Net $\to$ Attention-Gated IP-U-Net）。
- **與同時代文獻之對比**：
  - 對比 [[2025-xiong-multimodal-diff-transformer|Xiong et al. (2025)]]：兩者同期發表於 2025 年。Xiong 等人走向「Diffusion + 歷史功率多模態 + GPA」路線，精度更高（CRPS 2.44 kW）；Pan & Lim 則堅持「自迴歸 Transformer + 純影像抗延遲 + 輕量級推論（0.5s）」，工程實時性更佳。
- **涉及核心概念**：
  - [[continuous-ranked-probability-score|CRPS]]、[[winkler-score|Winkler Score]]、[[stanford-rooftop-dataset|Stanford Rooftop Dataset]]。
