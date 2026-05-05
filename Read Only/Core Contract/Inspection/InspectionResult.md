# InspectionResult

**定位：語言無關 × 專案無關**

本文件定義單次推論的完整結果資料模型。

依賴：引用 Inspection/Judgment、Inspection/DefectInfo、Common/Timestamp

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `ResultId` | 字串 | 結果唯一識別碼 |
| `Judgment` | `Judgment` | 本次推論的判定結果 |
| `Defects` | `DefectInfo` 列表 | 偵測到的缺陷清單 |
| `Timestamp` | 時間戳記 | 推論完成時刻，格式見 Common/Timestamp |
| `WorkOrderId` | 字串 或 空值 | 所屬工單識別碼；無工單情境（離線測試、開發驗證）時為空值 |

## 約束

- Result 層消費 `InspectionResult` 時，MUST 以 `Judgment` 作為最終判定；`Defects` 為純附加診斷資訊，與 `Judgment` 無強制綁定關係
- `Defects` 中每筆記錄 MUST 符合 `DefectInfo` 定義
- `ResultId` MUST 在同一平台實例中唯一
- `WorkOrderId` 不為空值時 MUST NOT 為空字串
