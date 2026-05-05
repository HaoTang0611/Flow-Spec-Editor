# LogEntry

**定位：語言無關 × 專案無關**

本文件定義單筆系統日誌條目資料模型。

依賴：引用 Logging/LogLevel、Common/Timestamp

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Timestamp` | 時間戳記 | 日誌寫入時刻，格式見 Common/Timestamp |
| `Level` | `LogLevel` | 日誌層級 |
| `Source` | 字串 或 空值 | 來源元件名稱；未知時為空值 |
| `Message` | 字串 | 事件訊息內文 |
| `ExceptionInfo` | 字串 或 空值 | 例外或錯誤細節（如堆疊追蹤、錯誤碼、原始訊息）；無例外時為空值 |

## 約束

- `Timestamp` MUST 符合 Common/Timestamp 規範的格式
- `Level` MUST 為 `LogLevel` 枚舉的合法值
- `Message` MUST NOT 為空字串
- `ExceptionInfo` 為選填；呼叫端 MUST NOT 將例外細節嵌入 `Message`，例外資訊 MUST 填入 `ExceptionInfo`
