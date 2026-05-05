# IProjectProvider

**定位：語言無關 × 專案無關**

本文件定義專案配置讀取介面，提供列舉可用專案資料夾、載入專案與查詢聚合配置的能力。

依賴：引用 Project/ProjectConfiguration、Project/ProjectMetadata、Project/ProjectOperationResult、Project/ProjectError

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `ListProjects` | 無 | `ProjectMetadata` 列表 | 掃描專案根目錄並回傳所有可用專案資料夾的識別資訊清單；無可用專案時 MUST 回傳空列表 |
| `Load` | 專案名稱（字串） | `ProjectOperationResult` | 載入指定專案資料夾內的全部配置；專案名稱無效、讀取失敗或內容不符合專案配置規範時 MUST 回傳失敗並填入對應錯誤 |
| `GetMetadata` | 無 | `ProjectMetadata` 或 空值 | 回傳目前載入的專案識別資訊；未載入時 MUST 回傳空值 |
| `GetConfiguration` | 無 | `ProjectConfiguration` 或 空值 | 回傳目前載入的聚合配置（含相機、通訊、前處理設定）；未載入時 MUST 回傳空值 |
| `IsLoaded` | 無 | 布林 | 回傳是否已成功載入專案 |

## 約束

- `Load` 的專案名稱 MUST NOT 為空字串；違反時 MUST 回傳失敗，且 `Error` MUST 為 `ProjectError.ProjectNameInvalid`
- `Load` 找不到指定專案時，MUST 回傳失敗，且 `Error` MUST 為 `ProjectError.ProjectNotFound`
- `Load` 無法讀取指定專案資料夾或其配置檔案時，MUST 回傳失敗，且 `Error` MUST 為 `ProjectError.ContentReadFailed`
- `Load` 載入內容不符合配置規範時，MUST 回傳失敗，且 `Error` MUST 為 `ProjectError.ConfigurationInvalid`
- `Load` 載入內容缺少必要配置且無法建立預設值時，MUST 回傳失敗，且 `Error` MUST 為 `ProjectError.ContentIncomplete`
- `Load` 遇到可由預設值補足的缺少配置時，MUST 建立預設值並視為載入成功
- `ListProjects` MUST NOT 依賴目前是否已載入專案；無論 `IsLoaded` 狀態為何均可呼叫
- `ListProjects` 回傳的每筆資料 MUST 包含可作為 `Load` 輸入的有效專案名稱（`ProjectName`）
- 未呼叫 `Load` 或載入失敗時，`GetMetadata` 與 `GetConfiguration` MUST 回傳空值
- 本介面 MUST NOT 提供任何寫入或儲存能力（Save / Write / Persist）
- 重複呼叫 `Load` MUST 以最新載入結果取代先前狀態
