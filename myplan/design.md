# Design: AirWise Spec-Driven Workflow

## Context
以 `DESIGN.md`（2026-10-28 v0.1）為唯一目前可見的技術設計基準。研究 Proposal v1.1 補充完整 26 行 Pseudocode、5+3+10、研究背景與評估。

## Architecture Decisions
1. LLM 只解析與轉述，不產生 AQI 數值、等級或健康指引。
2. 依序通過 Query/Profile → Data → Forecast（未來時段）→ Rule/RAG → Decision → Citation Check。
3. 資料缺漏採最近可用官方測站與資料時效檢查；仍失敗時揭露不可取得，提供官方資訊。
4. 敏感族群在預測區間跨級時採較高級規則（待測試）。
5. 所有建議保留 station/time/rule ID/document reference，引用未通過則模板輸出。

## Traceability
FR1–FR2 → Agent；FR3 → tools；FR4 → forecast；FR5/FR7 → rules/decision；FR6/FR8 → output；FR9 → feedback（Could）。

## Open Questions
- 實際部署程式與套件版本尚未提供。
- 官方 AQI 規則 YAML 原文校驗尚未完成。
- 模型切分、誤差區間校準與 E2 標註作業尚待實施。
