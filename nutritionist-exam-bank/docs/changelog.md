# Changelog

## [0.1.0] — 2026-04-15

### Added
- 首次發布：115 年第一次考試 280 題（6 科），schema v2.0-dual-layer。
- 目錄結構 `exams/{year}/{session}/questions.json`。
- Schema 雙層設計：頂層 5 欄相容舊 hermes-agent 提示；同層豐富欄位供 RAG/Bloom 分析。
- README、LICENSE (CC BY-NC-SA 4.0)、科目代碼文件、CI 驗證腳本。

### Fixed
- 下標漂移：SUBJ02 Q1、SUBJ07 Q39、SUBJ08 Q04/05/07 的維生素 B/D 下標（共 7 處）。

### Known
- `classification.primary_topic / sub_topic / bloom_level` 仍為 null（待 LLM enrichment）。
- `explanation` 仍為空字串（待 LLM enrichment + 專家審校）。
- `quality_flag` 目前為 `subscript_fixed`，尚未到 `llm_enriched`。
