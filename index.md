# Research Wiki Index

> **主題領域**：基於天空影像之生成式太陽光電機率預測 (Sky-Image-Based Generative Solar Irradiance and Photovoltaic Probabilistic Forecasting)  
> **最後更新**：2026-11-24

本索引為整個知識庫的語意目錄導航，隨文獻吸收與專題探討持續動態擴充。

---

## 1. 論文總覽 (`wiki/papers/`)
*已吸收論文之深度解讀、實驗規格與局限性剖析。*

- [[2024-nie-skygpt|2024 - Nie et al. - SkyGPT: Probabilistic ultra-short-term solar forecasting using synthetic sky images from physics-constrained VideoGPT]]：結合偏微分方程物理約束 (PhyCell) 與隨機視訊生成 (VideoGPT) 產生未來多情境天空圖，搭配改良 U-Net 達成 15 分鐘光電機率預測。
- [[2025-xiong-multimodal-diff-transformer|2025 - Xiong et al. - Multimodal ultra-short-term probabilistic solar power forecasting with generative AI and Transformer]]：利用 Neural SDE + 條件擴散模型生成未來天空視訊，並以多模態 Transformer 融合歷史功率，搭配高斯過程近似 (GPA) 實現全方差雙重不確定性量化。
- [[2025-pan-igpt-two-stage|2025 - Pan & Lim - A Generative Sky Image-Based Two-Stage Framework for Probabilistic Photovoltaic Power Forecasting]]：提出多尺度 PhyCell、動態加權融合模組 (DWFM) 與注意力門控 IP-U-Net，專注於純影像抗延遲特性與極致即時推論速度 (0.5s)。
- [[2025-atmioui-skyfusion|2025 - El Atmioui et al. - SkyFusion: A Physics-Informed GAN-Transformer Framework for Intra-Hour Sky Forecasting and PV Prediction]]：提出四階段 GAN-Transformer 藍圖，並利用 PVLib 構建 4 通道太陽位置二值遮罩 (RGB+Sun Mask)，結合 PatchGAN 提升雲層銳利度。
- [[2026-huang-pv-mm-diffusion|2026 - Huang et al. - PV-MM-diffusion: An end-to-end multi-modal diffusion model for ultra-short-term probabilistic photovoltaic forecasting]]：首創端到端多模態擴散模型，透過 Coupled U-Net 與隨機位移注意力 (RS-PV-MMA) 同步去噪未來天空影像與功率時序，並以空幀佔位支援缺模態容災。
- [[2026-suwandee-single-stage-diffusion|2026 - Suwandee & Suwanwimolkul - Single-stage joint diffusion of sky images and clear-sky index for ultra-short-term irradiance forecasting]]：首創單階段聯合擴散模型，消除中介回歸器與複合誤差傳遞，以去噪淨值估計 ($\hat{x}_0$) 引導日照頭，刷新 SIRTA 觀測站機率預測 SOTA。
- [[2026-fabel-farsky|2026 - Fabel et al. - FarSky: Task-Aware Latent-Space Coupling for Generative Intra-Hour Solar Forecasting]]：提出任務感知自編碼器 (DCAEIrr) 將日照特徵內嵌於潛空間，搭配流匹配潛擴散 (FAR) 直接解碼 GHI，消除分佈漂移負偏差並使爬坡 F1-Score 超過 60%。

---

## 2. 生成式模型與演算法 (`wiki/methods/`)
*天空影像生成、時空動態預測與深度學習回歸架構。*

- [[pv-mm-diffusion|PV-MM-Diffusion (End-to-End Multimodal Diffusion)]]：基於 Coupled U-Net 與 RS-PV-MMA 隨機位移注意力的端到端多模態聯合擴散架構。
- [[empty-frame-placeholder|Empty-Frame Placeholder Strategy]]：零張量佔位與混合模式損失，使單一擴散網路無縫支援全模態、純影像、純功率三種部署模式。
- [[single-stage-joint-diffusion|Single-Stage Joint Diffusion]]：將未來天空影像與晴空指數序列置於同一個擴散反向過程中同步協同去噪，徹底移除獨立回歸器。
- [[farsky-latent-coupling|FarSky Task-Aware Latent-Space Coupling]]：在自編碼器潛空間中同時受視覺重建與日照回歸監督（DCAEIrr），直接從預測潛狀態解碼 GHI。
- [[frame-autoregressive-diffusion|Frame AutoRegressive (FAR) Latent Diffusion]]：基於流匹配（Flow-Matching）的長程自迴歸潛空間視訊擴散架構。
- [[skyfusion|SkyFusion (GAN-Transformer Physics Pipeline)]]：結合 3D VQ-VAE、PhyCell Transformer、PatchGAN 與 CNN 功率頭的德國工業級四階段框架。
- [[skygpt|SkyGPT (Physics-Constrained VideoGPT)]]：解耦大尺度物理動力學（PDE PhyCell）與微觀雲層紋理（Transformer）的隨機視訊生成模型。
- [[i-gpt|I-GPT (Multi-scale Physics-Constrained Video Predictor)]]：多尺度 PhyCell 矩約束、動態加權融合模組 (DWFM) 與軸向注意力 VQ-VAE。
- [[neural-sde-diffusion-video|Neural SDE + Conditional Diffusion Video Predictor]]：利用隨機微分方程連續外推雲層運動，結合條件擴散逐步去噪生成高頻銳利天空影像。
- [[modified-unet-pv|Modified U-Net (Image-to-Power Regressor)]]：將影像分割 U-Net 改良為具空間加權可解釋性的天空影像至發電功率回歸架構。
- [[ip-unet|IP-U-Net (Attention-Gated Image-to-Power Regressor)]]：在跳躍連接處加入 Attention Gate 抑制背景雜訊、聚焦日盤光暈與雲邊緣。
- [[gaussian-process-approximation|Gaussian Process Approximation (GPA)]]：結合譜歸一化與隨機傅立葉特徵擴展，賦予多模態神經網路認知不確定性輸出能力。

