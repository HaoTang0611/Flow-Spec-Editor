# ImageData

**定位：語言無關 × 專案無關**

本文件定義平台核心的影像資料型別，作為各模組間傳遞影像的統一格式。

---

## 定義

`ImageData` 是一個代表單張影像內容的資料容器。

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Width` | 整數 | 影像寬度（像素） |
| `Height` | 整數 | 影像高度（像素） |
| `Channels` | 整數 | 色彩通道數 |
| `Data` | 位元組序列 | 原始像素資料，依列優先（row-major）順序排列 |

## 約束

- `Width` MUST 大於 0
- `Height` MUST 大於 0
- `Channels` MUST 為 `1`（灰階）或 `3`（RGB）
- `Data` 長度 MUST 等於 `Width × Height × Channels`
- `ImageData` 實例 MUST 為不可變，建立後像素資料 MUST NOT 被修改
