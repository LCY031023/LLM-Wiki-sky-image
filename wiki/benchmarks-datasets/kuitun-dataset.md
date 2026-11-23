# Kuitun Solar Power & Sky Image Dataset

- **地理位置 / 氣候類型**：中國新疆奎屯市 (Kuitun, Xinjiang, China / 溫帶大陸性乾旱氣候，日照充沛但雲遮突變強烈)
- **相機規格**：全天空相機，原始解析度 $2560 \times 1920$（降採樣至 $128 \times 128$），記錄時間 06:00 ~ 22:00。
- **資料規模**：2024 年 3 月 1 日至 7 月 31 日，共 112,599 幀天空影像與對齊光電發電功率數據。
- **天氣類別劃分**：晴天 (Sunny: 27天)、多雲 (Cloudy: 52天)、陰天 (Overcast: 15天)、雨天 (Rainy: 5天)。

---

## 跨模型排行榜與基準比對 (Leaderboard & Comparative Results)

### 預測時程：15 分鐘（15-min）、30 分鐘（30-min）、60 分鐘（60-min）綜合表現
| 預測時程 | 模型架構 | 類型 | RMSE (kW) ↓ | MAE (kW) ↓ | CRPS (kW) ↓ | MWIS (95% PI) ↓ | 表現評語 |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **15-min** | **Ours (Xiong et al., 2025)** | **多模態 SDE-Diffusion + GPA** | **7.80** | **4.96** | **4.07** | **48.22** | **SOTA**：多模態與雙重不確定性最優 |
| | Multimodal (Baseline) | 歷史影像+功率 Transformer | 8.85 | 5.65 | 4.45 | 64.84 | 缺乏未來視訊預測，精度次之 |
| | Time Series | 純功率時序 Transformer | 9.82 | 6.59 | 5.05 | 68.01 | 無法察覺雲層突發動態 |
| | Video Prediction | 純生成視訊 ResNet | 14.51 | 11.03 | 9.89 | 295.20 | 缺乏歷史功率標定，數值漂移嚴重 |
| | Image Only | 純歷史影像 ResNet | 14.79 | 11.09 | 8.35 | 100.76 | 缺乏時序動態與功率特徵 |
| **30-min** | **Ours (Xiong et al., 2025)** | **多模態 SDE-Diffusion + GPA** | **10.28** | **7.27** | **5.46** | **59.07** | **SOTA** |
| | Multimodal (Baseline) | 歷史影像+功率 Transformer | 11.66 | 8.01 | 6.11 | 74.51 | |
| | Time Series | 純功率時序 Transformer | 11.82 | 8.31 | 6.30 | 78.89 | |
| **60-min** | **Ours (Xiong et al., 2025)** | **多模態 SDE-Diffusion + GPA** | **12.83** | **9.38** | **6.90** | **74.11** | **SOTA** |
| | Multimodal (Baseline) | 歷史影像+功率 Transformer | 13.64 | 10.33 | 7.60 | 89.80 | |
| | Time Series | 純功率時序 Transformer | 14.80 | 10.45 | 7.96 | 99.33 | |
