# ICommunicationChannel

**定位：語言無關 × 專案無關**

本文件定義協定無關的通訊通道抽象介面，提供連線管理、資料收送能力。

依賴：引用 Communication/CommunicationState、Communication/CommunicationOptions、Communication/CommunicationError、Communication/CommunicationOperationResult、Communication/CommunicationReceiveResult

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Connect` | `CommunicationOptions` | `CommunicationOperationResult` | 依指定配置建立連線；連線已存在時 MUST 回傳成功而不重複連線 |
| `Disconnect` | 無 | 無 | 中斷連線；未連線時呼叫 MUST 靜默完成 |
| `Send` | 位元組序列 | `CommunicationOperationResult` | 送出資料；未連線時 MUST 回傳失敗，`Error` MUST 為 `CommunicationError.NotConnected`；傳送過程失敗時 `Error` MUST 為 `CommunicationError.SendFailed` |
| `Receive` | 超時時間（整數，毫秒） | `CommunicationReceiveResult` | 等待並接收資料；超時、無資料、未連線或等待途中斷線時 MUST 回傳失敗並填入對應錯誤 |
| `GetState` | 無 | `CommunicationState` | 回傳目前通道連線狀態 |

## 事件 / 回呼

| 事件名稱 | 資料 | 觸發時機 |
|---------|------|---------|
| `OnDataReceived` | 位元組序列 | 通道接收到新資料時（Push 模式） |
| `OnConnectionLost` | `CommunicationError` | 連線意外中斷時，MUST 附帶對應的錯誤識別碼 |

## 約束

- `Send` 資料長度 MUST 大於 0
- `Receive` 的超時時間參數 MUST 大於等於 -1；為 `-1` 時使用 `CommunicationOptions.ReceiveTimeoutMs` 作為超時時間；為 `0` 時表示非阻塞（立即回傳）；大於 `0` 時以指定毫秒為超時時間
- 呼叫端 MUST 選擇 Push 模式（`OnDataReceived`）或 Pull 模式（`Receive`）其中之一；同一通道實例 MUST NOT 同時使用兩種接收模式
- `Receive` 超時或無資料時，MUST 回傳失敗，且 `Error` MUST 為 `CommunicationError.ReceiveTimeout`
- 呼叫 `Receive` 時若通道未連線，MUST 回傳失敗，且 `Error` MUST 為 `CommunicationError.NotConnected`
- 連線在 `Receive` 等待途中意外中斷時，`Receive` MUST 立即回傳失敗，且 `Error` MUST 為 `CommunicationError.ConnectionLost`
- 本介面不定義協定細節，協定由 `CommunicationOptions.ProtocolType` 與 `ProtocolOptions` 共同決定
