---
title: "Single-stage joint diffusion of sky images and clear-sky index for ultra-short-term irradiance forecasting: what the frame-to-irradiance stage discards"
authors: ["Kanawut Suwandee", "Suwichaya Suwanwimolkul"]
year: 2026
venue: "arXiv preprint (arXiv:2610.21795v2 [cs.CV]) / Submitted to Elsevier"
arxiv: "https://arxiv.org/abs/2610.21795"
code: "https://github.com/GenAI-CUEE/single-stage-joint-diffusion"
forecast_horizon: "16 min (8 frames at 2-min cadence)"
target_variable: "Global Horizontal Irradiance (GHI) / Clear-Sky Index (k)"
input_modalities: ["RGB Sky Images (8 historical frames)", "Deterministic Time Covariates (hour, minute, day, I_clr)"]
method_type: "Single-Stage Joint Diffusion / Stable Diffusion 1.5 Backbone / ControlNet / CSI Head with Denoised Estimate Conditioning"
probabilistic: true
metrics:
  crps: 36.9 # W/m^2 on SIRTA
  rmse: 104.1 # W/m^2 on SIRTA
  skill_score: 0.341 # CRPSS against Smart Persistence
tags:
  - paper
  - sky-image
  - generative-model
  - probabilistic-forecasting
  - diffusion-model
  - single-stage-diffusion
  - sirta-dataset
  - clear-sky-index
---

# Single-stage joint diffusion of sky images and clear-sky index for ultra-short-term irradiance forecasting: what the frame-to-irradiance stage discards