---

## 3. 氣象物理與機率評估核心概念 (`wiki/concepts/`)
*光電工程、雲層物理、不確定性量化與機率評估準則。*

- [[sun-position-masking|Sun Position Masking (4th Channel via PVLib)]]：將太陽天頂角與方位角渲染為空間二值圓盤遮罩，作為第 4 通道輸入以約束日盤幾何。
- [[denoised-estimate-conditioning|Denoised Estimate Conditioning ($\hat{x}_0$)]]：利用擴散閉式解反推無噪未來目標以引導跨模態預測頭的關鍵機制。
- [[continuous-ranked-probability-score|Continuous Ranked Probability Score (CRPS)]]：綜合評定機率預測可靠性與銳利度的黃金標準指標，維度與物理量一致。
- [[winkler-score|Winkler Score (WS)]]：針對置信區間預測（如 90% / 95% PI）兼顧寬度懲罰與脫靶重罰的評估函數。

---

## 4. 評測基準與資料集排行榜 (`wiki/benchmarks-datasets/`)
*公開與自建天空影像光電資料集之硬體規格與各模型跨篇競爭對比表。*

- [[stanford-rooftop-dataset|Stanford 30-kW Rooftop PV & Sky Image Dataset (SKIPP'D)]]：3 年高頻魚眼鏡頭天空影像與 30-kW 屋頂光電數據，包含 SkyGPT、I-GPT、SDE-Diffusion 與 PV-MM-Diffusion 的 15 分鐘跨模型即時對比榜。
- [[psa-almeria-dataset|Plataforma Solar de Almería (PSA) Datasets (Spain)]]：西班牙冷沙漠氣候多機位觀測平台，涵蓋 PSA-Bench2019 與 SVA-Test 雙基準排行榜。
- [[sirta-dataset|SIRTA Atmospheric Observatory Dataset (France)]]：法國帕萊索 2023 全年高精度 GHI 與天空影像資料集，記錄單階段擴散全面領先 12 種基線之 SOTA 榜單。
- [[kuitun-dataset|Kuitun Solar Power & Sky Image Dataset]]：中國新疆奎屯市包含 4 大天氣類型（晴、多雲、陰、雨）之 11.2 萬幀天空影像與光電發電多尺度預測排行榜（15/30/60 分鐘）。
- [[german-pv-testfield-dataset|German PV Test Field ASI Dataset]]：德國境內光電測試場 17.8 萬幀 30 秒高頻魚眼影像（附帶 4 通道太陽遮罩）。

---

## 5. 專題綜述與演進分析 (`wiki/syntheses/`)
*跨論文橫向矩陣、技術演進脈絡與問答歸檔。*

- [[two-stage-generative-forecasting-evolution|天空影像生成式太陽光電/日照機率預測之技術演進樹 (Generative Solar Forecasting Evolution)]]：深度剖析四代技術範式演進（端到端確定性 $\to$ 兩階段解耦生成 $\to$ 端到端多模態擴散 $\to$ 單階段聯合擴散 $\to$ 任務感知潛空間耦合），橫向對比六大主流模型。

---

## 6. 互動簡報與視覺化投影片 (`slides/`)
*學術報告、專題演講與知識傳播之高互動式 HTML 簡報。*

- [如何生成未來天空影像？——全天空雲層演變時空預報技術 (HTML 簡報)](file:///c:/Users/sandy/OneDrive/Desktop/LLM%20Wiki/slides/how-to-generate-future-sky-images.html)：包含 13 頁純淨淺藍系高智感學術互動簡報，100% 聚焦未來天空動態視訊生成主題。全景解構全天空攝影機（ASI）觀測原理、傳統光流/MSE均值模糊瓶頸、四大生成流派（離散代碼本 SkyGPT/I-GPT、神經 SDE Xiong et al.、幾何遮罩 SkyFusion、潛空間流匹配 FarSky）、五步端到端工程管線、三維品質評估維度（視覺逼真度、大氣流體一致性、環日區幾何保真度）與嚴格錨定之 [1]~[7] 參考文獻庫。支援鍵盤導航、快速總覽（G）、全螢幕（F）與動態自適應等比縮放。

