# CameraError

**定位：語言無關 × 專案無關**

本文件定義相機操作錯誤語意，描述 `ICamera` 各操作可能的失敗原因。

---

## 錯誤類型

| 錯誤名稱 | 觸發條件 |
|---------|---------|
| `InitializationFailed` | `Initialize` 呼叫後無法分配驅動資源或載入驅動程式 |
| `NotInitialized` | 在 `Initialize` 成功前呼叫了需要初始化的操作 |
| `ConnectionFailed` | `Connect` 呼叫後無法建立連線（裝置未找到、拒絕連線等） |
| `NotConnected` | 在 `Connect` 成功前呼叫了需要連線的操作（如 `ApplyOptions`、`GrabFrame`） |
| `DeviceNotFound` | 指定相機裝置不存在或未連接至系統 |
| `AlreadyDisposed` | 在 `Dispose` 後呼叫了任何操作 |
| `GrabTimeout` | `GrabFrame` 超時無法取得影像 |
| `HardwareError` | 相機硬體回報錯誤，需排查裝置 |
| `OperationNotSupportedInMode` | 當前 `CameraTriggerMode` 下不支援此操作（例如：相機實作不支援在 `Continuous` 模式下呼叫 `GrabFrame`） |

## 約束

- 各錯誤碼 MUST 作為 `CameraOperationResult.Error` 或 `CameraGrabResult.Error` 的填入值回傳給呼叫端
- 下表說明各錯誤碼與觸發方法的對應關係：

| 錯誤名稱 | 可能觸發的方法 |
|---------|--------------|
| `InitializationFailed` | `Initialize` |
| `NotInitialized` | `Connect`（在 `Initialize` 前呼叫） |
| `ConnectionFailed` | `Connect` |
| `NotConnected` | `ApplyOptions`、`GrabFrame`（在 `Connect` 前呼叫） |
| `DeviceNotFound` | `Connect` |
| `AlreadyDisposed` | 所有具有結果物件的方法（在 `Dispose` 後呼叫；`GetState`、`Disconnect`、`Dispose` 除外） |
| `GrabTimeout` | `GrabFrame`（`Software` 模式無觸發、`Hardware` 模式無硬體訊號） |
| `HardwareError` | `Connect`、`GrabFrame`、`ApplyOptions` |
| `OperationNotSupportedInMode` | `GrabFrame`（在不支援的 `Continuous` 模式下） |

- 具體附帶資訊（如裝置名稱、參數名稱）MUST 填入對應結果物件的 `Detail` 欄位
