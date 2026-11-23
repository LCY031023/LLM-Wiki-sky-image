# Research Wiki Operations Log

Chronological, append-only record of all ingestions, synthesis queries, and maintenance lints.

---

## [2026-11-23] init | Initialized LLM-Wiki Knowledge Base
- Created directory schema: `sources/`, `wiki/{papers,methods,concepts,benchmarks-datasets,syntheses}/`.
- Initialized `AGENTS.md` tailored for Sky-Image Generative Solar Probabilistic Forecasting.
- Generated `index.md` and `log.md`.

## [2026-11-23] ingest | 2024-nie-skygpt.md | 吸收 SkyGPT 論文：結合物理約束 PhyCell 與 VideoGPT 進行 15 分鐘天空影像與光電機率預報
- 建立了論文筆記：`wiki/papers/2024-nie-skygpt.md`。
- 建立了方法頁面：`wiki/methods/skygpt.md`、`wiki/methods/modified-unet-pv.md`。
- 建立了評估概念頁面：`wiki/concepts/continuous-ranked-probability-score.md`、`wiki/concepts/winkler-score.md`。
- 建立了評測資料集排行榜：`wiki/benchmarks-datasets/stanford-rooftop-dataset.md`。
- 更新了知識庫索引 `index.md`。

## [2026-11-23] ingest | 2025-xiong-multimodal-diff-transformer.md | 吸收 Xiong 等人多模態擴散預測論文：Neural SDE + Diffusion 搭配 Transformer GPA 刷新 SOTA
- 建立了論文筆記：`wiki/papers/2025-xiong-multimodal-diff-transformer.md`。
- 建立了方法頁面：`wiki/methods/neural-sde-diffusion-video.md`、`wiki/methods/gaussian-process-approximation.md`。
- 建立了資料集頁面：`wiki/benchmarks-datasets/kuitun-dataset.md`。
- 更新了基準資料集排行榜：`wiki/benchmarks-datasets/stanford-rooftop-dataset.md`（加入 Xiong et al. 新 SOTA 成績：CRPS 2.44 kW, WS 19.62 kW）。
- 更新了文獻演進互連：在 `wiki/papers/2024-nie-skygpt.md` 中標註被 Xiong 等人突破的兩大盲點。
- 同步更新知識庫索引 `index.md`。

## [2026-11-23] ingest | 2025-pan-igpt-two-stage.md | 吸收 Pan & Lim 的 I-GPT 論文：多尺度 PhyCell + DWFM + 注意力門控 IP-U-Net
- 建立了論文筆記：`wiki/papers/2025-pan-igpt-two-stage.md`。
- 建立了方法頁面：`wiki/methods/i-gpt.md`、`wiki/methods/ip-unet.md`。
- 更新了評測基準排行榜：`wiki/benchmarks-datasets/stanford-rooftop-dataset.md`（加入 I-GPT 在多雲測試集之成績：CRPS 2.912 kW, WS 33.103 kW）。
- 建立了跨論文專題綜述：`wiki/syntheses/two-stage-generative-forecasting-evolution.md`，深度梳理 3 篇論文在視訊生成（VideoGPT vs. I-GPT vs. Diffusion）與下游回歸（純影像抗延遲 vs. 多模態 GPA）的技術演進與核心權衡。
- 同步更新知識庫索引 `index.md`。

## [2026-11-24] ingest | 2026-suwandee-single-stage-diffusion.md | 吸收單階段聯合擴散論文：顛覆兩階段級聯，以去噪淨值估計 \hat{x}_0 刷新 SIRTA SOTA
- 建立了論文筆記：`wiki/papers/2026-suwandee-single-stage-diffusion.md`。
- 建立了方法頁面：`wiki/methods/single-stage-joint-diffusion.md`。
- 建立了核心概念頁面：`wiki/concepts/denoised-estimate-conditioning.md`。
- 建立了評測資料集排行榜：`wiki/benchmarks-datasets/sirta-dataset.md`。
- 升級了專題綜述：更新 `wiki/syntheses/two-stage-generative-forecasting-evolution.md`，深度記錄從「兩階段級聯」到「單階段聯合擴散」的三代範式轉移，剖析中介回歸器浪費 28 W/m² 精度的實證發現。
- 同步更新知識庫索引 `index.md`。

