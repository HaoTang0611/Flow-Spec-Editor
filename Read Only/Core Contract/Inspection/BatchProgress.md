# BatchProgress

**定位：語言無關 × 專案無關**

本文件定義批量推論執行進度與摘要統計資料模型。

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `ProcessedCount` | 整數 | 已處理的影像張數 |
| `TotalCount` | 整數 | 本批次影像總張數 |
| `DefectCounts` | 字串→整數 映射 | 各缺陷種類的累積出現次數，鍵為缺陷種類名稱 |
| `DetectedCounts` | 字串→整數 映射 | 各缺陷種類出現在多少張影像中，鍵為缺陷種類名稱 |

## 約束

- `ProcessedCount` MUST 在 `[0, TotalCount]` 範圍內（含兩端）
- `TotalCount` MUST 大於 0
- `ProcessedCount` MUST 大於等於 0
- `DefectCounts` 與 `DetectedCounts` 中所有整數值 MUST 大於等於 0
- 本模型僅描述批次執行中的摘要進度，MUST NOT 取代每筆 `InspectionResult` 作為歷史結果記錄來源
