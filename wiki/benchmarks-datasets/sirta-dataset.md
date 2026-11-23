# SIRTA Atmospheric Observatory Dataset

- **地理位置 / 氣候類型**：法國帕萊索 (Palaiseau, France, 48.713°N, 2.208°E / 溫帶海洋性氣候 Köppen Cfb)
- **相機規格**：全天空相機（All-Sky Imager），取樣頻率 2 分鐘，解析度縮放至 $64 \times 64$。
- **輻射感測設備**：同步高精度地面日射計量測之水平面總日照強度（Global Horizontal Irradiance, GHI）。
- **資料規模**：
  - 主要評測年份：2023 全年（12,906 組序列，43 個測試日，匹配對齊測試子集包含 1,998 組樣本）。
  - 跨年份驗證集：2021–2022 年（用於零洩漏跨年度泛化重訓）。

---

## 跨模型排行榜與基準比對 (Leaderboard & Comparative Results)

### 預測時程：未來 16 分鐘（8 個時步，步長 2 分鐘）GHI 機率預測表現（1,998 筆匹配測試樣本）
| 論文 | 採用方法 | 架構分類 | 歷史日照輸入 | RMSE ($W/m^2$) ↓ | CRPS ($W/m^2$) ↓ | CRPSS (vs. SPM) ↑ | 表現特點 |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| [[2026-suwandee-single-stage-diffusion]] | **Proposed (joint, future sky)** | **單階段聯合擴散 (讀 $\hat{x}_0$)** | **否** | **104.1** | **36.9** | **+0.341** | **全場最優機率預報**：CRPS 最低，爬坡響應 $r=0.85$ |
| [[2026-suwandee-single-stage-diffusion]] | Proposed (joint, past sky) | 單階段聯合擴散 (讀歷史幀) | 否 | 114.1 | 42.9 | +0.235 | 讀歷史天空，次優 |
| [[2025-xiong-multimodal-diff-transformer]] | Xiong et al. (Re-impl.) | Neural SDE + Transformer GPA | 是 | 116.9 | 47.8 | +0.148 | 引入實測日照時序，機率表現居基線之首 |
| [[2024-nie-skygpt]] | SUNSET (Sun et al.) | 確定性 CNN 回歸 | 是 | **97.6** | 52.7 | +0.060 | **點預測 RMSE 最優**，但無機率不確定性 |
| (時序基準) | PatchTST (with $I_{clr}$) | 純數值時序 Transformer | 是 | 109.6 | 54.5 | +0.029 | 僅在均值上略勝持久模型 |
| (時序基準) | DLinear (with $I_{clr}$) | 純數值線性外推 | 是 | 109.8 | 58.0 | -0.034 | 缺乏天空影像，CRPS 劣於持久模型 |
| [[2024-nie-skygpt]] | PhyDNetGAN $\to$ Frozen U-Net | 物理 GAN 兩階段級聯 | 否 | 112.3 | 61.8 | -0.103 | 影像銳利但受兩階段回歸誤差拖累 |
| [[2024-nie-skygpt]] | ConvLSTM $\to$ Frozen U-Net | 確定性時空 RNN 兩階段 | 否 | 145.3 | 77.0 | -0.372 | 畫面嚴重平滑模糊，表現差 |
| [[2024-nie-skygpt]] | SkyGPT $\to$ Frozen U-Net | 隨機視訊兩階段級聯 | 否 | 216.8 | 121.1 | -1.159 | 移植至新案場經獨立回歸器產生嚴重分佈漂移 |
| (參考基準) | Smart Persistence (SPM) | 晴空指數恆定假說 | 是 | 127.4 | 56.1 | 0.000 | 基準無機率評估能力 |
