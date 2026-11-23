# Denoised Estimate Conditioning ($\hat{x}_0$)

## 1. 概念定義與數學原理 (Definition & Mathematical Formulation)
在擴散模型（Denoising Diffusion Probabilistic Models, DDPM）的反向採樣過程（Reverse Step $t \to t-1$）中，當前時步的模型狀態為帶有大量隨機噪聲的 $x_t$。
若下游模組（例如日照預測頭）直接讀取 $x_t$，在較大噪聲時步（$t \approx T$）時，特徵幾乎完全被高斯噪聲淹沒，無法提取有意義的雲層物理結構。

**去噪淨值估計 (Denoised Estimate $\hat{x}_0$)** 巧妙利用 DDPM 的前向閉式解公式進行逆推：
$$\hat{x}_0 = \frac{x_t - \sqrt{1 - \bar{\alpha}_t} \cdot \text{sg}[\epsilon_x]}{\sqrt{\bar{\alpha}_t}}$$
其中：
- $x_t$ 為當前噪聲潛變量；
- $\epsilon_x$ 為影像去噪神經網路在當前步預測的噪聲向量；
- $\text{sg}[\cdot]$ 為停止梯度算子（Stop-Gradient）；
- $\bar{\alpha}_t$ 為預定的噪聲排程係數。

---

## 2. 在跨模態預報中的關鍵價值 (Value in Cross-Modal Forecasting)
[[2026-suwandee-single-stage-diffusion|Suwandee & Suwanwimolkul (2026)]] 在消融實驗中測試了四種不同的資訊傳遞方式：
1. 僅讀歷史乾淨幀 $a'$（無未來資訊）：$\text{RMSE} = 114.50\ W/m^2$
2. 讀帶噪未來狀態 $x_t$（噪聲過大）：$\text{RMSE} = 114.68\ W/m^2$
3. 讀歷史幀與帶噪狀態拼接 $[x_t, a']$：$\text{RMSE} = 113.96\ W/m^2$
4. **讀取去噪淨值估計 $\hat{x}_0$（本文方法）**：$\text{RMSE} = \mathbf{104.58}\ W/m^2$（顯著降低近 10 $W/m^2$！）

### 結論：
在跨模態擴散中，**輔助預測頭只有在讀取無噪的未來目標估計值（$\hat{x}_0$）時才能有效發揮指導作用**；讀取中間的加噪態 $x_t$ 只會增加隨機擾動，無法提供實質的光通量空間資訊。

---

## 3. 關聯論文 (Associated Papers)
- [[2026-suwandee-single-stage-diffusion]]：詳細論證了 $\hat{x}_0$ 條件化在單階段聯合擴散中的理論與實證優勢。
