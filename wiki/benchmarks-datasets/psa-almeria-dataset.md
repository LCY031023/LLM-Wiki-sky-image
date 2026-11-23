# Plataforma Solar de Almería (PSA) Datasets

- **地理位置 / 氣候類型**：西班牙阿爾梅里亞塔韋爾納斯 (Tabernas, Almería, Spain, 37.09°N, 2.35°W / 海拔 500m，BWk 冷沙漠氣候，受內華達山脈地形強烈影響)
- **相機硬體**：德國 Mobotix Q25 / Q26 全球面魚眼攝影機（180° FOV），固定曝光 160 $\mu\text{s}$，採樣間隔 30 秒，影像尺寸裁切為 $128 \times 128$。
- **輻射感測設備**：ISO 9060:2018 Class A Kipp & Zonen 頂級日射計（1 Hz 採樣，1 分鐘平均值）。
- **涵蓋測站**：PSA 內部 4 個主要微氣象測站（HP, KONTAS, METAS, PVOT），包含 2019–2024 近 5 年跨季資料。

---

## 跨模型排行榜與基準比對 (Leaderboard & Comparative Results)

### 評測子集一：PSA-Bench2019 基準集（METAS 測站，28 天人工精選代表日）
| 模型名稱 | 架構分類 | 15-min RMSE ($W/m^2$) ↓ | 15-min MAE ($W/m^2$) ↓ | 系統偏差 MBE ($W/m^2$) | 機率技能 CRPSS ↑ | 爬坡檢測 F1-Score ↑ |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **FarSky (Fabel et al., 2026)** | **任務感知潛空間擴散 (DCAEIrr + FAR)** | **50.0** | **25.8** | **+3.6** | **0.715** | **0.618** |
| E2E-TF (Fabel et al., 2024) | 端到端時空 Transformer | 57.7 | 32.1 | -4.0 | 0.677 | 0.004 (完全失效) |
| FAR (解耦基準) | 解耦 DCAE + 獨立 CNN 回歸 | 69.7 | 49.1 | -34.9 | 0.396 | 0.592 |
| Scaled Persistence (SP) | 晴空指數恆定基準 | 73.5 | 49.0 | +0.5 | 0.000 | — |
| SkyGPT (Nie et al., 2024) | VQ-VAE + PhyCell + U-Net | 86.8 | 63.4 | +1.3 | 0.275 | 0.384 |

### 評測子集二：SolarVision Almería (SVA-Test, PVOT 測站，60 天獨立測試集)
| 模型名稱 | 架構分類 | 15-min RMSE ($W/m^2$) ↓ | 15-min MAE ($W/m^2$) ↓ | 系統偏差 MBE ($W/m^2$) | 機率技能 CRPSS ↑ | 爬坡檢測 F1-Score ↑ |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **FarSky (Fabel et al., 2026)** | **任務感知潛空間擴散 (DCAEIrr + FAR)** | **98.0** | **50.6** | **+15.8** | **0.540** | **0.673** |
| E2E-TF (Fabel et al., 2024) | 端到端時空 Transformer | 100.8 | 55.3 | -7.6 | 0.532 | 0.021 (失效) |
| FAR (解耦基準) | 解耦 DCAE + 獨立 CNN 回歸 | 122.8 | 80.2 | -44.4 | 0.213 | 0.678 |
| Scaled Persistence (SP) | 晴空指數恆定基準 | 129.2 | 70.0 | +0.2 | 0.000 | — |
| SkyGPT (Nie et al., 2024) | VQ-VAE + PhyCell + U-Net | 146.7 | 108.2 | -20.9 | -0.070 | 0.427 |
