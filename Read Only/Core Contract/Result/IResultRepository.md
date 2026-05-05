# IResultRepository

**定位：語言無關 × 專案無關**

本文件定義檢測結果查詢介面，提供已保存 `InspectionResult` 歷史紀錄、關聯影像、工單清單與統計資訊的讀取能力。

依賴：引用 Inspection/InspectionResult、Result/QueryCondition、Result/WorkOrder、Result/InspectionStatistics、Result/DefectStatistics、Common/ImageData

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Query` | `QueryCondition` | `InspectionResult` 列表 | 依條件查詢歷史紀錄；條件欄位全為空值時 MUST 回傳全部紀錄 |
| `GetById` | 結果識別碼（字串） | `InspectionResult` 或 空值 | 依識別碼取得單筆紀錄；不存在時 MUST 回傳空值 |
| `GetImageById` | 結果識別碼（字串） | `ImageData` 或 空值 | 依識別碼取得該筆結果關聯的歷史影像；結果不存在或未保留影像時 MUST 回傳空值 |
| `Count` | `QueryCondition` | 整數 | 回傳符合條件的紀錄總數，不回傳資料本體 |
| `GetStatistics` | `QueryCondition` | `InspectionStatistics` | 回傳符合條件的結果集統計資訊；條件欄位全為空值時計算全部紀錄 |
| `ListWorkOrders` | 無 | `WorkOrder` 列表 | 回傳所有結果紀錄中出現過的工單清單；無結果紀錄時 MUST 回傳空列表 |

## 約束

- `Query` 回傳結果 MUST 依 `Timestamp` 遞減排序（最新在前）
- `Count` MUST 忽略 `QueryCondition` 中的 `PageIndex` 與 `PageSize`，回傳符合篩選條件的完整筆數
- `GetStatistics` MUST 忽略 `QueryCondition` 中的 `PageIndex` 與 `PageSize`，統計範圍為符合篩選條件的完整結果集
- `GetStatistics` 的缺陷統計（`DefectStatistics`）計算範圍為上述結果集中所有 `InspectionResult.Defects`；跨模型的加總或去重規則由實作層定義
- `GetStatistics` 回傳的 `TotalCount` MUST 與相同條件下 `Count` 的回傳值一致
- 本介面回傳的歷史資料與統計 MUST 以已保存的 `InspectionResult` 紀錄為準，MUST NOT 由執行期間的摘要進度資料推導
- `ListWorkOrders` 回傳的每筆 `WorkOrder` MUST 在 `IResultRepository` 中至少有一筆對應的結果紀錄
- 查詢成功但無符合資料時，`Query` MUST 回傳空列表，`Count` MUST 回傳 `0`，`ListWorkOrders` MUST 回傳空列表
- 查詢成功但找不到指定結果時，`GetById` MUST 回傳空值
- 查詢成功但找不到指定結果或該結果未保留影像時，`GetImageById` MUST 回傳空值
- 查詢成功但無符合資料時，`GetStatistics` MUST 回傳 `TotalCount`、`OKCount`、`NGCount`、`UnknownCount` 皆為 `0`，且 `LatestJudgment` 與 `YieldRate` 皆為空值的 `InspectionStatistics`
- 查詢來源不可用或無法讀取時，各方法 MUST 回傳該方法無符合資料時的回傳值，且 MUST NOT 回傳不完整或無法滿足對應資料模型約束的資料
