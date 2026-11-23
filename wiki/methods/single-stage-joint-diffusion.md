# Single-Stage Joint Diffusion

## 1. 概述與典範革新 (Overview & Paradigm Shift)
**單階段聯合擴散 (Single-Stage Joint Diffusion)** 是由 Suwandee & Suwanwimolkul (2026) 提出的一種全新生成式太陽日照預測架構。
它根本性地顛覆了過去由 [[skygpt|SkyGPT]]、[[i-gpt|I-GPT]] 與 [[neural-sde-diffusion-video|Xiong et al.]] 所建立的「兩階段級聯（Two-stage Cascade）」框架。

### 兩階段架構的致命缺陷：
傳統兩階段方法中，第一階段生成高維視訊，第二階段以獨立回歸器（如 U-Net 或 ResNet）映射至發電量/日照。然而，**獨立回歸器通常在真實乾淨影像上訓練，推論時遇到生成影像的微小形變與伪影會產生嚴重的分佈偏移（OOD）懲罰**。Suwandee 等人證實，光是兩階段的割裂就會造成高達 28 $W/m^2$ 的額外 RMSE 誤差。

### 單階段聯合去噪機制：
單階段聯合擴散將「高維未來天空影像潛空間 $x_t$」與「標量未來晴空指數序列 $y_t$」置於同一個擴散反向過程（Reverse Diffusion Process）中同步去噪：
1. **影像分支 (Image Branch)**：基於預訓練 Stable Diffusion 1.5 + ControlNet + 跨幀時間細化器（Temporal Refiner），負責去噪生成未來天空潛特徵 $x_0$。
2. **日照預測頭 (Irradiance Head)**：專屬 Transformer，負責去噪未來的晴空指數（CSI）序列 $y_0$。
3. **單向耦合 (Open Gradient Topology)**：日照頭的 CSI tokens 無截斷進入影像分支的交叉注意力；影像分支產生的預測則透過 Stop-Gradient 截斷後的[[denoised-estimate-conditioning|去噪淨值估計 $\hat{x}_0$]] 反饋給日照頭。

---

## 2. 核心優勢 (Strengths)
1. **消除複合誤差傳遞**：日照預測直接在擴散過程中由專屬頭解析，未來天空影像僅為協同生成的「副產物」，徹底移除中介的影像回歸器。
2. **無須實測日照輸入**：即使沒有安裝高成本的地面日射計（Pyranometer），僅憑全天空攝影機影像與天文幾何協變量，即可輸出頂尖的機率預測區間。

---

## 3. 關聯論文 (Associated Papers)
- [[2026-suwandee-single-stage-diffusion]]：首次提出單階段聯合擴散並在 SIRTA 資料集上大幅超越 12 種基線。
