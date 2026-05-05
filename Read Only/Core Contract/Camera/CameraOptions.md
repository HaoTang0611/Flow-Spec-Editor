# CameraOptions

**定位：語言無關 × 專案無關**

本文件定義相機參數配置選項，描述可設定的拍攝參數及其合法值範圍。

依賴：引用 Camera/CameraTriggerMode

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `ExposureTimeMs` | 浮點數 | 曝光時間（毫秒） |
| `Gain` | 浮點數 | 影像增益 |
| `TriggerMode` | `CameraTriggerMode` | 觸發模式；決定 `GrabFrame` 與 `OnFrameArrived` 的主體與合法呼叫規則，詳見 `CameraTriggerMode` |

## 約束

- `ExposureTimeMs` MUST 大於 0
- `Gain` MUST 大於等於 0
- 具體相機裝置支援的參數範圍由實作層定義，本層僅定義欄位語意
