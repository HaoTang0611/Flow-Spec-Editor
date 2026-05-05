# Lighting — 索引

**定位：語言無關 × 專案無關**

本模組定義光源控制器能力契約，涵蓋連線管理、多頻道邏輯控制與參數套用。光源控制器透過通訊通道與硬體溝通，並負責將邏輯控制命令轉譯為控制器特有協定。

依賴：引用 Communication/CommunicationOptions

---

## 文件清單

| 文件 | 說明 |
|------|------|
| [ILightingController.md](ILightingController.md) | 光源控制器介面（連線、多頻道控制、配置套用） |
| [LightingOptions.md](LightingOptions.md) | 單一頻道邏輯控制參數資料模型 |
| [LightingTriggerMode.md](LightingTriggerMode.md) | 光源觸發模式枚舉（Continuous / Strobe） |
| [LightingState.md](LightingState.md) | 光源控制器連線狀態枚舉 |
| [LightingError.md](LightingError.md) | 光源控制器操作錯誤語意 |
| [LightingOperationResult.md](LightingOperationResult.md) | 光源控制器操作回傳結果資料模型 |
| [LightingBrightnessRange.md](LightingBrightnessRange.md) | 頻道亮度原生單位合法值範圍資料模型 |
