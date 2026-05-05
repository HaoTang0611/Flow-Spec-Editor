# DefectStatistics

**定位：語言無關 × 專案無關**

本文件定義單一缺陷種類的彙總統計資料模型。

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `DefectType` | 字串 | 缺陷種類名稱 |
| `Count` | 整數 | 此缺陷種類的累積數量 |
| `Percentage` | 浮點數 | 此缺陷種類的 `Count` 佔同一統計列表中所有 `Count` 總和的比例 |

## 約束

- `DefectType` MUST NOT 為空字串
- `Count` MUST 大於等於 0
- `Percentage` MUST 等於此缺陷種類的 `Count` 佔同一統計列表中所有 `Count` 總和的比例
- `Percentage` MUST 在 `[0.0, 1.0]` 範圍內（含兩端）
- 同一統計列表非空時，所有 `Percentage` 之和 MUST 等於 1.0
