---
title: "PV-MM-diffusion: An end-to-end multi-modal diffusion model for ultra-short-term probabilistic photovoltaic forecasting"
authors: ["Jing Huang", "Bopeng Shao", "Yan Ke", "Yuan Gao", "Stefano Mazzoni"]
year: 2026
venue: "Applied Energy (Vol. 413, 127816)"
arxiv: "https://doi.org/10.1016/j.apenergy.2026.127816"
code: null
forecast_horizon: "15 min (8 steps at 2-min cadence)"
target_variable: "PV Power Output (kW)"
input_modalities: ["RGB Sky Images (8 historical frames)", "Historical PV Power (8 past readings)"]
method_type: "End-to-End Multimodal Diffusion / Coupled U-Net / RS-PV-MMA / Empty-Frame Placeholder"
probabilistic: true
metrics:
  crps: 2.63 # kW on Stanford 30-kW test set
  rmse: null
  skill_score: null # WS: 21.46 kW (40% better than SkyGPT)
tags:
  - paper
  - sky-image
  - generative-model
  - probabilistic-forecasting
  - diffusion-model
  - end-to-end-multimodal
  - stanford-dataset
  - coupled-unet
---

# PV-MM-diffusion: An end-to-end multi-modal diffusion model for ultra-short-term probabilistic photovoltaic forecasting

