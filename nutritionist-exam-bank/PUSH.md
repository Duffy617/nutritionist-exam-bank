# Push 到 GitHub + 修好 hermes-agent 的作戰手冊

您剛刪掉 `Duffy617/nutritionist-exam-bank`，所以 hermes-agent 目前對外無題可答。
依下列順序執行，**預計 10 分鐘內 bot 恢復服務**。

---

## 1. 在 GitHub 重建同名 repo

到 <https://github.com/new>：
- Owner：`Duffy617`（您本人帳號）
- Repository name：`nutritionist-exam-bank`（**必須完全同名**，hermes-agent prompt 才不用改）
- Description：Taiwan Nutritionist National Exam QBank
- **Public**（hermes-agent 用 curl 不帶 token，必須 public）
- **不要**勾 Initialize with README / .gitignore / license（等下本機推上去時才有）

建完拿到 URL：`https://github.com/Duffy617/nutritionist-exam-bank.git`

---

## 2. 把 Cowork 產出下載到本機

Cowork 的 outputs 資料夾裡有一個準備好的 `nutritionist-exam-bank/` 完整目錄。
您可直接點 Cowork UI 「Open folder」到 outputs 資料夾，把整個 `nutritionist-exam-bank/` 複製到您電腦上慣用位置（例如 `~/repos/nutritionist-exam-bank/`）。

目錄結構會長這樣：

```
nutritionist-exam-bank/
├── .github/workflows/validate.yml
├── .gitignore
├── LICENSE
├── PUSH.md                          ← 就是本檔
├── README.md
├── docs/
│   ├── changelog.md
│   └── subject_codes.md
├── exams/
│   └── 115/first/
│       ├── questions.json           ← 280 題
│       └── meta.json
└── schema/
    └── questions.schema.json
```

---

## 3. 本機 git push

```bash
cd ~/repos/nutritionist-exam-bank        # 您放置的路徑

git init
git branch -M main
git add .
git commit -m "feat: initial release v0.1.0 with 115-1 (280 questions, dual-layer schema)"
git remote add origin https://github.com/Duffy617/nutritionist-exam-bank.git
git push -u origin main
```

> **首次 push 會被要求登入**。建議用 `gh auth login`（GitHub CLI）或瀏覽器跳轉認證，
> 避免把 Personal Access Token 貼進終端機歷史。

push 完到 GitHub 頁面刷新，應看到檔案結構與 README 渲染。

---

## 4. 驗證 raw URL 可用

```bash
curl -s https://raw.githubusercontent.com/Duffy617/nutritionist-exam-bank/main/exams/115/first/questions.json \
  | python -c "import sys,json;d=json.load(sys.stdin);print(d['meta']['total_questions'],'題')"
# 應輸出：280 題
```

**這一步若失敗**，就算 hermes-agent 也會失敗。通常原因：repo 還是 private（改 public）、分支不是 main（改 prompt 的 URL）、檔案路徑拼錯。

---

## 5. 驗證 hermes-agent 能正確讀新 schema

您目前 `SYSTEM_PROMPT` 的關鍵句是：
> 執行 curl -s "https://raw.githubusercontent.com/Duffy617/nutritionist-exam-bank/main/exams/{年}/{first|second}/questions.json" 取得 JSON（年份108-115，first=第一次，second=第二次），解析 JSON 後選一題...

新 schema 跟舊的**差別只有兩個**：

| 層面 | 舊版 | v2 雙層 |
|------|------|---------|
| 頂層結構 | `[{...}, {...}, ...]`（題目陣列） | `{"meta": {...}, "questions": [...]}`（物件） |
| 欄位 | `question, options, answer, corrected_answer, explanation` | **同上，再加** id, subject_code, classification, rag... |

**只需要改一個地方**：告訴 agent 現在要去 `.questions` 陣列裡抽題，不是直接抽整個物件。

### 5-1 進 Zeabur 修 SYSTEM_PROMPT

到 Zeabur console → 專案 `hermes-agent` → 服務 `hermes-agent` → **Variables** → 找到 `SYSTEM_PROMPT`，改成：

```
你是「青子老師」，台灣營養師國考 AI 助教。嚴格禁止使用 browser_navigate、web_search、wget 或任何搜尋引擎取得題目，只能用 shell 指令 curl 取得題庫 JSON。題庫來源：GitHub raw URL。取題步驟：執行 curl -s "https://raw.githubusercontent.com/Duffy617/nutritionist-exam-bank/main/exams/{年}/{first|second}/questions.json" 取得 JSON（年份目前僅 115，first=第一次，second=第二次），解析 JSON 後從 .questions 陣列中選一題（可依使用者需求指定 subject_code，例如 SUBJ03 膳食療養學），只顯示 question 和 options，學生作答後公布答案（corrected_answer 不為 null 時優先於 answer）並顯示 explanation（若為空則說明「詳解整理中」）。格式：繁體中文，標明年份場次科目題號。
```

改完按 Save，服務會自動重啟（約 30–60 秒）。

### 5-2 Discord 實測

回 Discord 伺服器：
- `/問` 先抽一題
- 回答後看是否正確揭示 `answer`
- 若回應題號帶 `SUBJ07/SUBJ08`，表示它能讀到新制科目（是 115-1 新制）✅

---

## 6. 安全清理（⚠️ 非常重要）

我先前查 Zeabur 時看到 `ANTHROPIC_API_KEY`、`DISCORD_BOT_TOKEN`、`PASSWORD` 都以**明文**存在 env vars 中，且我在對話記錄裡看過完整值。請依下列動作輪替：

1. **Anthropic Console** → API Keys → 將原 key `sk-ant-api03-ziJUPI...` **revoke**，產生新 key。
2. **Discord Developer Portal** → 您的 Bot → **Reset Token**，取得新 token。
3. Zeabur → `hermes-agent` service → Variables，把新 key 更新上去。
4. （可選但推薦）把 `PASSWORD` 這個 env var 也重設成新隨機字串。
5. 這些敏感值最好改用 Zeabur 的 **Sealed Variable** 而非 Plain，避免未來 MCP/log 讀取時暴露。

**這是擦屁股動作，務必做**——就算您信任 Anthropic（Claude 的執行環境），token 的外洩風險最小化永遠是對的。

---

## 7. 下一步（等 bot 穩定之後）

- [ ] 陸續上傳歷年 PDF 給 Cowork：114、113、112...
- [ ] 每收到一個場次，我即時 parse → 新增 `exams/{年}/{first|second}/questions.json`
- [ ] Git push 新檔 → CI 跑 schema 驗證 + 自動生成 meta.json
- [ ] 待題庫達 3–5 年份規模後，啟動 LLM enrichment（03_enrich_llm.py）補 `classification` 與 `explanation`
- [ ] v1.0.0 tag → Zenodo DOI → 可引用的研究資料集

---

## 附：本次 repo 統計

- 總題數：280
- 分科：SUBJ02 (50) + SUBJ03 (40) + SUBJ04 (40) + SUBJ05 (50) + SUBJ07 (50) + SUBJ08 (50)
- 檔案大小：約 450 KB
- quality_flag：`subscript_fixed`（已修 7 處下標漂移）
- 待補：classification、explanation
- Schema 驗證：**通過** (0 errors)
