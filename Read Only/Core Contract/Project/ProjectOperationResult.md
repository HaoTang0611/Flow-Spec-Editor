# ProjectOperationResult

**定位：語言無關 × 專案無關**

本文件定義 Project 模組操作結果型別，供專案載入等需識別失敗原因的操作使用。

依賴：引用 Project/ProjectError

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Success` | 布林 | 操作是否成功 |
| `Error` | `ProjectError` 或 空值 | 失敗原因；`Success` 為 `true` 時 MUST 為空值 |
| `Detail` | 字串 或 空值 | 診斷用附加說明；呼叫端 MUST NOT 依賴此欄位進行邏輯判斷 |

## 約束

- `Success` 為 `true` 時，`Error` MUST 為空值
- `Success` 為 `false` 時，`Error` MUST 填入 `ProjectError` 中定義的合法錯誤碼
