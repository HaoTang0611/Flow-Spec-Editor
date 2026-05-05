---
name: git-flow-zh
description: 互動式 Git 完整工作流程。當使用者提到 git-flow-zh、$git-flow-zh，或要求以繁體中文引導 Git add、commit、push 流程時使用。直接暫存變更，逐步處理提交訊息確認與推送確認。
metadata:
  short-description: 繁中互動式 Git add/commit/push 流程
---

# git-flow-zh

全程使用繁體中文。這是互動式流程：每個需要使用者選擇或確認的步驟，都必須先停下來等待使用者回覆後再繼續。直接用純文字列出選項並等待使用者以數字回覆。

## Skill Invocation

- 此 skill 由使用者提到 `git-flow-zh` 或 `$git-flow-zh` 觸發。
- 將觸發詞後方的文字視為使用者指定的檔案參數。
- 若未指定檔案，新增檔案步驟預設使用 `.`。

## 流程

### 1. 新增檔案

不檢查、不切換 Git 分支；直接在目前所在分支進行提交流程。

新增使用者指定的檔案；若未指定，預設使用 `.`：

```bash
git add <檔案或 .>
```

### 2. 顯示暫存狀態

執行：

```bash
git status
git diff --cached --stat
```

顯示即將提交的內容摘要。

### 3. 產生提交訊息

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

### 4. 執行提交

執行：

```bash
git commit -m "<已確認的訊息>"
```

### 5. 顯示最近紀錄並詢問是否推送

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
