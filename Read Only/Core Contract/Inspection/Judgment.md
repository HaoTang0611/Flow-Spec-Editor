# Judgment

**定位：語言無關 × 專案無關**

本文件定義影像檢測的判定結果枚舉。

---

## 枚舉值

| 值 | 說明 |
|----|------|
| `OK` | 檢測判定通過 |
| `NG` | 檢測判定不通過 |
| `Unknown` | 檢測無法可靠判定，需由人工或上層流程處理 |

## 約束

- 每次推論 MUST 產生一個 `Judgment` 值
- `Judgment` 表示本次推論的最終判定結果
- 當模型輸出信心低於對應模型設定的最低信心門檻時，`Judgment` MUST 為 `Unknown`
- `Judgment` 的產生規則（如依缺陷、分數閾值、量測值或其他決策邏輯）由模型配置與實作層決定，Core Contract 不強制其與 `InspectionResult.Defects` 建立一對一對應
- `Judgment` MUST 為 Result 層消費的唯一 OK / NG / Unknown 權威值；查詢、統計與存檔保留策略 MUST 以 `Judgment` 為準
- `InspectionResult.Defects` 為附加診斷資訊，MUST NOT 單獨取代 `Judgment` 作為結果統計或存檔判定依據
