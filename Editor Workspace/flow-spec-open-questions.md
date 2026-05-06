本文件屬於**編輯者工作層**，不屬於 `Flow Spec/` 正式輸出。

# Flow Spec — Open Questions

未解決前 MUST NOT 將相關內容寫入正式規格。

---

## 處理原則

1. 上游規格（Core Contract）缺乏明確定義，無法推導出業務流程行為時，列為待處理。
2. 業務場景有多種合理解釋，需確認本專案實際需求時，列為待處理。
3. 答案確認後，說明依據並更新為「已解決」；確認屬下游決策則更新為「已關閉」。
4. 已解決的議題仍保留，作為決策依據。

---

## 待處理

| # | 議題 | 影響的文件 |
|---|------|----------|
| OQ-05 | IO Input 通道是否作為取像觸發來源（`Hardware` 模式下）？若是，Flow Spec 需描述 IO 輸入信號觸發取像流程的序列。 | `flows/Inspection.md` |
| OQ-06 | `AlarmLevel.Error` 時業務流程「暫停作業」的語意是否包含：正在進行的推論允許完成（軟暫停）、還是立即中止（硬暫停）？ | `flows/AlarmHandling.md`、`cross-cutting-policy.md` |
| OQ-07 | 系統是否支援多個 `InspectionPipeline` 並行執行，還是一次只執行一個流程？這影響 `flows/Inspection.md` 的狀態設計。 | `flows/Inspection.md` |
| OQ-08 | 批次推論（`IBatchInspector`）期間是否允許即時單次推論（`IInspector`）並行執行？ | `flows/BatchInspection.md`、`flows/Inspection.md` |
| OQ-09 | 多相機環境下，是否需要定義多相機協調取像的流程（如：同步觸發、順序觸發）？ | `flows/Inspection.md` |
| OQ-10 | 專案切換時，是否需要先完成當前工單再切換，還是可以隨時切換（未完成工單的處理策略）？ | `flows/ProjectLoad.md`、`flows/WorkOrder.md` |

---

## 已解決

| # | 議題 | 決策依據 | 解決文件 |
|---|------|---------|---------|
| OQ-01 | `InspectionPipelineConfiguration` 的相機/前處理/模型組合規則 | Flow Spec 不描述組合規則；InspectionPipeline 以名稱識別，具體組合關係由 Project Implementation 維護，Core Contract 已明確聲明 | `flows/Inspection.md` 邊界規則 |
| OQ-02 | 工單建立時機（手動/通訊/自動） | 屬於 Project Implementation 的觸發條件決策；Flow Spec 只定義建立後的生命週期（`Idle` ↔ `WorkOrderActive`）| `flows/WorkOrder.md` 邊界規則 |
| OQ-03 | 存檔動作的責任邊界 | Flow Spec 定義「在何種 Judgment 下應存檔」的判定條件；原始影像的實際寫入由 Project Implementation 負責 | `flows/Inspection.md` 步驟 5-b |
| OQ-04 | IO 輸出通道映射邏輯的歸屬 | Flow Spec 描述「依 OutputChannelMapping 對對應通道輸出 Judgment 結果」；具體通道對應規則由 Project Implementation 定義 | `flows/Inspection.md` 步驟 5-a |

---

## 已關閉

（目前無已關閉項目，制定過程中填入）
