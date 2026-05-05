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
| OQ-01 | `InspectionPipelineConfiguration` 只定義流程「名稱」，相機/前處理/模型的具體組合由「外部定義」，此外部定義的來源與格式為何？Flow Spec 是否需要描述組合規則，還是這是 Project Implementation 的責任？ | `flows/Inspection.md`、`flows/Startup.md` |
| OQ-02 | 工單（WorkOrder）的建立時機：由使用者手動輸入 WorkOrderId、由上位機通訊下達、還是由系統自動產生？不同方式對應的流程觸發條件不同。 | `flows/WorkOrder.md`、`cross-cutting-policy.md` |
| OQ-03 | `StorageConfiguration` 決定是否存檔，但實際的存檔動作（將 `ImageData` 寫入磁碟）屬於哪一層責任？Flow Spec 描述「存檔時機」，Project Implementation 負責「存檔實作」，確認邊界是否正確。 | `flows/Inspection.md` |
| OQ-04 | IO 輸出（推論後送出 OK/NG 信號）的具體通道映射由 `ProjectConfiguration.OutputChannelMapping` 定義，Flow Spec 是否需要描述「依映射表決定輸出通道」的邏輯，還是這屬於 Project Implementation？ | `flows/Inspection.md` |
| OQ-05 | IO Input 通道是否作為取像觸發來源（`Hardware` 模式下）？若是，Flow Spec 需描述 IO 輸入信號觸發取像流程的序列。 | `flows/Inspection.md` |
| OQ-06 | `AlarmLevel.Error` 時業務流程「暫停作業」的語意是否包含：正在進行的推論允許完成（軟暫停）、還是立即中止（硬暫停）？ | `flows/AlarmHandling.md`、`cross-cutting-policy.md` |
| OQ-07 | 系統是否支援多個 `InspectionPipeline` 並行執行，還是一次只執行一個流程？這影響 Inspection.md 的狀態設計。 | `flows/Inspection.md` |
| OQ-08 | 批次推論（`IBatchInspector`）期間是否允許即時單次推論（`IInspector`）並行執行？ | `flows/BatchInspection.md`、`flows/Inspection.md` |
| OQ-09 | 多相機環境下，是否需要定義多相機協調取像的流程（如：同步觸發、順序觸發）？ | `flows/Inspection.md` |
| OQ-10 | 專案切換時，是否需要先完成當前工單再切換，還是可以隨時切換（未完成工單的處理策略）？ | `flows/ProjectLoad.md`、`flows/WorkOrder.md` |

---

## 已解決

（目前無已解決項目，制定過程中填入）

---

## 已關閉

（目前無已關閉項目，制定過程中填入）