## [2026-11-24] ingest | 2026-fabel-farsky.md | 吸收 FarSky 論文：任務感知潛空間耦合 (DCAEIrr + FAR) 消除分佈漂移並大幅提升爬坡 F1-Score
- 建立了論文筆記：`wiki/papers/2026-fabel-farsky.md`。
- 建立了方法頁面：`wiki/methods/farsky-latent-coupling.md`、`wiki/methods/frame-autoregressive-diffusion.md`。
- 建立了評測資料集頁面：`wiki/benchmarks-datasets/psa-almeria-dataset.md`（包含 PSA-Bench2019 與 SVA-Test 雙測試集排行榜）。
- 全面升級專題綜述：更新 `wiki/syntheses/two-stage-generative-forecasting-evolution.md`，將 FarSky 納入四代範式演進樹（任務感知潛空間直接解碼），完整呈現 5 大頂尖模型橫向對比矩陣。
- 同步更新知識庫索引 `index.md`。

## [2026-11-24] ingest | 2026-huang-pv-mm-diffusion.md | 吸收 PV-MM-Diffusion 論文：Coupled U-Net 端到端多模態擴散與空幀佔位策略刷新 Stanford WS (21.46 kW)
- 建立了論文筆記：`wiki/papers/2026-huang-pv-mm-diffusion.md`。
- 建立了方法頁面：`wiki/methods/pv-mm-diffusion.md`、`wiki/methods/empty-frame-placeholder.md`。
- 更新了評測基準排行榜：`wiki/benchmarks-datasets/stanford-rooftop-dataset.md`（加入 PV-MM-Diffusion 成績：CRPS 2.63 kW, WS 21.46 kW，WS 超越理論上限 30%）。
- 全面升級專題綜述：更新 `wiki/syntheses/two-stage-generative-forecasting-evolution.md`，擴展為六大主流模型全景對比矩陣。
- 同步更新知識庫索引 `index.md`。

## [2026-11-24] ingest | 2025-atmioui-skyfusion.md | 吸收德國 Helios 專案 SkyFusion 論文：4 通道太陽位置遮罩 (PVLib) 與 PatchGAN 局部紋理細化
- 建立了論文筆記：`wiki/papers/2025-atmioui-skyfusion.md`。
- 建立了方法與概念頁面：`wiki/methods/skyfusion.md`、`wiki/concepts/sun-position-masking.md`。
- 建立了資料集頁面：`wiki/benchmarks-datasets/german-pv-testfield-dataset.md`。
- 更新了知識庫索引 `index.md`。

## [2026-11-24] maintenance | AGENTS.md | 強化來源歸檔規範：補齊對話框上傳 PDF 之自動存入 sources/ 規則
- 在 `AGENTS.md` 之【操作一：文獻吸收 (ingest)】補增「第 0 步：來源歸檔與事實錨定」。
- 明確規定：未來若使用者透過對話框上傳 PDF 附件，助理在解析前必須第一步自動將該 PDF 複製／歸檔至 `sources/`，防止實體文獻遺失，確保 `sources/` 為 100% 完整的 Single Source of Truth。

## [2026-11-24] presentation | slides/how-to-generate-future-sky-images.html | 產出「如何生成未來天空影像」高互動式 HTML 簡報投影片
- 建立了獨立簡報目錄與檔案：`slides/how-to-generate-future-sky-images.html`。
- 提煉了 Wiki 內 7 篇論文所涉及之生成式核心技術，製作共 12 頁現代化深色學術簡報。
- 簡報內建：
  - 鍵盤快捷鍵導航（方向鍵、空白鍵、Enter、Home/End）。
  - 投影片總覽抽屜與即時跳轉模態框（按 G 或 O 鍵）。
  - 全螢幕簡報模式（按 F 鍵）。
  - KaTeX 數學公式即時渲染與響應式 16:9 自動等比縮放。
- 更新了知識庫索引 `index.md`。

## [2026-11-24] style-redesign | slides/how-to-generate-future-sky-images.html | 全面升級為「深藍・淺藍・純白・琥珀金點綴」之極簡高智感演講型投影片
- 依使用者偏好對標 Conference Keynote（大字留白演講型）與 Clean Light（淺色洗練基調）：
  - **色彩系統**：以純白與淡冰藍灰（#F8FAFC / #F0F7FF）為基底，標題與權威文字採用深邃海軍藍（#0F2445），指示標籤採用天際淺藍（#0284C7），關鍵突破數值點綴沉穩太陽金黃（#D97706 / #F59E0B）。
  - **視覺質感**：融入微光細膩層次（Subtle Modern Tech），配備 1px 晶透微邊框、大字級留白、大指標強調（如 0.5s、28.1 W/m²、F1 > 60%）與精緻卡片懸浮微互動。
  - **精簡文字密度**：每頁聚焦 1~2 個核心論點與對照架構，保留 LaTeX 公式渲染與全功能鍵盤導航。

