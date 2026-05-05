# CameraOperationResult

**定位：語言無關 × 專案無關**

本文件定義相機操作的回傳結果資料模型，供 `ICamera` 中不回傳影像的操作使用（如 `Initialize`、`Connect`、`ApplyOptions`）。

依賴：引用 Camera/CameraError

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Success` | 布林 | 操作是否成功 |
| `Error` | CameraError 或 空值 | 失敗時 MUST 填入對應錯誤碼；成功時 MUST 為空值 |
| `Detail` | 字串 或 空值 | 可選附加描述（如裝置名稱、參數名稱、原始錯誤訊息） |

## 約束

- `Success` 為 `true` 時，`Error` MUST 為空值
- `Success` 為 `false` 時，`Error` MUST 填入可識別的 `CameraError` 錯誤碼
- `Detail` 為選填；呼叫端 MUST NOT 依賴 `Detail` 進行邏輯判斷，僅供人工診斷使用
