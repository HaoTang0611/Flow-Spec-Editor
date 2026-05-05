# InspectionInput

**定位：語言無關 × 專案無關**

本文件定義推論輸入資料模型，封裝執行一次推論所需的全部輸入資訊。

依賴：引用 Common/ImageData

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Image` | `ImageData` | 待推論的影像資料 |
| `WorkOrderId` | 字串 或 空值 | 當前工單識別碼，用於將結果歸屬至特定工單；離線測試、開發驗證等無工單情境時為空值 |

## 約束

- `Image` MUST NOT 為空值
- `WorkOrderId` 不為空值時 MUST NOT 為空字串
