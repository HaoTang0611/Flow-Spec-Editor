# Project — 索引

**定位：語言無關 × 專案無關**

本模組定義專案配置的讀取能力契約與聚合配置資料模型。ProjectConfiguration 聚合硬體設備、檢測流程識別、存檔與通道映射設定，是 Camera / Communication 等模組 Options 型別的上層依賴。

依賴：引用 Camera/CameraOptions、Communication/CommunicationOptions、Lighting/LightingOptions

---

## 文件清單

| 文件 | 說明 |
|------|------|
| [IProjectProvider.md](IProjectProvider.md) | 專案配置讀取介面 |
| [ProjectMetadata.md](ProjectMetadata.md) | 專案識別資訊資料模型 |
| [ProjectConfiguration.md](ProjectConfiguration.md) | 聚合配置資料模型（含硬體、存檔與輸出輸入設定） |
| [HardwareConfiguration.md](HardwareConfiguration.md) | 硬體設備配置聚合資料模型（多相機、通訊、光源控制器） |
| [InspectionPipelineConfiguration.md](InspectionPipelineConfiguration.md) | 單一檢測流程識別資料模型 |
| [ProjectCameraConfiguration.md](ProjectCameraConfiguration.md) | 單一相機於專案中的配置資料模型（識別碼 + 顯示名稱 + 硬體參數） |
| [ProjectLightingControllerConfiguration.md](ProjectLightingControllerConfiguration.md) | 單一光源控制器於專案中的配置資料模型 |
| [ProjectModelConfiguration.md](ProjectModelConfiguration.md) | 單一模型設定資料模型 |
| [StorageConfiguration.md](StorageConfiguration.md) | 影像存檔策略配置資料模型 |
| [ProjectError.md](ProjectError.md) | 專案配置操作錯誤語意 |
| [ProjectOperationResult.md](ProjectOperationResult.md) | 專案配置操作結果型別 |