## [2026-11-24] style-refine | slides/how-to-generate-future-sky-images.html | 強化「淺藍為主」之大氣色彩層次，消弭大面積刺眼白底與單調感
- **主色調轉向大氣淺藍**：視窗全域底色升級為溫潤天際淺藍漸層（#EBF5FE ➔ #D8ECFC ➔ #CDE5FA），徹底移除死白與反光刺眼感。
- **異質化卡片層次節奏（告別千篇一律）**：
  - 引入高對比深海軍藍錨點卡片（Deep Navy Anchor #091C36），形成強烈的視覺焦點與閱讀呼吸感。
  - 核心突破數據與物理流體方程導入暖金琥珀色（Solar Amber #FFFBEB ➔ #FEF3C7）點綴卡片。
  - 天際淺藍漸層卡片（#F0F8FF ➔ #DCF0FE）作為基底，搭配 1.5px 堅實淡藍結構邊框與深海軍藍高對比文字（#081A36 / #1D3356），閱讀舒適度與層次大幅提升。

## [2026-11-24] style-harmony | slides/how-to-generate-future-sky-images.html | 全面校準留白間距與色彩協調性，消弭突兀對比並統一卡片語言
- **嚴格規範呼吸留白**：
  - 卡片內部內距擴增至充裕的 `26px 30px`，徹底解決文字緊貼邊框的擁擠感。
  - 文字行高統一設為舒適的 `1.6`，段落間距與列表項間距拉開，呼吸感充足。
- **統一卡片結構與消除突兀對比**：
  - 廢除隨意翻轉的整張深黑卡或大黃卡，所有投影片統一採用高雅純白微透光卡片（#FFFFFF，邊框 #C6DBEE）。
  - 將主視窗畫布調和為舒適的清新淡藍（#EFF6FD），卡片與畫布邊界柔和清晰、層次自然。
  - 主文字統一使用深邃海軍藍（#0A2347），文字對比清晰銳利但絕不刺眼。
  - 金黃琥珀色（#D97706 / #F59E0B）僅作為克制的小徽章與關鍵數值點綴，維持簡約高智感。

## [2026-11-24] cite-and-scale | slides/how-to-generate-future-sky-images.html | 概念易讀性重構、全域字級放大調校與 [1]~[7] 參考文獻庫建立
- **概念通俗與易讀性重構**：
  - 在 Slide 02 徹底重寫爬坡事件（Ramp Events）之物理意義（厚雲突發遮日造成的發電斷崖式暴跌）。
  - 清晰定義評估突發爬坡預警能力之綜合指標 **F1-Score（兼顧漏報與誤報的調和平均值）**，直觀說明為何傳統模型趨近於 0（100% 漏報遮日跳變）而生成式模型能突破 60%~67%（召回率 > 80%）。
- **普通字級全面放大**：正文與說明字級擴增至 `1.02rem ~ 1.05rem`，標旨句增至 `1.12rem`，表格字級增至 `1.0rem`，顯著改善遠距投影與近距閱讀體驗。
- **文獻引用體系建立**：
  - 增設第 13 頁專用文獻頁 `[1]`~`[7]`，詳列 7 篇核心論文之標題、作者、期刊會議年份與關鍵貢獻。
  - 全篇數據、公式與架構均標註可點擊跳轉之引用小標籤（如 `0.5s [2]`、`2.44 kW [3]`、`>60% [5]`、`-28.1 W/m² [6]` 等）。

## [2026-11-24] uniform-blue | slides/how-to-generate-future-sky-images.html | 嚴格統一藍色系卡片規範、修正數值折行破版、移除艱澀術語堆疊
- **嚴格統一卡片邊框與樣式（解決用戶反饋圖二）**：
  - 徹底移除第 4 頁各卡片隨機出現的藍色粗頂邊線與多色雜色徽章，4 張卡片 100% 採用相同的純淨藍色系邊框與統一高度。
  - 平衡所有卡片文字行數（統一 3 行精簡說明），卡片版面整齊劃一。
