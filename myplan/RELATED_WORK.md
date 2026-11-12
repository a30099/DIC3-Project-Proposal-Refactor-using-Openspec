# 2. Related Work — AirWise（Refactored）

> 版本：2026-11-12｜以團隊 Proposal v1.1 第 2 節及 References 重新組織，維持老師要求的 **AI Technology 約 1/2 + Domain Know-how 約 1/2**。`[n]` 對應原提案編號；本版為來源導向的文獻整理，**未獨立完成每篇論文原文核驗**。

## 2.1 Review Scope and Taxonomy

AirWise 同時涉及資料取得、AQI 短期預測、LLM 工具調用、RAG、官方風險分級與個人活動決策。本章不以羅列技術名詞為終點，而按研究可回答的問題將工作分為兩群：**AI 端**解決如何查詢、預測、整合與解釋資訊；**領域端**定義何種暴露對何種使用者有風險，以及應如何在降低暴露與維持活動之間權衡。

| Research Stream | 主要任務 | 對 AirWise 的啟發 |
|---|---|---|
| AI: LLM / Agent / MCP | 解析問句、工具呼叫、提供對話介面 | Planner 不直接產生 AQI 數值 |
| AI: Forecasting / RAG | 短時預測、資料檢索、證據組合 | 預測與決策、引用需分層 |
| Domain: AQI / Health Guidance | 健康風險分級、敏感族群活動建議 | 官方規則轉成可驗證決策表 |
| Domain: Exposure / Behavior / Sensors | 活動強度、風險權衡、空間誤差 | 個人情境、替代方案、資料品質 |

## 2.2 AI Technology: Retrieval, Agents, and Forecasting

### 2.2.1 Agent and Tool-Grounded Interfaces

Gao et al. [6] 以 Instructor–Worker 多代理人架構處理野火期間的空氣品質分析與政策建議，展示 LLM 協調多步工具與報告生成的潛力；但原提案指出其研究情境依賴特定 API，且沒有直接處理個人敏感度、活動強度與未來時段的聯合決策。Pan 與 Nipu [7] 透過 Model Context Protocol（MCP）將感測資料帶入 LLM 對話介面，使回答能連結實際資料；不過「取到即時數據」尚不能等同「提供未來活動決策」。

近期室內空品警示系統 [9] 及 AirAwareTX [14] 分別將 LLM 用於感測警示與排放事件報告摘要；這些工作支持自然語言介面在環境資訊溝通上的用途，但研究場域與 AirWise 的臺灣戶外活動規劃不同。提示表示方式的比較工作 [10] 更顯示，當模型接收到明確門檻或狀態摘要時，判斷可受輸入表示法影響。AirWise 因此選擇**先由規則引擎計算或判定，再交給 LLM 解釋**，而非把 AQI 分級留給自由生成。

### 2.2.2 Forecasting and Data-Grounded Explanation

以天空影像結合視覺語言模型的研究 [8] 嘗試把空品預測和視覺解釋結合，但影像會受天候與照明影響，亦不等同臺灣測站情境的逐時 AQI 預測。EnvSentry [11] 藉由多引擎模型及 RAG 支援環境監測決策；原提案摘要指出其與專家預報一致率具潛力，但系統主要服務監測或治理情境，仍需轉換成面向一般民眾的活動建議。

臺灣 PM2.5 預測研究提供重要的在地模型比較基礎。Kow 與 Kow [12] 使用 Transformer 概念處理 PM2.5 預測；Chuang 與 Lin [13] 比較 iTransformer、PatchTST 與 LSTM+CNN，強調大尺度氣象特徵的角色。這些研究聚焦數值預測，其成果**不能直接推出對敏感族群的行動建議必然正確**。AirWise 規劃以 Persistence 與前日同時段作基準，使用 XGBoost 預測 1–24 小時 AQI，並透過獨立決策模組把模型結果轉譯成活動選項。

### 2.2.3 AI-Side Synthesis

