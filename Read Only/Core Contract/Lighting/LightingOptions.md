# LightingOptions

**定位：語言無關 × 專案無關**

本文件定義單一光源頻道的邏輯控制參數資料模型。通訊連線設定不屬於本模型的範疇，由 `ILightingController.Connect` 的入參承載。

依賴：引用 Lighting/LightingTriggerMode

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Brightness` | 整數 | 亮度值（控制器原生單位；合法範圍由外部定義，可透過 `ILightingController.GetBrightnessRange` 查詢） |
| `TriggerMode` | `LightingTriggerMode` | 光源觸發模式 |
| `Delay` | 整數 或 空值 | 觸發延遲時間（毫秒）；僅 `Strobe` 模式下有意義，`Continuous` 模式下 MUST 忽略 |

## 約束

- `Brightness` MUST 為非負整數
- 當 `ILightingController.GetBrightnessRange(channelId)` 可取得範圍時，`Brightness` MUST 落在其回傳的 `Min`–`Max` 範圍內（含邊界）；超出範圍 MUST 導致 `ApplyOptions` 回傳 `ApplyOptionsFailed`
- `TriggerMode` MUST 為 `LightingTriggerMode` 的合法值
- `Delay` 為選填；`TriggerMode` 為 `Continuous` 時，`Delay` MUST 被忽略