- **重構 Metric Box 排版（解決用戶反饋圖一）**：
  - 將指標數值與文獻角標綁定為 `white-space: nowrap;`，徹底消除 `[5]` 單獨掉落下一行的尷尬排版。
  - 採用左右對齊的彈性卡片容器（數值在左，文字在右），杜絕單詞懸空折行。
- **語意通俗化與去重**：
  - 移除冷僻術語（如「PhyCell 平流 PDE」、「CDM」、「GPA 偶然認知不確定性」等抽象名詞堆疊），改用大氣物理與工程直觀語言解釋。
  - 去除各頁重複解釋名詞的贅述，精煉全篇表達。
- **全域色系收斂**：全篇嚴格以清爽藍色系為主調，僅在「特別數值指標」克制點綴暖金琥珀色。

## [2026-11-24] remove-emoji | slides/how-to-generate-future-sky-images.html | 全面移除投影片中所有 Emoji 表情符號，純化學術專業質感
- 徹底清除封面、橫幅條、註記與結論區塊中出現的所有 Emoji 符號（如 🏛️、⏱️、💡、🎯、📌、⚠️、✨）。
- 改以端正純淨的學術排版標籤（如 `[文獻基礎]`、`[預測尺度]`、`破解方案：`、`演進核心規律：`、`實證註記：`、`核心結論：`）進行結構化引導，維持嚴謹自洽的高智感論文報告風格。









## [2026-11-24] presentation | slides/how-to-generate-future-sky-images.html | 完成「如何生成未來天空影像」13 頁純淨淺藍系學術互動簡報
- **主題對齊**：100% 聚焦於「未來天空影像與視訊生成（Video Nowcasting）」，剔除偏離之發電量迴歸內容。
- **基礎入門導入**：在第 02 頁建立對全天空攝影機（ASI 180° 魚眼鏡頭）、1 分鐘連續採樣與 0–30 分鐘時空生成任務的完整基礎介紹。
- **純淨淺藍系統一視覺**：
  - 畫布以柔和淺藍（#EEF6FD / #D4E3F0）為主色調，解決過往過多刺眼白底或對比過於突兀問題。
  - 卡片維持一致純白底色、統一 1.5px 淺藍邊框（#C4D9ED）與充裕呼吸內邊距（22px 24px）。
  - 數值指標盒（Metric Box）實施白底防折行（white-space: nowrap;），標籤 [2]、[5] 絕不落單換行。
  - 全篇 100% 零 Emoji，改以簡潔學術徽章與幾何標註替代。
- **文獻事實錨定**：文獻庫 [1]~[7] 完整收錄於第 13 頁，各頁數據與代表架構均提供可點擊跳轉之引用標記。

## [2026-11-24] fix | slides/how-to-generate-future-sky-images.html | 修復 LaTeX 數學公式渲染：配置 Delimiter、消除跳脫字元污染與增加離線降級
- **修復 Delimiter 預設缺失**：KaTeX `renderMathInElement` 預設不支援單錢字號 `$...$`，已顯式配置 `delimiters: [{left: '$', right: '$', display: false}, ...]`，解決 `$T$`、`$\mathbf{X}$` 等行內公式無法被識別的問題。
- **根除轉義字元污染**：修復生成腳本中反斜線遭轉義導致 `\frac` 變異為 `\x0crac`、`\nabla` 變異為換行字元、`\theta` 變異為 Tab 字元的致命語法錯誤，恢復標準正確 LaTeX 公式。
- **雙重可靠度與離線優雅降級**：新增 `fallbackRenderMath()` 引擎，若遇 CDN 阻擋、延遲或離線情境，自動啟動純 HTML/Unicode 格式化替代方案，確保公式永遠不會以壞掉的原始碼形式裸露。

## [2026-11-24] fix | slides/how-to-generate-future-sky-images.html | 修復翻頁按鍵失靈：消除腳本語法錯誤、掛載全域視窗函數與新增懸浮導航鍵
- **根除 JavaScript 語法錯誤**：修復在備用數學渲染器中由於單引號嵌套 `'JetBrains Mono'` 導致的 `Unexpected identifier` 語法錯誤，使瀏覽器恢復正常執行整份腳本。
- **全域視窗掛載 (Global Window Bindings)**：將 `nextSlide()`、`prevSlide()`、`showSlide()` 等導航函數明確綁定至 `window` 全域命名空間，確保 HTML 行內 `onclick` 能 100% 呼叫執行。
- **雙側懸浮翻頁箭頭**：在簡報畫布兩側新增現代化微圓懸浮按鈕（`‹` 與 `›`），並維持鍵盤方向鍵、空白鍵、換行鍵與頂部功能按鈕的多途徑翻頁支援。

