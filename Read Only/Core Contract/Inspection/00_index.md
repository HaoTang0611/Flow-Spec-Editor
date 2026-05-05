# Inspection — 索引

**定位：語言無關 × 專案無關**

本模組定義 AI 推論能力契約，包含即時推論與離線批次推論介面。

依賴：引用 Common/ImageData、Common/Timestamp、Project/ProjectModelConfiguration

---

## 文件清單

| 文件 | 說明 |
|------|------|
| [IInspector.md](IInspector.md) | AI 推論介面 |
| [IBatchInspector.md](IBatchInspector.md) | 離線批次推論介面 |
| [InspectionInput.md](InspectionInput.md) | 推論輸入資料模型 |
| [InspectionResult.md](InspectionResult.md) | 推論結果資料模型（含判定與缺陷列表） |
| [DefectInfo.md](DefectInfo.md) | 單個缺陷描述資料模型 |
| [BoundingBox.md](BoundingBox.md) | 缺陷位置矩形邊界框資料模型 |
| [Judgment.md](Judgment.md) | 檢測判定枚舉（OK / NG / Unknown） |
| [InspectionOperationResult.md](InspectionOperationResult.md) | Inspection 模組操作結果型別（Success / Error / Detail） |
| [InspectionInvokeResult.md](InspectionInvokeResult.md) | `IInspector.Inspect` 操作結果型別（含推論結果欄位） |
| [InspectionError.md](InspectionError.md) | Inspection 模組操作錯誤語意（推論與批次） |
| [BatchProgress.md](BatchProgress.md) | 批次推論執行進度資料模型 |
| [BatchFailureInfo.md](BatchFailureInfo.md) | 批次推論失敗事件資料模型 |
