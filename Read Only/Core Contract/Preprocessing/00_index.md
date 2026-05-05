# Preprocessing — 索引

**定位：語言無關 × 專案無關**

本模組定義影像前處理管線能力契約，提供依步驟序列對影像執行轉換的介面與相關資料模型。

依賴：引用 Common/ImageData

---

## 文件清單

| 文件 | 說明 |
|------|------|
| [IImagePreprocessor.md](IImagePreprocessor.md) | 影像前處理管線介面 |
| [PreprocessingStep.md](PreprocessingStep.md) | 單一前處理步驟資料模型（類型 + 參數） |
| [PreprocessingStepType.md](PreprocessingStepType.md) | 前處理步驟類型枚舉（含各類型參數規格） |
| [PreprocessingOperationResult.md](PreprocessingOperationResult.md) | `IImagePreprocessor.Configure` 操作結果型別 |
| [PreprocessingResult.md](PreprocessingResult.md) | `IImagePreprocessor.Process` 操作結果型別（含結果影像欄位） |
| [PreprocessingError.md](PreprocessingError.md) | Preprocessing 模組操作錯誤語意 |
