# PDF 處理腳本說明

## 使用方式
1. 從考選部下載 PDF（試題 + 答案 + 更正答案）
2. 上傳到這個對話視窗給 Claude
3. Claude 自動解析並推送到此 Repository

## 檔案命名規則
- 試題：`{年份}_{次別}_{科目}_questions.json`
- 答案：`{年份}_{次別}_{科目}_answers.json`

## 年份說明
- 108 年 = 2019 年
- 115 年 = 2026 年

## 更正答案處理
若有更正答案，`corrected_answer` 欄位會填入更正後的正確答案。
Bot 查詢時永遠以 `corrected_answer` 優先，若為 null 則用 `answer`。
