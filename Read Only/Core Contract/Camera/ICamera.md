# ICamera

**定位：語言無關 × 專案無關**

本文件定義相機取像介面，提供連線管理、影像擷取與配置套用能力。

依賴：引用 Common/ImageData、Camera/CameraState、Camera/CameraOptions、Camera/CameraTriggerMode、Camera/DeviceInfo、Camera/CameraOperationResult、Camera/CameraGrabResult

---

## 生命週期序列

```
Initialize → Connect → (ApplyOptions / GrabFrame / ...) → Disconnect → Dispose
```

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Initialize` | 無 | `CameraOperationResult` | 分配驅動資源並準備相機物件；MUST 在 `Connect` 前呼叫；已初始化時重複呼叫 MUST 回傳成功而不重複初始化 |
| `Connect` | 無 | `CameraOperationResult` | 建立與相機硬體的連線；連線已存在時 MUST 回傳成功而不重複連線 |
| `Disconnect` | 無 | 無 | 中斷與相機的連線；未連線時呼叫 MUST 靜默完成 |
| `Dispose` | 無 | 無 | 釋放所有驅動資源；若仍處於連線狀態 MUST 隱式執行 `Disconnect`；MUST 靜默完成 |
| `ApplyOptions` | `CameraOptions` | `CameraOperationResult` | 套用結構化相機配置；為唯一配置入口；相機未連線時 MUST 回傳失敗並填入對應錯誤碼 |
| `GrabFrame` | 無 | `CameraGrabResult` | 擷取當前畫面；相機未連線時 MUST 回傳失敗並填入對應錯誤碼 |
| `GetDeviceInfo` | 無 | `DeviceInfo` | 回傳此相機實例的裝置識別資訊；任何生命週期階段皆可呼叫 |
| `GetState` | 無 | `CameraState` | 回傳目前相機連線狀態；任何生命週期階段皆可呼叫 |

## 事件 / 回呼

| 事件名稱 | 資料 | 觸發時機 |
|---------|------|---------|
| `OnFrameArrived` | `ImageData` | 相機產生新的影像幀時（`Continuous` 與 `Hardware` 模式為主要幀傳遞路徑） |
| `OnConnectionLost` | `CameraOperationResult` | 連線意外中斷時；`Error` MUST 填入對應錯誤碼，供呼叫端識別中斷原因 |

## 約束

- 每個 `ICamera` 實例 MUST 對應單一 `DeviceInfo.Id`；裝置選擇與實例建立由上層負責，`ICamera` 本身不接受裝置選擇輸入
- `Connect` MUST 在 `Initialize` 成功後方可呼叫
- `Connect` 成功後，相機 MUST 處於預設參數狀態；後續由呼叫端透過 `ApplyOptions` 疊加平台所需的參數
- `ApplyOptions` 為唯一配置入口；MUST 在 `Connect` 成功後方可呼叫；MUST 僅寫入 `CameraOptions` 中定義的欄位，`CameraOptions` 範圍外的參數 MUST 維持現狀不被更動
- `Disconnect` 與 `Dispose` 為清理型操作，不回傳 `CameraOperationResult`；操作失敗的細節僅可由實作層記錄，呼叫端 MUST NOT 以其回傳值作為流程判斷依據
- `Dispose` 後，除 `GetState`、`Disconnect`、`Dispose` 外，所有具有結果物件的方法 MUST 回傳 `AlreadyDisposed` 錯誤；呼叫端 MUST NOT 在 `Dispose` 後繼續使用此實例
- `GrabFrame` 與 `OnFrameArrived` 的行為語意依 `CameraOptions.TriggerMode` 而定；各模式下的主要幀傳遞路徑與合法呼叫規則詳見 `CameraTriggerMode`
