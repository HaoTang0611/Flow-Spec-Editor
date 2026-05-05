# LightingError

**定位：語言無關 × 專案無關**

本文件定義光源控制器操作錯誤語意，描述 `ILightingController` 各操作可能的失敗原因。

---

## 錯誤類型

| 錯誤名稱 | 觸發條件 |
|---------|---------|
| `ConnectionFailed` | `Connect` 呼叫後無法建立通訊連線（裝置未回應、位址錯誤等） |
| `NotConnected` | 在 `Connect` 成功前呼叫了需要連線的操作（如 `ApplyOptions`、`TurnOn`、`TurnOff`） |
| `InvalidChannelId` | 傳入的 `channelId` 不屬於目標控制器的合法頻道識別碼 |
| `ApplyOptionsFailed` | `ApplyOptions` 呼叫後控制器拒絕或無法套用指定頻道的參數 |
| `OperationFailed` | `TurnOn` 或 `TurnOff` 呼叫後控制器無法執行操作 |

## 約束

- 各錯誤碼 MUST 作為 `LightingOperationResult.Error` 的填入值回傳給呼叫端
- 下表說明各錯誤碼與觸發方法的對應關係：

| 錯誤名稱 | 可能觸發的方法 |
|---------|--------------|
| `ConnectionFailed` | `Connect` |
| `NotConnected` | `ApplyOptions`、`TurnOn`、`TurnOff`（在 `Connect` 前呼叫） |
| `InvalidChannelId` | `ApplyOptions`、`TurnOn`、`TurnOff` |
| `ApplyOptionsFailed` | `ApplyOptions` |
| `OperationFailed` | `TurnOn`、`TurnOff` |

- 具體附帶資訊（如頻道識別碼、原始錯誤訊息）MUST 填入對應結果物件的 `Detail` 欄位
