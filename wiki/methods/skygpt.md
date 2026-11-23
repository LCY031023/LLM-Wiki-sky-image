# SkyGPT (Physics-Constrained VideoGPT)

## 1. 概述與數學直覺 (Overview & Intuition)
**SkyGPT** 是一種專為雲層運動與天空影像預測設計的隨機視訊生成模型（Stochastic Video Prediction Model），由 Nie 等人於 2024 年提出。
其核心精神在於**「物理動態與視覺外觀的解耦表示（Disentanglement of Dynamics and Appearance）」**：
傳統純深度學習模型（如 VideoGPT）在小樣本資料上容易生成不符合流體力學的隨機雲漂移；而傳統確定性物理模型（如 PhyDNet 或 ConvLSTM）在面對長時程時容易因最小化均方誤差（MSE）而產生「平均化」的嚴重模糊影像。

SkyGPT 巧妙結合了兩者優勢：
- **離散表示空間 (VQ-VAE)**：將天空影像特徵壓縮至離散碼本（Codebook），天然適合表徵晴天、厚雲、卷雲等不同大氣離散形態。
- **物理約束分支 (PhyCell)**：利用卷積核近似二維線性偏微分方程（Linear PDEs），約束雲層宏觀對流與擴散（Advection-Diffusion Dynamics）。
- **細節生成分支 (Transformer)**：在潛空間以自迴歸（Auto-regressive）方式建立局部 Patch 關聯，生成清晰的雲邊緣與光影細節。

---

## 2. 在天空影像預測中的條件化形式 (Formulation in Sky-Image Forecasting)
輸入過去 15 分鐘的天空影像序列 $\mathcal{I}_{t-15:2:t-1}$（共 8 幀）：
1. 影像輸入 VQ-VAE Encoder 提取空間特徵，並透過 3D-ResNet 抽取歷史條件特徵。
2. 潛變量在 Phy-Transformer 內部進行雙分支預測：
   $$\hat{E}_{t+1} = \hat{E}_{t+1}^{Phy} \oplus \hat{E}_{t+1}^{Trans}$$
   其中 $\hat{E}^{Phy}$ 保證雲團大尺度位移符合物理定律，$\hat{E}^{Trans}$ 透過交叉注意力機制合成微觀外觀。
3. 推論時，從學得的條件先驗中隨機採樣 $N_f$ 條不同的潛路徑，解碼為 $N_f$ 種具有物理合理性且具多樣性的未來天空影像。

---

## 3. 關聯論文 (Associated Papers)
- [[2024-nie-skygpt|Nie et al. (2024)]]：首次提出 SkyGPT，並驗證其在 15 分鐘超短期太陽光電機率預測中的優異表現。

---

## 4. 核心優勢與權衡代價 (Strengths vs. Trade-offs)
- **優勢**：
  - **高保真度與銳利度**：超越傳統 ConvLSTM 與 PhyDNet 的模糊瓶頸，5 分鐘以上的 VGG 餘弦相似度顯著領先。
  - **物理一致性引導的多樣性**：避免純生成模型在長時程預報時產生逆風向漂移等不合常理的幻想。
- **權衡代價**：
  - **採樣運算開銷**：自迴歸 Transformer 與多次隨機採樣推論延遲較高（A100 上生成 50 個情境需約 32 秒）。
  - **未與下游任務聯合微調**：第一階段視訊生成與第二階段功率回歸分離，可能在日照穩定時生成不必要的過寬預測區間。
