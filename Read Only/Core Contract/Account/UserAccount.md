# UserAccount

**定位：語言無關 × 專案無關**

本文件定義平台使用者帳號資料模型。

依賴：引用 Common/UserRole

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `AccountName` | 字串 | 使用者帳號名稱 |
| `Role` | `UserRole` | 使用者角色 |

## 約束

- `AccountName` MUST NOT 為空字串
- `Role` MUST 為 `UserRole` 的合法值
