---
name: git-flow-zh
description: 互動式 Git 完整工作流程，自動使用主線分支，逐步引導 stash 衝突處理、提交訊息產生、提交與推送。使用 /git-flow-zh 進入此流程。
allowed-tools:
  - Bash
---

# git-flow-zh：互動式 Git 工作流程

全程使用繁體中文。每個需要使用者選擇或確認的步驟，都必須先停下來等待使用者回覆後再繼續。不要使用 `AskUserQuestion`；直接用純文字列出選項並等待回覆。

---

## 流程

### 1. 自動使用主線分支

執行：

```bash
git branch --show-current
git branch
```

顯示目前所在分支與所有分支清單，但不要詢問使用者是否切換分支。

主線分支判定規則：

- 若存在 `main`，主線分支為 `main`。
- 若不存在 `main` 但存在 `master`，主線分支為 `master`。
- 若兩者都不存在，停止流程並告知使用者找不到主線分支。

若目前已在主線分支，直接進入新增檔案步驟。

若目前不在主線分支：

1. 執行 `git status` 檢查是否有未提交變更。
2. 若有變更，執行 `git stash push -u -m "stash-<timestamp>"`，並告知使用者「已暫存目前的變更」。
3. 執行 `git checkout <主線分支>`。
4. 若有 stash，執行 `git stash pop` 嘗試恢復。

若恢復成功，告知「已恢復變更到主線分支」，繼續新增檔案步驟。

若恢復失敗且有衝突，列出衝突檔案，對每個檔案顯示主線分支版本、暫存版本與差異對比，然後詢問：

```text
請選擇處理方式：
1. 保留主線分支版本
2. 使用暫存版本
3. 手動處理
```

等待回覆後執行對應操作，再繼續新增檔案步驟。

### 2. 新增檔案

新增使用者指定的檔案；若未指定，預設使用 `.`：

```bash
git add <檔案或 .>
```

### 3. 顯示暫存狀態

執行：

```bash
git status
git diff --cached --stat
```

顯示即將提交的內容摘要。

### 4. 產生提交訊息

分析已暫存變更，依 Conventional Commits 產生繁體中文提交訊息：

```text
<type>(<scope>): <description>

[body，可選]

[footer，可選]
```

type 範圍：

| type | 用途 |
|------|------|
| `feat` | 新功能 |
| `fix` | 錯誤修正 |
| `docs` | 文件變更 |
| `style` | 格式調整，不影響邏輯 |
| `refactor` | 重構，無新功能、無修 bug |
| `test` | 新增或修改測試 |
| `chore` | 建置流程、工具設定等雜項 |

規則：

- `description` 使用繁體中文，動詞開頭，例如「新增」「修正」「更新」。
- `scope` 為受影響的模組或路徑，可省略。
- 若有 Breaking Change，在 footer 加上 `BREAKING CHANGE: <說明>`。

向使用者展示產生的訊息，詢問：

```text
確認使用此提交訊息？
1. 確認提交
2. 取消
```

等待使用者回覆。若取消，中止流程，不執行提交。

### 5. 執行提交

執行：

```bash
git commit -m "<已確認的訊息>"
```

### 6. 顯示最近紀錄並詢問是否推送

執行：

```bash
git log --oneline -5
```

顯示前五筆提交後，詢問：

```text
提交完成。是否要推送到遠端？
1. 推送
2. 略過
```

若使用者選擇推送，檢查目前分支是否有遠端追蹤分支：

```bash
git rev-parse --abbrev-ref --symbolic-full-name @{u}
```

- 若有遠端追蹤分支，執行 `git push`。
- 若無遠端追蹤分支，執行 `git push -u origin <分支名稱>`。

推送完成後顯示結果。若使用者選擇略過，告知「已略過推送，流程完成。」