AI 文獻共同說明：資料錨定讓 LLM 有機會查詢可信事實，時間序列模型則能推估未來條件；但**模型的預測準確**與**健康決策的正確、可追溯**是不同的研究目標。AirWise 將兩者置於明確的資料、規則與引用驗證流程，並以 E1（預測）和 E2（建議品質）分別測量。

## 2.3 Domain Know-how: Health Risk, Exposure, and Behavior

### 2.3.1 Health Burden and Taiwanese AQI Rules

*State of Global Air 2025* [1] 是原提案理解全球空污健康負擔的重要背景；但人口層級疾病負擔不能直接用來回答個別使用者是否應在某時段運動。臺灣環境部 AQI 健康影響及活動建議 [15] 才是 AirWise 擬使用的主要規則基準。官方 AQI 分級與敏感族群指引，將污染程度轉為相對應的健康提醒；然而實際套用仍需指定族群、活動強度及資料時間。

國際參考方面，紐約市衛生局的戶外空品活動建議 [16] 特別考量劇烈活動與活動時間，提醒研究者不能把 AQI 當作脫離暴露情境的唯一風險因素。AirWise 依此將 `location`、`time`、`group`、`activity`、`intensity` 視為重要欄位；但本系統對臺灣使用者的主要決策仍以臺灣主管機關版本為依據，其他國際指引只作背景與比較，不應混成單一法定門檻。

### 2.3.2 Exposure Versus Benefits of Physical Activity

Tainio et al. [17] 討論步行與騎乘等身體活動效益是否可能被污染暴露抵消，指出不宜將「有空污」直接等同「所有戶外活動都應取消」。但長期群體層級的運動效益分析不能代替高污染日、急性暴露或個別病人的專業評估。這項張力對系統設計尤其重要：建議應有**更合適時段、降低強度、改室內**等替代選項，並清楚表示不適用情況，而非作出籠統的「一定安全」承諾。

AirRater 使用者研究 [18] 提供環境資訊與行為改變的應用背景；不過對介面的偏好或主觀信心，不等於可證明健康結果改善。因此，AirWise 的 E3 設計只宣稱測量可理解性、偏好與主觀決策信心，不會把問卷結果解釋成醫療有效性。

### 2.3.3 Measurement Reliability and Spatial Representation

低成本感測器能提升空間覆蓋度，但 Feenstra et al. [19] 與 Chanapromma et al. [20] 涉及感測器量測表現及校正問題，原提案特別指出濕度可能使數值偏移。對個人活動建議而言，測站代表性、資料時效與污染物差異都可能影響判斷。AirWise 第一版規劃採官方測站，不直接納入未校正微型感測器；但距離最近測站仍不保證測值能代表使用者所在地。這是應主動揭露的系統限制。

### 2.3.4 Domain-Side Synthesis

領域文獻補上 AI 模型本身所欠缺的三項判準：**誰可能更敏感、暴露情境如何改變風險、何時應在預防原則與活動效益之間做取捨**。因此，AirWise 把這些判準明確設計為可測試的 Profile、規則與替代方案流程，而不是只交由 Prompt 自行猜測。

## 2.4 Article Summary Table

以下依原 Proposal 第 2.3 節重整；**Findings 為來源提案對該研究的摘要，尚非本次獨立驗證結果**。若正式提交，請補全標題、作者、出版資訊及可點擊 DOI／原文連結。

