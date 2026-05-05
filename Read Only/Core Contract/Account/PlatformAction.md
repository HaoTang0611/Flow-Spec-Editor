# PlatformAction

**定位：語言無關 × 專案無關**

本文件定義平台層級可供權限檢查的操作動作枚舉。

依賴：引用 Common/UserRole

---

## 枚舉值

| 值 | 說明 |
|----|------|
| `ViewStatus` | 查看系統狀態、警報與檢測結果 |
| `ExecuteOperation` | 觸發運作型操作（如啟動 / 停止檢測） |
| `LoadProject` | 讀取專案配置（唯讀） |
| `ModifyProject` | 修改並儲存專案配置與參數 |
| `ManageAccounts` | 新增、編輯、刪除使用者帳戶及指派角色與密碼 |

## 約束

- `PlatformAction` 僅定義可被授權政策判定的動作集合，不定義各角色的固定許可矩陣
- 各角色對各動作的允許關係 MUST 由專案層級或平台層級的授權政策決定
