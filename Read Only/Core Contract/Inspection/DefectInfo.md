# DefectInfo

**定位：語言無關 × 專案無關**

本文件定義單個缺陷的描述資料模型。BoundingBox 為選填，不提供位置資訊的模型（分類、Anomaly、OCR、量測型）可為空值。

依賴：引用 Inspection/BoundingBox

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `DefectType` | 字串 | 缺陷種類名稱 |
| `Confidence` | 浮點數 | 模型對此缺陷的信心分數 |
| `BoundingBox` | `BoundingBox` 或 空值 | 缺陷位置資訊；模型不提供位置資訊時為空值 |

## 約束

- `DefectType` MUST NOT 為空字串
- `Confidence` MUST 在 `[0.0, 1.0]` 範圍內（含兩端）
