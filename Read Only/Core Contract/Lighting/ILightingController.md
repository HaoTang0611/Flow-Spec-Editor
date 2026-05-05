# ILightingController

**定位：語言無關 × 專案無關**

本文件定義光源控制器介面，提供連線管理、頻道參數套用與明滅控制能力。光源控制器負責將邏輯控制命令（亮度、觸發模式）轉譯為控制器特有通訊協定，並透過底層通訊通道送達硬體。

依賴：引用 Communication/CommunicationOptions、Lighting/LightingState、Lighting/LightingOptions、Lighting/LightingOperationResult、Lighting/LightingBrightnessRange、Lighting/LightingError

---

## 生命週期序列

```
Connect → (ApplyOptions / TurnOn / TurnOff / ...) → Disconnect
```

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Connect` | `CommunicationOptions` | `LightingOperationResult` | 建立與光源控制器的通訊連線；連線已存在時 MUST 回傳成功而不重複連線 |
| `Disconnect` | 無 | 無 | 中斷通訊連線；未連線時呼叫 MUST 靜默完成 |
| `ApplyOptions` | `channelId`（整數）、`options`（`LightingOptions`） | `LightingOperationResult` | 將指定頻道的邏輯控制參數轉譯並套用至硬體；控制器未連線時 MUST 回傳失敗 |
| `TurnOn` | `channelId`（整數） | `LightingOperationResult` | 啟用指定頻道發光；在 `Strobe` 模式下表示允許該頻道響應觸發訊號 |
| `TurnOff` | `channelId`（整數） | `LightingOperationResult` | 停止指定頻道發光；在 `Strobe` 模式下表示禁止該頻道響應觸發訊號 |
| `GetBrightnessRange` | `channelId`（整數） | `LightingBrightnessRange 或 空值` | 回傳指定頻道已知的亮度原生單位合法範圍；`channelId` 不合法或無已知範圍時 MUST 回傳空值 |
| `GetState` | 無 | `LightingState` | 回傳目前控制器連線狀態；任何生命週期階段皆可呼叫 |

## 事件 / 回呼

| 事件名稱 | 資料 | 觸發時機 |
|---------|------|---------|
| `OnConnectionLost` | `LightingOperationResult` | 通訊連線意外中斷時；`Error` MUST 填入對應錯誤碼 |

## 約束

- 每個 `ILightingController` 實例 MUST 對應一台物理光源控制器
- `ApplyOptions`、`TurnOn`、`TurnOff` MUST 在 `Connect` 成功後方可呼叫；否則 MUST 回傳 `LightingError.NotConnected` 錯誤
- `channelId` MUST 為目標控制器支援的合法頻道識別碼；傳入不合法識別碼 MUST 回傳 `LightingError.InvalidChannelId` 錯誤
- `ApplyOptions` 為唯一邏輯參數套用入口；呼叫端 MUST NOT 繞過此介面直接對控制器通訊
- `GetBrightnessRange` 回傳的範圍來源 MUST 由外部定義；Core Contract 不規定其具體來源或儲存機制
- 外部定義的亮度範圍 MUST 可解析為 `LightingBrightnessRange`
- Core Contract 不要求控制器硬體提供即時查詢能力
- 呼叫端 MUST NOT 假設 `GetBrightnessRange` 會觸發硬體通訊
- `Disconnect` 為清理型操作，不回傳 `LightingOperationResult`；操作細節僅可由實作層記錄
