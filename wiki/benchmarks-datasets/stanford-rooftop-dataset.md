# Stanford 30-kW Rooftop PV & Sky Image Dataset (SKIPP'D Precursor)

- **地理位置 / 氣候類型**：美國加州史丹佛大學 (Stanford, California / 地中海型氣候 Köppen Csb)
- **攝影機規格**：海康威視魚眼全天空攝影機 (Hikvision DS-2CD6362F-IV)，180° FOV，每 1 分鐘擷取 1 幀，原圖 $2048 \times 2048$。
- **光電系統規格**：位於 Green Earth Sciences 大樓屋頂之 30-kW 光電案場（距相機約 125m），傾角 22.5°，方位角 195°。
- **公開資料集連結**：[SKIPP'D Dataset GitHub](https://github.com/yuhao-nie/Stanford-solar-forecasting-dataset)

---

## 1. 資料集概要
該資料集涵蓋 2017 年 3 月至 2019 年 12 月近三年的高頻同步天空影像與發電功率紀錄（總計 334,038 組對齊樣本）。
經由過濾晴空穩定樣本後，多雲挑戰子集（Cloudy Subset）包含 132,305 筆樣本；依據 2 分鐘步長進一步整理出 60,385 筆包含前後 15 分鐘序列的有效樣本：
- **訓練集 (Training Set)**：53,336 筆樣本（佔 88%）
- **驗證集 (Validation Set)**：4,467 筆樣本（佔 7%，手動挑選 10 個具代表性多雲日）
- **獨立測試集 (Test Set)**：2,582 筆樣本（佔 4%，2019 年 11~12 月完全跨時段的 5 個多雲日）

---

## 2. 跨模型排行榜與基準比對 (Leaderboard & Comparative Results)

### 預測時程：未來 15 分鐘（$\Delta t = 15$ min）機率預測表現（測試集 Test Set，2,582 筆樣本）
| 論文 | 採用方法 | 預測架構分類 | CRPS (kW) ↓ | WS (90% PI) ↓ | 技能分數 FS (%) ↑ | 預測品質評語 |
| :--- | :--- | :--- | :---: | :---: | :---: | :--- |
| [[2025-xiong-multimodal-diff-transformer]] | **SDE-Diffusion + Multimodal GPA** | 隨機擴散視訊 + 多模態 Transformer | **2.44** | **19.62** | **33.5%** | **多模態 SOTA**：納入歷史發電時序與雙重不確定性，指標全面領先 |
| [[2026-huang-pv-mm-diffusion]] | **PV-MM-Diffusion (SI PV $\to$ SI PV)** | **端到端多模態聯合擴散 (Coupled U-Net)** | **2.63** | **21.46** | **28.0%** | **端到端擴散 SOTA**：WS 改善 40%，大幅超越真實未來影像基準 |
| [[2025-xiong-multimodal-diff-transformer]] | Multimodal Baseline | 歷史影像+功率 Transformer | 2.70 | 23.36 | 26.4% | 未使用視訊生成，但勝過單模態 SkyGPT |
| [[2024-nie-skygpt]] | SkyGPT $\to$ Modified U-Net | 隨機視訊生成 + 空間加權 CNN | 2.81 ~ 3.21 | 26.72 ~ 35.62 | 23.0% | 具單尺度 PDE 物理約束，但未加 AG 與多尺度 |
| [[2025-pan-igpt-two-stage]] | **I-GPT $\to$ IP-U-Net** | 多尺度 PDE 視訊 + 注意力門控 CNN | **2.912** | **33.103** | **35.89%** | **純影像架構 SOTA**：多尺度物理+DWFM+AG，推論僅需 0.5 秒 |
| [[2026-huang-pv-mm-diffusion]] | PV-MM-Diffusion (SI $\to$ SI PV) | 純影像輸入端到端擴散 | 3.09 | 24.99 | 16.0% | 無日照計時之表現，區間表現依然出色 |
| [[2024-nie-skygpt]] | PhyDNet+GAN $\to$ U-Net | 確定性物理 GAN 生成 | 3.02 | 39.54 | 17.7% | 紋理銳利，但缺乏隨機多樣性 |
| [[2024-nie-skygpt]] | ConvLSTM $\to$ U-Net | 確定性時空 RNN 生成 | 3.09 | 43.82 | 15.8% | 超過 5 分鐘畫面嚴重平滑模糊 |
| [[2025-pan-igpt-two-stage]] | VideoGPT $\to$ IP-U-Net | 隨機自回歸視訊生成 | 3.090 | 36.830 | 31.97% | 隨機性強但缺乏物理引導，泛化性波動較大 |
| [[2024-nie-skygpt]] | PhyDNet $\to$ U-Net | 確定性 PDE 物理生成 | 3.12 | 45.20 | 15.0% | 捕捉主體位移，缺乏微觀細節 |
| [[2024-nie-skygpt]] | SUNSET (Sun et al., 2019) | 端到端歷史影像+功率 CNN | 3.31 | 56.95 | 9.8% | 嚴重依賴歷史功率趨勢，突變時延遲明顯 |
| *(基準對照)* | Smart Persistence (SPM) | 晴空指數恆定假說 | 3.67 ~ 4.54 | — | 0% | 基準無機率評估能力 |
| *(理論上限)* | *Real Future Image $\to$ IP-U-Net* | *真實未來影像 (Oracle)* | *2.34 ~ 2.595* | *27.13 ~ 32.599* | *36.2% ~ 42.87%* | *視訊生成模型完美情況下的理論極限* |