| Ref. | Year | Problem | AI Method | Domain Knowledge | Dataset / Context | Findings (as reported) | Limitation for AirWise |
|---|---:|---|---|---|---|---|---|
| [6] | 2025 | 野火期間空品分析 | Instructor–Worker LLM | 野火煙霧、健康警示 | 洛杉磯野火、空品 API | 能生成政策建議 | 未考慮個人族群與活動時段 |
| [7] | 2025 | 非專家解讀感測資料 | LLM + MCP | 室內 PM2.5 | IoT 感測器 | 提供資料錨定介面 | 無明確預測及規則決策 |
| [8] | 2025 | 天空影像空品預測 | CNN / VLM | 能見度與污染 | 天空影像 | 結合預測與視覺溝通 | 受天候、光線限制 |
| [9] | 2026 | 室內空品警示 | LLM + IoT | 室內暴露風險 | 室內感測情境 | 個人化警示構想 | 非戶外活動規劃 |
| [10] | 2026 | Prompt 影響空品判斷 | 本地／雲端 LLM 比較 | 門檻與狀態資訊 | 室內外空品資料 | 門檻資訊有助判斷 | 未完整生成可追溯活動建議 |
| [11] | 2026 | 環境監測支援 | 多引擎 LLM + RAG | 環境預報 | 環境監測資料 | 原提案稱具專家一致性 | 偏管理決策，非個人族群 |
| [12] | 2026 | 臺灣 PM2.5 預測 | Transformer | 污染物關聯 | 臺灣空品資料 | 預測模型比較 | 不直接生成健康行動 |
| [13] | 2026 | PM2.5 時空預測 | iTransformer / PatchTST | 大尺度氣象因素 | 臺灣測站及氣象資料 | 原提案稱 R² 約 0.86 | 模型成本及應用轉譯限制 |
| [14] | 2025 | 排放事件資訊溝通 | LLM + RAG | 化學品危害 | 德州排放報告 | 摘要與查詢 | 非一般日常 AQI 活動 |
| [1] | 2025 | 全球空污健康負擔 | 流行病學估計 | PM2.5、疾病負擔 | 全球疾病負擔資料 | 支持問題重要性 | 人口尺度，不能直接決策 |
| [17] | 2016 | 運動收益與污染風險 | 暴露效益模型 | 身體活動、空污 | 都市情境 | 探討收益風險權衡 | 非個別患者短期風險 |
| [18] | 2021 | 空品資訊與行為 | 使用者研究 | 保護行為 | AirRater App 使用者 | 資訊可能促成行為變更 | 不能推論臨床效果 |
| [19] | 2019 | 低成本感測可靠度 | 性能比較 | 感測誤差 | 空品監測站 | 感測器表現有差異 | 需要校正與品質控管 |
| [20] | 2025 | 低成本 PM2.5 校正 | 非線性校正 | 濕度／感測原理 | 實驗測量 | 校正可改善表現 | 情境外推需驗證 |

## 2.5 Three Professional Trade-offs → System Decisions

| Trade-off | 觀點 A | 觀點 B | 設計決策 | 待研究問題 |
|---|---|---|---|---|
| PM2.5 標準的嚴格程度 | 臺灣分階段調整 [3][21][22] | 參考 WHO 更嚴格目標 | 臺灣官方 AQI 規則為決策基準；比較資訊分開顯示 | 長期政策演進與版本更新 |
| 污染日是否應運動 | 運動長期效益 [17] | 敏感族群短期暴露風險 [15] | 提供替代時段／降低強度，不保證安全 | 個體化劑量與急性風險 |
| 微型感測器的使用 | 高空間解析度 [23] | 校正與濕度偏差 [19][20] | 第一版只採官方測站並顯示位置限制 | 大規模校正及測站代表性 |

## 2.6 Gap Analysis: Existing Research → Limitations → Our Project

**Existing Research.** Agent 與工具調用可提高資料可取得性，預測模型可推估未來污染狀況，RAG 可以補充來源；領域指引則規範 AQI 健康警示及活動調整方式。

**Limitations.** (i) 單純預測模型並不產生可稽核的族群化建議；(ii) LLM 空品對話介面未必處理未來時段、活動強度與跨級不確定性；(iii) 健康指引若只存在於文件檢索而非明確規則，仍可能造成解釋不一致。

