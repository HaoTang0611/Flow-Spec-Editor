# PreprocessingStep

**定位：語言無關 × 專案無關**

本文件定義單一影像前處理步驟資料模型，將步驟類型與其所需參數封裝為一個完整配置單元。

依賴：引用 Preprocessing/PreprocessingStepType

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Type` | `PreprocessingStepType` | 前處理步驟類型 |
| `Parameters` | 字串→字串 映射 | 該步驟所需的具名參數；無參數需求時為空映射 |

## 約束

- `Type` MUST 為 `PreprocessingStepType` 的合法值
- `Parameters` 的鍵 MUST NOT 為空字串
- `Parameters` 所需的鍵集合由各 `PreprocessingStepType` 的規格定義；缺少必要鍵時 MUST 視為無效配置
