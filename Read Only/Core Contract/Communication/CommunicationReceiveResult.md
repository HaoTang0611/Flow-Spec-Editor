# CommunicationReceiveResult

**定位：語言無關 × 專案無關**

本文件定義通訊接收操作的回傳結果，讓呼叫端可區分成功接收、逾時、未連線與連線中斷。

依賴：引用 Communication/CommunicationError

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Success` | 布林 | 是否成功接收到資料 |
| `Data` | 位元組序列 或 空值 | 成功接收時的資料；`Success` 為 `false` 時 MUST 為空值 |
| `Error` | `CommunicationError` 或 空值 | 接收失敗原因；`Success` 為 `true` 時 MUST 為空值 |
| `Detail` | 字串 或 空值 | 診斷用附加說明；呼叫端 MUST NOT 依賴此欄位進行邏輯判斷 |

## 約束

- `Success` 為 `true` 時，`Data` MUST NOT 為空，`Error` MUST 為空值
- `Success` 為 `false` 時，`Data` MUST 為空值，`Error` MUST 填入合法 `CommunicationError`
