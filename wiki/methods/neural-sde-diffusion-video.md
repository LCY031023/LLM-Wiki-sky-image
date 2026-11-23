# Neural SDE + Conditional Diffusion Video Predictor

## 1. 概述與數學原理 (Overview & Mathematical Formulation)
此視訊預測架構由 [[2025-xiong-multimodal-diff-transformer|Xiong et al. (2025)]] 提出，旨在克服傳統自迴歸視訊生成（如 VideoGPT）或確定性時空模型在長時程天空影像預報中出現的動態失真與畫面平滑問題。

該架構由三部分構成：
1. **Conv-GRU 運動特徵提取器**：計算前後幀差分矩陣，經 CNN 與 GRU 編碼為時空運動隱狀態 $m_{t_0-1}$。
2. **神經隨機微分方程 (Neural SDE) 運動預測器**：
   $$dX_t = a(X_t, t)dt + b(X_t, t)dE_t$$
   其中 $a(\cdot)$ 為漂移係數（Drift），$b(\cdot)$ 為擴散係數（Diffusion），$E_t$ 為維納過程（Wiener Process）。兩者由小型神經網路參數化，使運動軌跡在連續時間維度上具備隨機微擾特性，逼真模擬大氣湍流與雲層形態演變。
3. **條件擴散生成模型 (Conditional Diffusion Model, CDM)**：
   - 前向過程逐步注入高斯噪聲，反向過程以 SDE 預測之運動特徵 $m_t$ 作為條件向量，引導 U-Net 逐步去噪生成未來清晰天空幀。

---

## 2. 在光電預測中的適用性 (Application in Solar Forecasting)
相較於 [[skygpt|SkyGPT]] 採用的 VQ-VAE 碼本量化，擴散模型能夠在高維連續像素空間中生成更豐富的雲層高頻邊緣細節與太陽光暈陰影。消融實驗顯示，在相同的下游回歸網路下，採用該模型生成影像的 15 分鐘光電預報 CRPS 較 SkyGPT 改善了 15.2%。

---

## 3. 關聯論文 (Associated Papers)
- [[2025-xiong-multimodal-diff-transformer]]：首次將 Neural SDE 與條件擴散模型結合應用於太陽光電超短期多情境預測。
