# Proposal: Refactor AirWise Proposal into Verifiable Specifications

## Why
現有 DESIGN.md 已具模組、FR1–FR9 與評估計畫，但需要把自然語言設計整理成可驗收情境，銜接 SDD 與論文章節重構。

## What Changes
- 以功能需求為起點定義 AirWise 的輸入、取得資料、預測、規則、證據與失效處理情境。
- 將預測模型、規則決策、LLM 輸出責任分離，與 DESIGN.md 保持一致。
- 連結 `docs/INTRODUCTION.md`、`docs/RELATED_WORK.md` 及測試 E1–E3。

## Scope
僅規格與文件重構，不聲稱已完成 API、模型、應用或測試。原有 DESIGN.md 保留為完整設計依據。

## Success Criteria
每個 Must requirement 可追到規格情境、模組、驗收條件與（未來）測試；論文章節可回溯到原 Proposal 來源。
