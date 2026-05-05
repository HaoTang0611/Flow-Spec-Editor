# CameraGrabResult

**定位：語言無關 × 專案無關**

本文件定義相機擷取操作的回傳結果資料模型，供 `ICamera.GrabFrame` 使用。

依賴：引用 Common/ImageData、Camera/CameraError

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Success` | 布林 | 擷取是否成功 |
| `Frame` | ImageData 或 空值 | 成功時 MUST 填入擷取到的影像；失敗時 MUST 為空值 |
| `Error` | CameraError 或 空值 | 失敗時 MUST 填入對應錯誤碼；成功時 MUST 為空值 |
| `Detail` | 字串 或 空值 | 可選附加描述（如逾時時長、原始錯誤訊息） |

## 約束

- `Success` 為 `true` 時，`Frame` MUST 有值，`Error` MUST 為空值
- `Success` 為 `false` 時，`Frame` MUST 為空值，`Error` MUST 填入可識別的 `CameraError` 錯誤碼
- `Detail` 為選填；呼叫端 MUST NOT 依賴 `Detail` 進行邏輯判斷，僅供人工診斷使用
