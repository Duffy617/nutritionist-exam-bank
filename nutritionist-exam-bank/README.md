# Taiwan Nutritionist National Exam QBank

臺灣專門職業及技術人員高等考試**營養師**歷屆試題結構化題庫。

- **來源**：考選部公開題目與答案 PDF。
- **授權**：題目原文為中華民國考選部公開資料，屬公共資訊；本專案的**結構化標記、分類、解析**採 [CC BY-NC-SA 4.0](LICENSE)。
- **用途**：供學習、研究、RAG 與 AI 助教使用。禁止商業販售。

## 目錄結構

```
exams/
  115/first/questions.json        # 115 年第一次試題（280 題，6 科）
  115/first/meta.json             # 該場次精簡 metadata（由 CI 從 questions.json 派生）
  114/first/questions.json        # ...
  ...
  91/second/questions.json
schema/
  questions.schema.json           # JSON Schema Draft 2020-12
docs/
  subject_codes.md                # 科目代碼對照
  changelog.md                    # 版本紀錄
```

每個 `questions.json` 為單一物件：

```jsonc
{
  "meta": {
    "year": 115,
    "session": 1,
    "session_label": "first",
    "total_questions": 280,
    "subject_counts": { "SUBJ02": 50, "SUBJ03": 40, ... },
    "schema_version": "2.0-dual-layer",
    "license": "CC-BY-NC-SA-4.0"
  },
  "questions": [
    {
      // ── 頂層 5 欄：相容舊 hermes-agent system prompt ──
      "question": "...",
      "options": { "A": "...", "B": "...", "C": "...", "D": "..." },
      "answer": "B",
      "corrected_answer": null,
      "explanation": "",

      // ── 豐富 metadata：供 RAG / Bloom 分析 ──
      "id": "營養師_115-1_SUBJ02_01",
      "year": 115, "session": 1, "session_label": "first",
      "subject_code": "SUBJ02", "subject_name": "公共衛生營養學",
      "new_curriculum": true,
      "question_number": 1, "question_type": "single_choice",
      "score": 1.25, "negative_marking": false,
      "classification": {
        "primary_topic": null, "sub_topic": null,
        "keywords": [], "bloom_level": null, "difficulty": null
      },
      "explanation_detail": {
        "correct_rationale": null, "distractor_analysis": {},
        "references": [], "related_concepts": []
      },
      "rag": {
        "chunk_text": "題目：...\n(A) ...\n(B) ...\n正解：B",
        "version": "1.0",
        "last_updated": "2026-04-15",
        "quality_flag": "auto_parsed"
      },
      "source_pdf": "公共衛生營養學 Ｑ.pdf"
    }
  ]
}
```

### 為何採雙層 schema？

Taiwan Nutritionist QBank v1 用扁平 5 欄，適合簡單 curl 取題；但無法支援 RAG、Bloom 分析、跨年度趨勢。v2 做折衷：

- **頂層 5 欄**：`question / options / answer / corrected_answer / explanation` → 相容既有下游 agent，切版不用改下游程式。
- **同層豐富欄位**：classification、rag.chunk_text、explanation_detail 等 → 未來 RAG embedding、Bloom 分析直接從這些欄位讀。

## 科目代碼

| Code | 名稱 | 制度 |
|------|------|------|
| SUBJ01 | 生物化學與營養學 | **舊制**（≤113 年） |
| SUBJ02 | 公共衛生營養學 | 共通 |
| SUBJ03 | 膳食療養學 | 共通 |
| SUBJ04 | 團體膳食設計與管理 | 共通 |
| SUBJ05 | 食品衛生與安全 | 共通 |
| SUBJ06 | 食品學 | **舊制**（≤113 年） |
| SUBJ07 | 生理學與生物化學 | **新制**（≥114 年） |
| SUBJ08 | 營養學 | **新制**（≥114 年） |

114 年起考選部將 SUBJ01 拆分為 SUBJ07 + SUBJ08，同時刪除 SUBJ06 食品學。詳見 [docs/subject_codes.md](docs/subject_codes.md)。

## quality_flag 生命週期

| flag | 意義 |
|------|------|
| `auto_parsed` | PDF 自動解析，尚未人工審查 |
| `subscript_fixed` | 自動解析 + 手動修補下標漂移（見 fix_*.py） |
| `llm_enriched` | LLM 補齊 classification / explanation |
| `expert_reviewed` | 專家（作者或助教）審校確認 |

目前 **115-1 為 `subscript_fixed`**，其他欄位待 LLM 補。

## 使用範例

### Python
```python
import urllib.request, json
url = "https://raw.githubusercontent.com/Duffy617/nutritionist-exam-bank/main/exams/115/first/questions.json"
data = json.loads(urllib.request.urlopen(url).read())
print(f"{data['meta']['year']}-{data['meta']['session']}: {data['meta']['total_questions']} 題")
print(data['questions'][0]['question'])
```

### curl
```bash
curl -s https://raw.githubusercontent.com/Duffy617/nutritionist-exam-bank/main/exams/115/first/questions.json | jq '.questions[0]'
```

## 貢獻

- 發現題目錯字、缺字、下標漂移 → 開 issue，附 `id` 與應有內容。
- 補 `classification` / `explanation` → PR（或先送 issue 討論分類系統）。
- 送分題 / 一題多解 → 改 `corrected_answer`（維持 `answer` 原樣），並在 `explanation` 說明考選部公告。

## 引用

若本題庫用於學術研究，請引用：

> Liaw, Y.C. (2026). Taiwan Nutritionist National Exam QBank [Data set]. GitHub. https://github.com/Duffy617/nutritionist-exam-bank

（未來發布 Zenodo DOI 後更新此段。）

## 授權

原始題目：中華民國考選部公開資料（公共資訊）。
結構化標記、解析、分類：**CC BY-NC-SA 4.0**。詳見 [LICENSE](LICENSE)。
