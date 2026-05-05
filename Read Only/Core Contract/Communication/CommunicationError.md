# CommunicationError

**定位：語言無關 × 專案無關**

本文件定義通訊操作錯誤識別碼枚舉，描述 `ICommunicationChannel` 各操作可能的失敗原因。

---

## 錯誤識別碼

| 識別碼 | 觸發條件 |
|--------|---------|
| `ConnectionFailed` | `Connect` 呼叫後無法建立連線（裝置未找到、拒絕連線、位址無效等） |
| `ConnectionTimeout` | `Connect` 在超時時間內未能完成連線 |
| `SendFailed` | `Send` 呼叫時資料傳送失敗 |
| `ReceiveTimeout` | `Receive` 在指定超時時間內未收到任何資料 |
| `ConnectionLost` | 已連線狀態下連線意外中斷 |
| `OptionsInvalid` | `Connect` 傳入的 `CommunicationOptions` 內容不合法 |
| `NotConnected` | 呼叫需要連線狀態的操作時，通道尚未連線 |
| `ReceiveFailed` | `Receive` 呼叫時發生非逾時且非連線中斷的接收失敗 |

## 約束

- `Connect` 與 `Send` 操作失敗時，回傳之 `CommunicationOperationResult.Error` MUST 為本文件定義的合法識別碼之一
- `Receive` 操作失敗時，回傳之 `CommunicationReceiveResult.Error` MUST 為本文件定義的合法識別碼之一
- `OnConnectionLost` 事件 MUST 附帶本文件定義的 `CommunicationError` 識別碼，且 MUST 為 `ConnectionLost`