**Research Gap.** 原提案指出，尚缺乏一個同時結合**臺灣在地測站與氣象資料、未來 AQI 預測、不確定性處理、個人敏感度、活動情境、官方規則與來源稽核**的整合式 Agent，並對預測與建議品質做分離、可量化的評測。這是本專題**擬驗證的研究缺口主張**，不是已透過系統性文獻回顧證明全球完全沒有類似研究。

**Our Project.** AirWise 以 ①–⑧ 八模組建立資料→預測→規則→建議→證據的閉環，E1 比較預測模型，E2 以 50 題情境集比較不同 LLM／資料／規則組合，E3 評估使用者主觀清楚度與決策信心。此設計可檢驗每一模組是否帶來增益，而不是只展示成功案例。

## 2.7 Evidence and Citation Audit Prior to Submission

1. 對 [6]–[14] 的論文逐筆核對是否為 arXiv 預印本、會議摘要或正式期刊；查找正式版本及完整作者。
2. 對 [1]、[2]、[3]、[21]、[22] 的死亡數字、PM2.5 濃度與政策階段核對**發布日期與統計年度**，避免把政策提案當成已施行制度。
3. 以環境部官方網站原文及更新日期取代轉載版本 [15]，核對 AQI 六級活動建議與例外條件。
4. 確認 [12][13] 所報告的測試切分、R² 數值及評估對象是否適用 AirWise 的逐時 AQI 任務。
5. 在實作時建立「Claim → Reference → Rule ID → Evaluation Scenario」對照，以支援系統追溯。

## References（沿用原 Proposal 編號，精簡列示）

[1] Health Effects Institute (2025), *State of Global Air 2025*.  
[3] 公視新聞網，臺灣空品標準加嚴相關報導（詳原提案）。  
[6] Gao, K. et al. (2025), *Instructor-Worker Large Language Model System for Policy Recommendation: Air Quality of 2025 LA Wildfires*, arXiv:2503.00566.  
[7] Pan, Y.-E. & Nipu, A. S. (2025), *LLM-enhanced Air Quality Monitoring Interface via Model Context Protocol*, arXiv:2511.03706.  
[8] (2025), *Forecasting and Visualizing Air Quality from Sky Images with Vision-Language Models*, arXiv:2509.15076.  
[9] (2026), *Large Language Model based air quality monitoring and localized alert generation*, arXiv:2609.17954.  
[10] (2026), *Enabling Cloud-Level Accuracy in Edge AI through IoT Data Preprocessing*, arXiv:2606.22496.  
[11] (2026), *EnvSentry*, *Applied Sciences*, 16(13), 6557 (原提案資訊)。  
[12] Kow, P.-Y. & Kow, P.-E. (2026), *Transformers for Air Quality*, EGU26-1459.  
[13] Chuang, Y. & Lin, Y.-C. (2026), *Spatiotemporal Prediction of PM2.5 in Taiwan*, EGU26-6363.  
[14] UC Berkeley School of Information (2025), *AirAwareTX* capstone.  
[15] 環境部空氣品質監測網，AQI 健康影響及活動建議。  
[16] NYC Department of Health，戶外空品健康建議。  
[17] Tainio et al. (2016), *Can Air Pollution Negate the Health Benefits of Cycling and Walking?*, *Preventive Medicine*.  
[18] Workman et al. (2021), AirRater App user study, *IJERPH*, 18(7), 3591.  
[19] Feenstra et al. (2019), Low-Cost PM2.5 Sensor Evaluation, *Atmospheric Environment*, 216, 116946.  
[20] Chanapromma et al. (2025), Low-Cost PM2.5 Sensor Pre-Calibration, *Engineering and Technology Horizons*, 42(4).  
[21] 公視新聞網（2024），PM2.5 標準與平均餘命報導。  
[22] 公視新聞網，環境部 PM2.5 分階段調降報導。  
[23] 國網中心資料集平台，環境部微型感測器資料集。

> 以上仍為**原提案參考文獻的整理版**，保留其編號供對照；正式論文請由組員核實後統一為 IEEE reference style。
