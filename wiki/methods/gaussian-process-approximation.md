# Gaussian Process Approximation (GPA)

## 1. 概述與數學原理 (Overview & Formulation)
在深度學習中，神經網路輸出常存在「過度自信（Overconfidence）」的認知缺陷，尤其在遇到未見過的天氣分佈（Out-of-Distribution, OOD）時無法如實反映不確定性。
**高斯過程近似 (Gaussian Process Approximation, GPA)** 是一種使確定性深度神經網路具備距離感知（Distance-Awareness）與不確定性輸出能力的理論方法。

### 核心條件：
1. **譜歸一化 (Spectral Normalization)**：對殘差塊權重施加譜範數約束：
   $$A_l = \begin{cases} c \frac{A_l}{\hat{\lambda}} & \text{if } c < \hat{\lambda} \\ A_l & \text{otherwise} \end{cases}$$
   保證特徵隱空間保持雙射距離（Distance-Preserving Lipschitz Condition）。
2. **隨機傅立葉特徵擴展 (Random Fourier Features, RFF)**：
   $$\Phi(x) = \sqrt{\frac{2}{D_L}} \cos(-A_L h(x) + b_L)$$
   利用 Laplace 近似將深度特徵映射至再生核希爾伯特空間（RKHS），輸出預測均值 $\hat{\mu}$ 與共變異數矩陣 $\hat{\Sigma}$。

---

## 2. 雙重不確定性全方差聚合 (Total Variance Law)
在 [[2025-xiong-multimodal-diff-transformer|Xiong et al. (2025)]] 中，GPA 與隨機生成視訊結合，透過全方差定律整合兩類不確定性：
$$\hat{\mu}_d = \frac{1}{N}\sum_{i=1}^N \hat{\mu}_{m_i}$$
$$\hat{\sigma}_d^2 = \frac{1}{N}\sum_{i=1}^N \left( \hat{\sigma}_{m_i}^2 + \hat{\mu}_{m_i}^2 \right) - \hat{\mu}_d^2$$
其中 $\hat{\sigma}_{m_i}^2$ 表徵神經網路模型本身的不確定性（Epistemic），各生成影像間均值的分散程度則表徵雲層運動的物理隨機性（Aleatoric）。

---

## 3. 關聯論文 (Associated Papers)
- [[2025-xiong-multimodal-diff-transformer]]：結合 GPA 與多模態 Transformer 建立端到端機率預測區間。