- **作者**：Kanawut Suwandee, Suwichaya Suwanwimolkul (Chulalongkorn University, Thailand)
- **年份 / 發表場合**：2026 / *arXiv:2610.21795v2* (Preprint submitted to Elsevier)
- **論文來源**：[arXiv:2610.21795](https://arxiv.org/abs/2610.21795) | **程式碼**：[GitHub - single-stage-joint-diffusion](https://github.com/GenAI-CUEE/single-stage-joint-diffusion)
- **主要方法**：[[single-stage-joint-diffusion|Single-Stage Joint Diffusion]] + [[denoised-estimate-conditioning|Denoised Estimate Conditioning ($\hat{x}_0$)]]
- **評測資料集**：[[sirta-dataset|SIRTA Atmospheric Observatory Dataset (France)]]

---

## 1. 核心摘要與主要貢獻 (Executive Summary & Contributions)

本論文對近年來主導天空影像生成式預測的**「兩階段級聯架構（Two-Stage Cascade）」**（如 [[2024-nie-skygpt|SkyGPT]]、[[2025-xiong-multimodal-diff-transformer|Xiong et al.]]、[[2025-pan-igpt-two-stage|I-GPT]]）提出了極具震撼力的批判與範式革新。

作者指出：兩階段設計（先預測未來天空高維視訊，再送入獨立回歸器計算日照）存在根本性缺陷：
1. **複合誤差累積 (Compounded Error Propagation)**：將高維影像生成作為中間媒介，下游回歸器會同時繼承視訊生成失真與影像至功率映射誤差。
2. **分佈偏移懲罰 (OOD Penalty on Regressors)**：獨立回歸器（如 U-Net）是在真實清晰影像上訓練的，但推論時讀取的卻是生成模型帶有微小形變的合成影像。論文實驗證明：**將本模型生成的影像餵給獨立訓練的 U-Net 回歸器，RMSE 高達 132.2 $W/m^2$；但改由聯合訓練的日照頭讀取，RMSE 驟降至 104.1 $W/m^2$——兩階段割裂本身就平白浪費了 28 $W/m^2$ 的精度！**

為此，作者提出了**「單階段聯合擴散模型（Single-Stage Joint Diffusion）」**：在一個統一的反向去噪過程中，同步對「未來天空影像潛變量」與「未來晴空指數（Clear-Sky Index, CSI）時序」進行協同去噪，並設計讓日照預測頭讀取影像分支的**「去噪淨值估計（Denoised Estimate $\hat{x}_0$）」**，徹底拋棄獨立的下游回歸器。

### 主要學術貢獻：
1. **首創單階段日照與天空影像聯合擴散**：在 Stable Diffusion 1.5 + ControlNet 骨幹上，同時去噪未來 8 幀影像與未來 8 步 CSI 序列（覆蓋 16 分鐘），完全免除 Stage 2 回歸器。
2. **去噪淨值引導機制 (Denoised Estimate Conditioning)**：在擴散反向每一步 $t$ 中，利用閉式解 $\hat{x}_0 = \frac{x_t - \sqrt{1-\bar{\alpha}_t}\text{sg}[\epsilon_x]}{\sqrt{\bar{\alpha}_t}}$ 推算當前估計的無噪未來天空，日照預測頭直接讀取 $\hat{x}_0$ 而非充滿隨機噪聲的 $x_t$。消融實驗證實，相比讀取噪聲狀態，讀取 $\hat{x}_0$ 使 RMSE 降低 9.3 $W/m^2$。
3. **零歷史實測日照輸入（Zero-Irradiance Input）**：模型輸入僅需歷史天空影像與時間特徵（無須日射計歷史讀數），非常適合僅裝設相機而無日射儀的偏遠場站。
4. **刷新 SIRTA 機率預測 SOTA**：在法國 SIRTA 觀測站 2023 年測試集上，達成 **CRPS 36.9 $W/m^2$**、**CRPSS +0.341**（相較持久模型），全面碾壓 12 種基線（包括 SkyGPT, Xiong et al., ConvLSTM, PhyDNet, DLinear, PatchTST 等）。

---

## 2. 問題定義與硬體配置 (Problem Formulation & Hardware Setup)

- **預測目標變數**：未來 16 分鐘內 8 個時步（步長 2 分鐘）的水平面總日照強度 $GHI(t+1 \dots t+16)$。
  - 透過晴空指數 $k = GHI / \max(I_{clr}, 1)$ 進行標準化去噪預測，再經由 Ineichen-Perez 晴空模型 $I_{clr}$ 反算為物理 GHI。
- **預測時間尺度 (Forecast Horizon)**：16 分鐘（$t+2, t+4, \dots, t+16$ 分鐘，共 8 步）。
- **輸入特徵組合**：
  - 過去 15 分鐘的 8 幀天空影像（$t-14:2:t$），解析度 $64 \times 64$。
  - 確定性時間協變量 $\mathbf{e} = [\text{hour, minute, day of year}, I_{clr}]$。
  - **不輸入歷史日射計數值**（嚴苛但貼近實際邊緣案場）。
- **資料集 (SIRTA Atmospheric Observatory, France)**：
  - 地點：法國帕萊索（Palaiseau, France, 48.713°N, 2.208°E）。
  - 時間切分：2023 全年（12,906 序列，43 個測試日），且進行 2021–2022 跨年份無洩漏重訓驗證。
  - 匹配評測子集：涵蓋 40 個測試日、共 1,998 組對齊測試序列。

---

## 3. 方法論與架構細節 (Methodology & Architecture)

模型參數量約 198.8M，由四個核心部分組成：

```text
[歷史影像 a (8 幀)] -> VAE -> clean tokens a' (128x256)
[最新單幀 a^i]     -> ControlNet (空間殘差注入)
[時間特徵 e]       -> Cross-Attention 條件

反向擴散去噪迴圈 (50 Reverse Steps, t=T -> 0):
┌────────────────────────────────────────────────────────┐
│ 1. 條件傳遞 pass:                                      │
│    H_phi(y_t, t; e, a') -> 產生當前 CSI token s_t      │
├────────────────────────────────────────────────────────┤
│ 2. 影像去噪 pass:                                      │
│    G_theta(x_t, t; [e, s_t], a^i) -> 預測噪聲 epsilon_x│
│    計算去噪淨值: \hat{x}_0 = (x_t - sqrt(1-a)*sg[eps]) / sqrt(a) │
├────────────────────────────────────────────────────────┤
│ 3. 日照預測 pass:                                      │
│    H_phi(y_t, t; e, T(\hat{x}_0)) -> 預測 CSI 噪聲 epsilon_y │
└────────────────────────────────────────────────────────┘
          │ (t=0 時輸出)
          ▼
未來 8 幀生成影像 (副產物) + 未來 8 步 CSI 預測 (主輸出 GHI)
```

### 3.1 梯度拓撲與單向流動 (Gradient Topology)
- CSI token $s_t$ 無截斷進入影像分支，因此日照預測能提供梯度給影像特徵；
- 但從影像反饋至日照預測時，$\hat{x}_0$ 採用了 **Stop-Gradient 截斷 $\text{sg}[\epsilon_x]$**。
- **優勢**：防止雙向梯度循環引發震盪，使影像分支專注於高保真天空重建，日照分支則專注於從重建的未來雲圖中解讀輻射通量。

---

## 4. 關鍵實驗結果與基準比對 (Key Results & Benchmarks)

### 4.1 SIRTA 測試集（1,998 組樣本）綜合評測對比
| 模型名稱 | 模型架構類型 | 歷史日照輸入 | RMSE ($W/m^2$) ↓ | CRPS ($W/m^2$) ↓ | CRPSS (vs. SPM) ↑ | 預測特性評語 |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Smart Persistence (SPM)** | 確定性基準 | 是 | 127.4 | 56.1 | 0.000 | 晴空指數恆定，無法預測雲動 |
| **DLinear** | 純時序線性模型 | 是 | 109.8 | 58.0 | -0.034 | 均值有改善，但分佈劣於持久模型 |
| **PatchTST** | 時序 Transformer | 是 | 109.6 | 54.5 | +0.029 | 略微改善機率分佈 |
| **SUNSET (Sun et al.)** | 確定性 CNN | 是 | **97.6** | 52.7 | +0.060 | **點預測 RMSE 最優**，但無機率分佈 |
| **ConvLSTM $\to$ U-Net** | 確定性視訊兩階段 | 否 | 145.3 | 77.0 | -0.372 | 畫面模糊，嚴重劣於持久模型 |
| **PhyDNetGAN $\to$ U-Net** | 物理 GAN 兩階段 | 否 | 112.3 | 61.8 | -0.103 | 兩階段中較佳者，但仍為負技能 |
| **SkyGPT (Nie et al.)** | 隨機視訊兩階段 | 否 | 216.8 | 121.1 | -1.159 | 移植至 SIRTA 經獨立 U-Net 產生嚴重失真 |
| **Xiong et al. (2025)** | SDE-Diff + GPA | 是 | 116.9 | 47.8 | +0.148 | 引入歷史日照，機率表現良好 |
| **Proposed (joint, past sky)** | 聯合擴散 (讀歷史幀) | 否 | 114.1 | 42.9 | +0.235 | 讀歷史天空已優於所有基準 |
| **Proposed (joint, future sky)** | **聯合單階段擴散 (讀 $\hat{x}_0$)** | **否** | **104.1** | **36.9** | **+0.341** | **CRPS 與 CRPSS 全場第一** |

### 4.2 兩階段 vs. 單階段的致命對比（Table 3）
在完全相同的 1,998 組樣本上，評估「生成的未來影像」如何被解讀：
- 用獨立訓練的 Frozen U-Net 讀取生成的未來影像：$\text{RMSE} = \mathbf{132.2}\ W/m^2$
- 用本論文聯合訓練的 Irradiance Head 讀取完全相同的生成影像：$\text{RMSE} = \mathbf{104.1}\ W/m^2$
- **結論**：單是「拿掉兩階段級聯、改用聯合單階段去噪」，就直接抹平了 **28.1 $W/m^2$ 的系統性誤差**！

### 4.3 爬坡事件（Ramp Events, $n=334$）與不確定性響應
- 90% 置信區間寬度與未來發生的實際爬坡幅度呈高度強相關（**Pearson $r = +0.85$**）；
- 在平穩非爬坡時，區間平均寬度僅 87 $W/m^2$；當劇烈爬坡來臨時，區間自發擴展至 **417 $W/m^2$**；
- 在爬坡子集上，本論文方法的 CRPSS 高達 **+0.440**（Xiong et al. 為 +0.310，SUNSET 僅 +0.198）。

---

## 5. 深入剖析與局限性 (Critical Analysis & Limitations)

### 優勢剖析：
1. **打破兩階段迷思**：徹底釐清了過去 SkyGPT 等文獻中「為什麼生成的畫面很漂亮，但算出來的功率誤差很大」的謎團——問題出在獨立訓練的回歸器對生成影像的特徵分佈漂移極度敏感。
2. **無日射計依賴**：完全不吃歷史 GHI 數值，打破了「無實測時序就無法精準預報」的桎梏。

### 局限性與代價：
1. **點預測 RMSE 仍略遜於 SUNSET**：SUNSET 達到 97.6 $W/m^2$，優於本模型的 104.1 $W/m^2$。消融實驗（Table 10）指出，若為本模型加入過去 8 步實測日照，RMSE 可立刻下降 4.8~7.1 $W/m^2$，縮小差距。
2. **算力與模型體積龐大**：參數量高達 1.98 億（基於 Stable Diffusion 1.5），50 步反向擴散需要較高 GPU 顯存與時間，不及 I-GPT（0.5秒）的邊緣輕量性。

---

## 6. 與其他文獻之脈絡關聯 (Connections to Other Work)

- **直面挑戰與對比**：
  - 挑戰了 [[2024-nie-skygpt|SkyGPT]] 與 [[2025-pan-igpt-two-stage|I-GPT]] 所代表的「視訊生成 $\to$ 獨立回歸」兩階段級聯典範。
  - 對比同期的 [[2025-xiong-multimodal-diff-transformer|Xiong et al. (2025)]]：Xiong 採用 Neural SDE 視訊模型加上獨立多模態 Transformer；本篇則直接將視訊與日照放在同一個 Diffusion 逆過程中 Co-denoising，且在無歷史日照輸入的劣勢下依然在 CRPS 與 CRPSS 上超越 Xiong et al.。
- **關鍵概念**：
  - [[single-stage-joint-diffusion|Single-Stage Joint Diffusion]]
  - [[denoised-estimate-conditioning|Denoised Estimate Conditioning ($\hat{x}_0$)]]
  - [[sirta-dataset|SIRTA Dataset]]
