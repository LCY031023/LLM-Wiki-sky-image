# I-GPT (Image-GPT with Multi-Scale Physics Constraints)

## 1. 概述與架構演進 (Overview & Architectural Evolution)
**I-GPT** 是由 Pan & Lim (2025) 提出的一種多尺度物理約束隨機天空影像預測模型，旨在解決先前 [[skygpt|SkyGPT]] 中單尺度偏微分方程約束無法兼顧細碎雲絲與大尺度雲團、以及潛空間特徵簡單相加導致資訊耦合不良的缺陷。

### 三大關鍵技術改進：
1. **多尺度物理約束模組 (Multi-scale PhyCell)**：
   - 傳統 PhyCell 僅採用單一卷積核大小（如 $3 \times 3$），難以應對真實大氣中不同尺度的流體運動。
   - Multi-scale PhyCell 引入多個並行卷積組，透過改變卷積核尺寸（$3\times3, 5\times5, 7\times7$）、步長（Stride）與膨脹率（Dilation）獲取不同的感受野（Receptive Fields），並同時受線性 PDE 矩損失（$\mathcal{L}_{moment}$）約束，分別捕捉高頻微觀雲絲動態與中尺度大雲體平流。
2. **動態加權融合模組 (Dynamic Weight Fusion Module, DWFM)**：
   - 摒棄 SkyGPT 中的「物理特徵 + 外觀特徵」直接逐元素相加。
   - DWFM 採用兩條並行 3D 卷積分支：
     - **GELU 激活分支**：提取大氣光照與雲彩紋理的高階非線性特徵。
     - **Sigmoid 門控分支**：產生自適應空間權重矩陣（Spatial Gating Mask），依據天候劇烈程度自動平衡物理動力學與視覺外觀特徵的權重。
3. **軸向注意力自編碼器 (Axial Attention VQ-VAE)**：
   - 在 VQ-VAE 的編碼器與解碼器殘差塊中採用軸向注意力（Axial Attention），將二維空間與時間維度解耦運算，大幅降低多幀高維視訊自注意力的運算開銷。

---

## 2. 性能優勢 (Strengths)
在史丹佛大學天空影像測試集上，相較於 SkyGPT，I-GPT 在 5~15 分鐘長時程預報中展現出更慢的 VGG-CS 衰減率，生成的雲邊緣更加銳利，並有效消除了非物理漂移幻覺。

---

## 3. 關聯論文 (Associated Papers)
- [[2025-pan-igpt-two-stage]]：首次提出 I-GPT 架構並成功應用於 15 分鐘光電機率預報。
- [[2024-nie-skygpt]]：I-GPT 之原型與改進基礎。