## [2026-11-24] layout-adjust | slides/how-to-generate-future-sky-images.html | 第 13 頁參考文獻庫改為正常由上至下單欄標準列點排列
- **單欄垂直清單**：將原本雙欄交錯網格重構為由上到下單欄標準學術文獻清單（`[1]` 至 `[7]`），每列由左側標號、主標題作者與期刊簡述組成，大幅提升閱讀流暢度與縱向條線性。
- **平滑高亮維持**：保留全篇點擊引用跳轉並自動聚焦高亮（金色微光）的互動體驗。

## [2026-11-24] pedagogical-rewrite | slides/how-to-generate-future-sky-images.html | 全面降維重寫：以直覺比喻與生活化引導重塑 13 頁核心內容
- **新手友善視角重構**：全面剔除晦澀抽象的純數學符號堆疊，改以「生活化比喻 + 核心直覺 + 物理本質」三層引導：
  - 將光流法比喻為「把雲當硬紙板剪貼（無法模擬蒸發消散）」。
  - 將 ConvLSTM 均方誤差盲點比喻為「因不敢賭未來而畫成一片灰色均值塗抹」。
  - 將 SkyGPT 比喻為「將百萬像素轉化為樂高視覺字典，再用風向方程推演積木接龍」。
  - 將 Xiong et al. 比喻為「數學家算大風吹輪廓 + 藝術家（擴散模型）精雕雲朵微觀紋理」。
  - 將 SkyFusion 4 通道比喻為「給神經網絡戴上天文指南針圖層 + 嚴格專屬裁判」。
  - 將 FarSky 流匹配比喻為「直達未來的直線高鐵（最優傳輸），打破 10 分鐘邊緣模糊天花板」。
- **三問式白話評估**：第 11 頁以「看著像不像？符不符合物理？太陽畫對了沒？」三問化繁為簡。
- **嚴謹性與學術錨點保留**：專有名詞以括號附註於大白話之後，並完整保留文獻庫 [1]~[7] 事實錨點與互動跳轉。

## [2026-11-24] typography-upgrade | slides/how-to-generate-future-sky-images.html | 全面提升字體層級並解鎖滿版自適應放大
- **字級階梯全面放大**：
  - 簡報主標題由 26px 升至 30px（封面主標題由 36px 升至 42px）。
  - 副標題由 14.5px 升至 16.5px（封面副標題升至 20px）。
  - 卡片標題由 17.5px 升至 20px（網格中卡片升至 17~18.5px）。
  - 正文、解說與比喻文字全面由 12~13.5px 提升至 14~15px。
  - 標籤、徽章與附註提升至 12.5~13.5px，表格內文升至 14.5px。
- **解鎖 1.0 縮放上限**：移除過去 `autoScaleViewport()` 中 `Math.min(..., 1.0)` 的縮放封頂限制，允許在 1080p、2K 與 4K 螢幕上自適應等比例放大填滿螢幕（上限解鎖至 2.2x），徹底解決大螢幕字體偏小或留白過大的閱讀痛點。

## [2026-11-24] layout-fix | slides/how-to-generate-future-sky-images.html | 修復邊框留白不足與懸浮按鈕覆蓋文字問題
- **表格邊框內邊距修復**：補齊丟失之 `.eval-table th/td` 內邊距規則，特別對第一欄（`padding-left: 26px;`）與最後一欄（`padding-right: 26px;`）給予大間隔留白，徹底杜絕文字緊貼表格左右邊框的問題。
- **畫布外側留白拓寬**：將 `--pad-viewport` 左右邊界大幅拓寬至 `72px`，使所有投影片內容自畫布邊緣充分向內縮進。
- **懸浮翻頁鈕縮排避讓**：縮減兩側懸浮翻頁箭頭直徑（由 44px 縮減至 38px），並收斂至外側（`left: 14px;`、`right: 14px;`），與內側內容邊緣保留超過 20px 之純淨留白通道，徹底解決箭頭覆蓋文字的尷尬疊放。
- **全域卡片呼吸空間**：將 `--pad-card` 提升至 `22px 26px;`，並擴大比喻框（`.analogy-box`）與提示條（`.callout-strip`）之內襯空間。
