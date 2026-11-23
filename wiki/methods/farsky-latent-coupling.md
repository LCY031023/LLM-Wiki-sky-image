# FarSky Task-Aware Latent-Space Coupling

## 1. 概念定義與動機 (Definition & Motivation)
在天空影像生成式預測中，傳統做法（如 [[skygpt|SkyGPT]]、[[i-gpt|I-GPT]]）先利用未經下游任務監督的自編碼器（VAE/DCAE）將影像壓縮至潛空間，在潛空間預測未來視訊後，再以獨立訓練的回歸器計算功率。
**任務感知潛空間耦合 (Task-Aware Latent-Space Coupling)** 是由 Fabel et al. (2026, FarSky) 提出的一種表徵學習機制，其主張：
**自編碼器的潛空間必須在訓練之初就同時受到「影像視覺重建」與「下游日照估計」的雙重監督**。

---

## 2. DCAEIrr 架構設計
DCAEIrr 擴展了 Deep Compression Autoencoder (DCAE)：
- **編碼器**：$z = E(x)$ 將 $128 \times 128 \times 3$ 天空圖壓縮為低維潛變量 $z$；
- **雙解碼頭**：
  1. 影像重建解碼器：$\hat{x} = D_x(z)$
  2. 日照估計 MLP 頭：$\hat{y} = D_y(z)$（全域平均池化 + 2 層 MLP，輸出標量 GHI）
- **多任務聯合損失**：
  $$\mathcal{L} = \mathcal{L}_{DCAE} + \lambda_{irr} \|\hat{y} - y\|_1$$
  其中 $\mathcal{L}_{DCAE}$ 包含像素 L1、感知損失（Perceptual Loss）與對抗損失（Adversarial Loss）。

---

## 3. 解決的核心痛點
1. **消除介面級分佈偏移 (Mitigating Interface Domain Shift)**：
   獨立訓練的 CNN/U-Net 回歸器在面對生成視訊時會產生嚴重的負向系統偏差（MBE 達到 -35 至 -44 $W/m^2$）；而經由 DCAEIrr 訓練的潛空間 MLP 頭，直接從擴散模型生成的未來潛狀態 $z_{t+h}$ 中解碼 GHI，將系統偏差縮小至接近 0（MBE 僅 +3.6 $W/m^2$）。
2. **零時步真實影像精度翻倍**：在未經時間外推的真實影像上，DCAEIrr 的單幀 GHI 估計 RMSE 由獨立 CNN 的 52.0 $W/m^2$ 驟降至 25.9 $W/m^2$。

---

## 4. 關聯論文 (Associated Papers)
- [[2026-fabel-farsky]]：提出 FarSky 架構與 DCAEIrr 潛空間耦合機制。
