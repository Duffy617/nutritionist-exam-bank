# 🎓 台灣營養師國考題庫 108-115年

> 由 YCL LAB 精準營養學實驗室整理，供學生免費使用

## 📚 考試科目（六科）

| 科目 | 說明 |
|------|------|
| 生理學與生物化學 | 代謝、激素、酵素 |
| 營養學 | 巨量/微量營養素 |
| 膳食療養學 | 各疾病飲食治療 |
| 公共衛生營養學 | 流行病學、介入 |
| 食品衛生與安全 | 法規、污染、標準 |
| 團體膳食設計與管理 | 供餐、菜單設計 |

## 📂 資料夾結構

```
exams/
  108/
    first/    # 第一次考試
      questions.json  # 題目
      answers.json    # 答案（含更正）
    second/   # 第二次考試
  109/ ~ 115/ ...
```

## 📋 題目格式（JSON）

```json
{
  "year": "113",
  "term": "第二次",
  "subject": "營養學",
  "question_number": 1,
  "question": "題目內容",
  "options": {
    "A": "選項A",
    "B": "選項B",
    "C": "選項C",
    "D": "選項D"
  },
  "answer": "B",
  "corrected_answer": null,
  "explanation": "AI 生成的詳細解析",
  "keywords": ["必需胺基酸", "離胺酸"],
  "difficulty": "中等",
  "source": "考選部"
}
```

## 🤖 Discord Bot 使用方式

在 Hayakawa Aoko 伺服器的 #chat-with-ai 頻道：
- `@Hermes Aoko 出一題營養學選擇題`
- `@Hermes Aoko 考我膳食療養學，難度中等`
- `@Hermes Aoko 解析 113年第2次營養學第15題`
- `@Hermes Aoko 我不懂必需胺基酸，幫我解釋`

## 📥 資料來源

考選部：https://wwwq.moex.gov.tw/exam/wFrmExamQandASearch.aspx

> ⚠️ 本題庫僅供學習參考，以考選部公告之正式答案為準
