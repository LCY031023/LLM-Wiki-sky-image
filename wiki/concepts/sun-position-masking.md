# Sun Position Masking (4th Channel Conditioning)

## 1. 概念定義與動機 (Definition & Motivation)
在全天空影像中，太陽的位置是決定地面光電發電功率與直接日射（DNI）的核心幾何中心。
然而在傳統模型中：
- 僅輸入 RGB 3 通道影像，當薄雲或厚雲遮蔽太陽時，網路極易「丟失太陽座標」；
- 若僅將太陽天頂角與方位角作為 1D 數值純量餵給 MLP，卷積神經網路很難建立 1D 角度與 2D 影像特徵圖的空間幾何關聯。

**太陽位置遮罩 (Sun Position Masking)** 是由 [[2025-atmioui-skyfusion|El Atmioui et al. (2025, SkyFusion)]] 提出的一種物理幾何嵌入特徵工程：
利用天文演算法庫（如 `pvlib`）精確計算給定時間與地理座標的太陽天頂角與方位角，在全黑背景上繪製白色日盤圓形二值遮罩，並作為**第四個空間通道（4th Channel）**與原始 RGB 影像拼接。

---

## 2. 數學與張量表示
輸入張量擴展為：
$$\mathbf{X}_{input} \in \mathbb{R}^{B \times 4 \times H \times W}$$
其中前 3 個通道為天空 RGB 影像，第 4 個通道為：
$$\mathbf{M}_{sun}(u, v) = \begin{cases} 1 & \text{若像素 } (u,v) \text{ 位於太陽盤投影半徑內} \\ 0 & \text{其他區域} \end{cases}$$

---

## 3. 核心優勢 (Strengths)
1. **防止日盤漂移幻覺**：在自迴歸視訊生成或長時程外推時，約束模型嚴格遵守天體力學規律，避免生成模型將雲團位移誤認為太陽位置跳躍。
2. **空間注意力天然引導**：卷積核與注意力機制能以極低的計算代價提取「雲層邊緣相對於太陽盤邊界的距離與幾何截斷關係」。

---

## 4. 關聯論文 (Associated Papers)
- [[2025-atmioui-skyfusion]]：首次在 ConvLSTM-UNet 與生成架構中全面引入 4 通道 Sun Masking。
