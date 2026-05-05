# CommunicationOperationResult

**定位：語言無關 × 專案無關**

本文件定義通訊操作的回傳結果資料模型，供 `ICommunicationChannel` 中的 `Connect`、`Send` 操作使用。

依賴：引用 Communication/CommunicationError

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Success` | 布林 | 操作是否成功 |
| `Error` | `CommunicationError` 或 空值 | 失敗時 MUST 填入對應錯誤碼；成功時 MUST 為空值 |
| `Detail` | 字串 或 空值 | 可選附加描述（如裝置識別、原始錯誤訊息）；呼叫端 MUST NOT 依賴此欄位進行邏輯判斷 |

## 約束

- `Success` 為 `true` 時，`Error` MUST 為空值
- `Success` 為 `false` 時，`Error` MUST 填入可識別的 `CommunicationError` 錯誤碼
- `Detail` 為選填；呼叫端 MUST NOT 依賴 `Detail` 進行邏輯判斷，僅供人工診斷使用
