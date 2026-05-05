# InspectionStatistics

**定位：語言無關 × 專案無關**

本文件定義累積統計資訊資料模型，描述一個作業週期的整體良率與缺陷分類統計。

依賴：引用 Inspection/Judgment、Result/DefectStatistics

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `LatestJudgment` | `Judgment` 或 空值 | 最近一筆推論的判定結果；作業尚未開始時為空值 |
| `OKCount` | 整數 | 累積 OK 筆數 |
| `NGCount` | 整數 | 累積 NG 筆數 |
| `UnknownCount` | 整數 | 累積 Unknown 筆數 |
| `TotalCount` | 整數 | 累積推論總筆數 |
| `YieldRate` | 浮點數 或 空值 | 良率比例；作業尚未開始時為空值 |
| `DefectStatistics` | `DefectStatistics` 列表 | 依缺陷種類彙總的統計資訊 |

## 約束

- `TotalCount` MUST 等於 `OKCount + NGCount + UnknownCount`
- `OKCount`、`NGCount`、`UnknownCount`、`TotalCount` MUST 大於等於 0
- `TotalCount` 為 0 時，`YieldRate` MUST 為空值
- `OKCount + NGCount` 為 0 時，`YieldRate` MUST 為空值
- `OKCount + NGCount` 大於 0 時，`YieldRate` MUST 等於 `OKCount / (OKCount + NGCount)`
- `OKCount + NGCount` 大於 0 時，`YieldRate` MUST 在 `[0.0, 1.0]` 範圍內（含兩端）
- `DefectStatistics` 中的每筆資料 MUST 符合 `DefectStatistics` 定義
