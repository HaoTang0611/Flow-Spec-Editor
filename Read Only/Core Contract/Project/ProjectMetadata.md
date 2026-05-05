# ProjectMetadata

**定位：語言無關 × 專案無關**

本文件定義專案識別資訊資料模型，描述可唯一識別一個專案資料夾的基本屬性。

依賴：引用 Common/Timestamp

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `ProjectName` | 字串 | 專案資料夾名稱；同時作為 `IProjectProvider.Load` 的輸入識別資訊 |
| `Version` | 字串 | 專案版本號 |
| `CreatedAt` | 時間戳記 | 專案建立時刻，格式見 Common/Timestamp |
| `UpdatedAt` | 時間戳記 | 專案最後更新時刻，格式見 Common/Timestamp |

## 約束

- `ProjectName` MUST NOT 為空字串
- `ProjectName` MUST 在可用專案清單中唯一
- `UpdatedAt` MUST 不早於 `CreatedAt`