- **作者**：Jing Huang, Bopeng Shao, Yan Ke, Yuan Gao, Stefano Mazzoni (Zhejiang Gongshang University, YQ Future Energy, Kyushu University, University of Roma Tor Vergata)
- **年份 / 發表場合**：2026 / *Applied Energy* (Vol. 413, 127816)
- **論文來源**：[DOI: 10.1016/j.apenergy.2026.127816](https://doi.org/10.1016/j.apenergy.2026.127816)
- **主要方法**：[[pv-mm-diffusion|PV-MM-Diffusion (Coupled U-Net with RS-PV-MMA)]] + [[empty-frame-placeholder|Empty-Frame Placeholder Strategy]]
- **評測資料集**：[[stanford-rooftop-dataset|Stanford 30-kW Rooftop PV Dataset (SKIPP'D)]]

---

## 1. 核心摘要與主要貢獻 (Executive Summary & Contributions)

本論文針對傳統光電預測中「將視覺資料與數值時序割裂處理」的痛點，提出了全球首套**端到端多模態擴散預測模型 (PV-MM-Diffusion)**。
過去文獻要嘛是「兩階段級聯（先生成視訊再回歸功率）」，要嘛是單純將歷史影像嵌入時序回歸模型，無法同時建模雲層連續演變與光電功率波動的協同不確定性。

PV-MM-Diffusion 透過一個高度耦合的 **Coupled U-Net**，在**同一個去噪軌跡**中同時對「未來 15 分鐘天空影像序列」與「未來 15 分鐘發電功率序列」進行聯合去噪生成，實現跨模態特徵的步步交互與不確定性傳遞。

### 主要學術貢獻：
1. **端到端多模態擴散統一生成**：打破兩階段框架，在單一擴散過程中同時生成未來天空影像與對應的光電功率分佈，雲層運動的模糊性與功率突變的偶發不確定性得以在同一步長中同步解析。
2. **隨機位移多模態注意力 (RS-PV-MMA)**：
   針對高維影像（$L \times H \times W$）與一維功率（$L$）全局注意力複雜度過高的問題，設計局部時間窗口 $L_w$ 搭配隨機位移偏移量 $R$ 的局部交叉注意力機制，在大幅壓低計算開銷的同時精準對齊雲遮邊緣與發電階躍。
3. **空幀佔位策略 (Empty-Frame Placeholder Strategy)**：
   設計混合模式訓練損失，使單一擴散網路無須修改結構即可靈活支援「影像+功率 $\to$ 影像+功率（全模態）」、「純影像 $\to$ 影像+功率」以及「純功率 $\to$ 功率（相機離線容災模式）」三種運行型態。
4. **刷新 Stanford 案場紀錄**：在 Stanford 30-kW 屋頂光電多雲測試集上，達成 **CRPS 2.63 kW**（較 SkyGPT 改善 18%）與 **Winkler Score (WS) 21.46 kW**（較 SkyGPT 改善 40%）。其 Winkler Score 甚至顯著超越了輸入真實未來影像的理論上限（Real Future SI $\to$ U-Net 之 WS 為 30.72 kW），展現出極為緊緻可靠的區間包絡能力。

---

## 2. 問題定義與硬體配置 (Problem Formulation & Hardware Setup)

- **預測目標變數**：未來 15 分鐘（8 個時步，步長 2 分鐘）的光電輸出功率序列 $P^{8:15}$ 與天空影像序列 $S^{8:15}$ 的聯合條件分佈：
  $$(\hat{S}^{8:15}, \hat{P}^{8:15}) \sim p_\theta(S^{8:15}, P^{8:15} \mid S^{0:7}, P^{0:7})$$
- **預測時間尺度 (Forecast Horizon)**：15 分鐘超短期臨近預報。
  - 歷史輸入：過去 16 分鐘（$t-15 \dots t-1$ 步長 2 分鐘，8 幀）天空圖與發電量。
  - 未來生成：未來 15 分鐘（$t+1 \dots t+15$ 步長 2 分鐘，8 步）同步時序與影像。
- **資料集 (Stanford SKIPP'D Dataset)**：
  - 史丹佛大學 30 kW 屋頂太陽能板與海康威視魚眼鏡頭（2017–2019）。
  - 訓練集：53,336 組樣本；驗證集：4,467 組（10 個多雲日）；測試集：2,582 組（跨期 5 個多雲日）。
- **運算與推論成本**：
  - 訓練環境：8 張 NVIDIA RTX 3090 GPU 耗時 43.06 小時（100,000 步）。
  - 推論採樣：採用 **DPM-Solver++** 加速採樣（60 步），在單張 RTX 3090 上生成 10 個採樣僅需 **45.38 秒**（在 15 分鐘滾動窗口內完全滿足即時調度需求）。

---

## 3. 方法論與架構細節 (Methodology & Architecture)

整體架構由雙分支 Coupled U-Net 驅動：

```text
[歷史天空影像 S^{0:7}] (固定條件)       [歷史發電功率 P^{0:7}] (固定條件)
[高斯噪聲未來影像 S_T^{8:15}]             [高斯噪聲未來功率 P_T^{8:15}]
          │                                         │
          ▼                                         ▼
┌────────────────────────────────────────────────────────┐
│  Coupled U-Net 反向去噪 (DPM-Solver++ 60 步, \tau=T -> 1)│
│                                                        │
│  【天空影像分支】                    【光電功率分支】       │
│  - 2D 空間卷積 + 1D 時間卷積        - 1D 卷積 + 時間自注意力│
│  - 空間 2D 注意力 + 時間 1D 注意力   - 時間軸不降採樣       │
│             ▲                                ▲         │
│             └─── 雙向 RS-PV-MMA 交叉注意力 ───┘         │
│                  (隨機位移局部窗口特徵對齊)             │
└────────────────────────────────────────────────────────┘
          │                                         │
          ▼                                         ▼
[未來 8 幀生成天空影像 S_0^{8:15}]       [未來 8 步 PV 功率分佈 P_0^{8:15}]
```

### 3.1 隨機位移多模態注意力 (RS-PV-MMA)
- 影像特徵塊尺寸為 $[L \times H_l \times W_l]$，PV 特徵序列尺寸為 $[C_l \times L]$；
- 定義時間窗口 $L_w$ 並採樣隨機偏移量 $R \in [0, L - L_w]$：
  $$f_s = (i + R) \bmod L, \quad f_e = (i + R + L_w) \bmod L$$
  僅在長度為 $L_w$ 的局部時間窗口內計算雙向 Cross-Attention，將計算複雜度從 $O((L \cdot H \cdot W) \cdot L)$ 壓縮至 $O((L_w \cdot H \cdot W) \cdot L_w)$。

### 3.2 空幀佔位訓練策略 (Empty-Frame Placeholder)
為使單一模型兼具容災能力，訓練時依機率隨機切換模式並注入零張量佔位符：
$$\mathcal{L}_{\theta_{PS}} = \mathbb{E}\left[ w_{SP} \mathcal{L}_{SP} + w_{SI} \mathcal{L}_{SI} + w_{PV} \mathcal{L}_{PV} \right]$$
其中 $w_{SP} = w_{SI} = w_{PV} = \frac{1}{3}$，訓練出對「相機離線」或「功率感測器失效」皆具強大魯棒性的通用模型。

---

## 4. 關鍵實驗結果與基準比對 (Key Results & Benchmarks)

### 4.1 史丹佛測試集（2,582 筆樣本，15 分鐘預報）對比
| 模型名稱 | 模型架構類型 | CRPS (kW) ↓ | FS (CRPS vs. SkyGPT) ↑ | Winkler Score (kW) ↓ | FS (WS vs. SkyGPT) ↑ | 表現評語 |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Smart Persistence (SPM)** | 確定性基準 | 3.67 | -14% | — | — | 無機率估計能力 |
| **SUNSET (CNN)** | 傳統端到端深度學習 | 3.28 | -2% | 56.29 | -58% | 嚴重的爬坡時滯，區間極窄 |
| **MC Dropout SUNSET** | 貝葉斯近似回歸 | 3.10 | +3% | 43.68 | -23% | 改善不確定性，但依然存在時滯 |
| **QR-GRU** | 純功率分位數回歸 | 2.95 | +8% | 33.38 | +6% | 無影像資訊，靠時序統計包絡 |
| **ConvLSTM $\to$ U-Net** | 確定性視訊兩階段 | 3.12 | +3% | 46.33 | -30% | 視訊模糊，兩階段傳遞失真 |
| **SkyGPT $\to$ U-Net** | 隨機視訊兩階段級聯 | 3.21 | 0% | 35.62 | 0% | 兩階段經典基準 |
| *Real Future SI $\to$ U-Net* | *真實未來天空圖 (上限)* | *2.52* | *+21%* | *30.72* | *+14%* | *兩階段架構在完美生成下的上限* |
| **PV-MM-Diffusion (PV $\to$ PV)** | 擴散模型 (純功率模式) | 3.02 | +6% | 35.86 | -1% | 相機離線時之後備模式 |
| **PV-MM-Diffusion (SI $\to$ SI PV)** | 擴散模型 (純影像輸入) | 3.09 | +4% | 24.99 | +30% | 無日照計時之表現，WS 依然優異 |
| **PV-MM-Diffusion (SI PV $\to$ SI PV)** | **端到端多模態聯合擴散** | **2.63** | **+18%** | **21.46** | **+40%** | **全場最優**：WS 甚至大幅超越理論上限！ |

### 4.2 核心發現：打破「真實影像理論上限」的神話
- 過去兩階段文獻（如 SkyGPT 與 I-GPT）普遍認為 Real Future SI $\to$ U-Net 是不可逾越的「理論極限（Oracle）」；
- 但在 Table 3 中，**PV-MM-Diffusion 的 Winkler Score 達到 21.46 kW，比輸入真實未來影像的 30.72 kW 還大幅降低了 30%**！
- **原因**：即使使用真實未來影像，U-Net 進行的是確定性單點映射，其經驗分佈由次模型硬湊；而 PV-MM-Diffusion 的擴散去噪是連續平滑的機率分佈建模，預測區間的銳利度與可靠度遠勝確定性回歸。

---

## 5. 深入剖析與局限性 (Critical Analysis & Limitations)

### 優勢剖析：
1. **解決模態割裂**：過去視訊生成與功率回歸彼此不知道對方的目標；PV-MM-Diffusion 在每個去噪步 $\tau$ 讓雲層運動特徵與功率階躍特徵直接藉由注意力交乘，物理一致性極佳。
2. **多模式容災**：支援相機故障時退回純時序、日射計缺失時純靠相機，實用性極高。

### 局限性與與後續文獻的關聯：
1. **封閉梯度循環 (Closed Loop)**：
   後續文獻 [[2026-suwandee-single-stage-diffusion|Suwandee et al. (2026)]] 指出，PV-MM-Diffusion 在反向每一步讓帶噪的影像與帶噪的功率完全雙向 cross-attend，這種 closed gradient loop 在高噪聲初期（$\tau \approx 1000$）容易將無效的噪聲當作資訊傳遞。Suwandee 提出改用 $\hat{x}_0$ 截斷估計（Open Loop）進行了針對性改進。

---

## 6. 與其他文獻之脈絡關聯 (Connections to Other Work)

- **繼承與超越**：
  - 直接針對 [[2024-nie-skygpt|SkyGPT]] 與傳統兩階段提出全面超越，在同一 Stanford 案場上達成 CRPS 2.63 kW 與 WS 21.46 kW。
- **與同時代/後續文獻對話**：
  - 與 [[2025-xiong-multimodal-diff-transformer|Xiong et al. (2025)]]：兩者皆主張結合歷史功率與影像，但 Xiong 採用 Neural SDE 兩階段 + GPA，而 PV-MM-Diffusion 採用端到端聯合 Coupled U-Net。
  - 被 [[2026-suwandee-single-stage-diffusion|Suwandee et al. (2026)]] 作為代表性基線深入剖析其去噪狀態耦合機制。
