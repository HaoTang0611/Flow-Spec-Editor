本文件屬於**編輯者工作層**，不屬於 `Flow Spec/` 正式輸出。

# Flow Spec — 概念盤點

本文件盤點 Core Contract 各模組概念在 Flow Spec 中的角色與責任邊界。

欄位說明：
- **流程相關性**：高 = 直接出現於流程序列或狀態轉移；中 = 流程條件或錯誤回應；低 = 配置依據，不描述流程
- **Flow Spec 角色**：此概念在業務流程中的用途
- **邊界備註**：Flow Spec 應描述什麼、不應描述什麼

---

## 跨模組治理主題

需在 `cross-cutting-policy.md` 統一說明：

- **工單語意**：`WorkOrderId` 何時建立、何時結束、何時為空值（離線測試 vs. 正式作業）
- **警報觸發規則**：哪些事件應觸發 `IAlarmReporter.Raise`，各 `AlarmLevel` 的業務回應策略
- **日誌寫入規則**：哪些流程節點 MUST 寫入日誌、使用哪個 `LogLevel`
- **硬體錯誤回應通則**：相機/通訊/光源控制器錯誤時的共同業務回應策略

---

## Common 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `ImageData` | 資料模型 | 高 | 流程中影像在取像→前處理→推論階段的傳遞媒介，描述影像傳遞的時機節點 | Flow Spec 不描述 `ImageData` 的內部格式（Width/Height/Channels），只描述「影像在哪個步驟被傳遞」 |
| `Timestamp` | 格式規範 | 低 | 結果、工單、日誌等事件時間點的格式依據 | Flow Spec 引用「記錄時刻」時，說明時間戳記格式遵循 Core Contract，不重複定義格式 |
| `UserRole` | 枚舉 | 高 | 各業務操作的前置權限條件（如：`ModifyProject` 需 Engineer 以上角色） | Flow Spec 描述「哪個操作需要哪個角色」；不描述角色授權的實現機制 |
| `SystemStatus` | 枚舉 | 高 | 系統整體狀態，影響業務流程是否可繼續（如：`Error` 時作業 MUST 暫停） | Flow Spec 描述各 SystemStatus 值對業務流程的影響；不描述 SystemStatus 的計算邏輯 |
| `ISystemStatusProvider` | 介面 | 中 | 流程節點查詢系統整體狀態的能力來源 | Flow Spec 引用「查詢系統狀態」時對應此介面；不描述其實作策略 |

---

## Account 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `IAccountSession` | 介面 | 高 | 登入/登出流程觸發點；`AccountSessionState` 影響作業是否可繼續 | Flow Spec 描述登入/登出的業務流程序列；不描述密碼驗證邏輯 |
| `IUserManagementService` | 介面 | 中 | 帳戶管理流程（建立/調整角色/刪除）的能力來源 | Flow Spec 描述帳戶管理的操作序列與前置角色條件；不描述帳戶資料持久化 |
| `UserAccount` | 資料模型 | 中 | 登入後的使用者身份識別依據 | Flow Spec 引用「目前使用者」時對應此模型；不描述帳戶資料格式 |
| `AccountSessionState` | 枚舉 | 高 | `Unauthenticated` / `Authenticated` 狀態影響業務流程觸發條件 | Flow Spec 描述登入狀態如何限制操作；不重複枚舉值定義 |
| `AccountError` | 錯誤枚舉 | 中 | 登入失敗時的業務回應策略依據 | Flow Spec 描述各錯誤的業務回應（如：`InvalidCredential` 時提示重試）；不重複錯誤語意 |
| `AccountOperationResult` | 資料模型 | 中 | 帳號操作結果，流程分岐依據 | Flow Spec 只描述成功/失敗分岐；不描述欄位結構 |
| `PlatformAction` | 枚舉 | 高 | 定義可被授權政策判定的操作集合，Flow Spec 以此描述各操作前的權限檢查步驟 | Flow Spec 描述「執行 X 前 MUST 確認目前角色允許 X」；不定義角色許可矩陣 |
| `IPermissionChecker` | 介面 | 高 | 流程節點執行授權操作前的檢查能力來源 | Flow Spec 描述「操作前呼叫權限查詢，失敗則不執行」；不描述授權政策實作 |

---

## Camera 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `ICamera` | 介面 | 高 | 相機生命週期管理（初始化→連線→取像→斷線）的業務觸發時機 | Flow Spec 描述「啟動時初始化相機、取像前確認連線狀態」；不描述 SDK 或驅動細節 |
| `DeviceInfo` | 資料模型 | 低 | 相機識別依據，由 `ProjectCameraConfiguration` 提供 | Flow Spec 不描述 DeviceInfo 欄位；只在需要指稱「相機識別」時引用 |
| `CameraState` | 枚舉 | 高 | 相機連線狀態，作為取像流程的前置條件（需 `Connected` 才可取像） | Flow Spec 描述各狀態下允許/禁止的業務操作；不描述狀態的硬體語意 |
| `CameraTriggerMode` | 枚舉 | 高 | 觸發模式決定取像流程路徑（Software = 呼叫端主動觸發；Hardware = 外部訊號；Continuous = 連續流） | Flow Spec 描述各觸發模式下的業務流程差異；不描述幀緩衝或 SDK 行為 |
| `CameraOptions` | 資料模型 | 低 | 相機參數，由 `ProjectCameraConfiguration` 提供，啟動時套用 | Flow Spec 描述「專案載入後套用相機配置」的時機；不描述參數格式 |
| `CameraError` | 錯誤枚舉 | 中 | 相機錯誤的業務回應依據（如：`ConnectionFailed` 時觸發警報） | Flow Spec 描述各錯誤的業務回應策略；不重複錯誤觸發條件 |
| `CameraOperationResult` | 資料模型 | 中 | 相機操作結果，流程分岐依據 | Flow Spec 只描述成功/失敗分岐；不描述欄位結構 |
| `CameraGrabResult` | 資料模型 | 高 | 取像結果，決定影像是否可進入下一流程步驟（前處理/推論） | Flow Spec 描述取像成功後的流程走向與取像失敗的業務回應 |

---

## Communication 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `ICommunicationChannel` | 介面 | 中 | 通訊連線管理，啟動時建立連線、斷線時的業務回應 | Flow Spec 描述通訊連線的業務觸發時機與斷線回應策略；不描述協定細節 |
| `CommunicationState` | 枚舉 | 中 | 連線狀態，影響依賴通訊的業務操作是否可執行 | Flow Spec 描述通訊狀態對業務流程的影響 |
| `CommunicationProtocol` | 枚舉 | 低 | 協定類型，由 `HardwareConfiguration` 決定，Flow Spec 不描述協定選擇 | 配置依據，Flow Spec 不引用 |
| `CommunicationError` | 錯誤枚舉 | 中 | 通訊錯誤時的業務回應依據 | Flow Spec 描述通訊錯誤的業務回應；不重複錯誤語意 |
| `CommunicationOperationResult` | 資料模型 | 中 | 通訊操作結果，流程分岐依據 | Flow Spec 只描述成功/失敗分岐 |
| `CommunicationReceiveResult` | 資料模型 | 中 | 接收結果，上位機通訊流程的資料來源 | Flow Spec 描述接收到資料後的業務處理流程 |
| `CommunicationOptions` | 資料模型 | 低 | 通訊配置，由 `HardwareConfiguration` 提供 | 配置依據，Flow Spec 不描述配置格式 |
| `SerialCommunicationOptions` | 資料模型 | 低 | 序列埠配置，由 `HardwareConfiguration` 提供 | 配置依據，Flow Spec 不引用 |
| `TcpCommunicationOptions` | 資料模型 | 低 | TCP 配置，由 `HardwareConfiguration` 提供 | 配置依據，Flow Spec 不引用 |

---

## Inspection 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `IInspector` | 介面 | 高 | 即時推論能力，核心業務流程（取像→推論→結果）的中樞 | Flow Spec 描述推論觸發條件、輸入準備與結果處理流程；不描述模型推論細節 |
| `IBatchInspector` | 介面 | 高 | 離線批次推論能力，批次作業流程的核心 | Flow Spec 描述批次啟動條件、進度監視與完成/失敗/取消流程 |
| `InspectionInput` | 資料模型 | 高 | 推論輸入，流程中組裝推論輸入的依據（影像 + WorkOrderId） | Flow Spec 描述如何準備 `InspectionInput`；不描述資料模型格式 |
| `InspectionResult` | 資料模型 | 高 | 推論結果，流程中判定分岐的核心依據 | Flow Spec 描述結果取得後的業務處理（IO輸出、存檔、統計）；不描述結果格式 |
| `DefectInfo` | 資料模型 | 低 | 缺陷描述，附加診斷資訊，非業務流程主分岐依據 | Flow Spec 不引用 DefectInfo 進行流程決策；以 `Judgment` 為唯一判定依據 |
| `BoundingBox` | 資料模型 | 低 | 缺陷位置，附加診斷資訊 | Flow Spec 不引用，屬於結果展示層責任 |
| `Judgment` | 枚舉 | 高 | **最重要的流程分岐依據**：OK / NG / Unknown 決定後續業務行為（IO輸出、存檔、統計更新） | Flow Spec 必須明確定義各 Judgment 值的業務回應序列 |
| `InspectionOperationResult` | 資料模型 | 中 | 推論操作結果（LoadModel/BatchStart），流程分岐依據 | Flow Spec 描述成功/失敗分岐 |
| `InspectionInvokeResult` | 資料模型 | 高 | 單次推論結果封裝，流程中取得推論成功/失敗的依據 | Flow Spec 描述推論成功/失敗後的業務走向 |
| `InspectionError` | 錯誤枚舉 | 中 | 推論錯誤的業務回應依據（如：`ModelNotLoaded` 時不可啟動作業） | Flow Spec 描述各錯誤的業務回應 |
| `BatchProgress` | 資料模型 | 高 | 批次進度，批次流程中的進度狀態 | Flow Spec 描述批次進度如何影響流程（何時觸發 `OnBatchCompleted`） |
| `BatchFailureInfo` | 資料模型 | 高 | 批次失敗資訊，批次中止時的流程回應依據 | Flow Spec 描述批次失敗時的業務回應策略 |

---

## Preprocessing 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `IImagePreprocessor` | 介面 | 高 | 前處理管線，在取像→推論中間的可選步驟 | Flow Spec 描述前處理在業務流程中的位置與觸發條件；不描述步驟演算法 |
| `PreprocessingStep` | 資料模型 | 低 | 前處理步驟，由專案配置決定 | 配置依據，Flow Spec 不描述步驟格式 |
| `PreprocessingStepType` | 枚舉 | 低 | 前處理類型（PatternMatch / ROICrop），配置時使用 | 配置依據，Flow Spec 不引用具體步驟類型 |
| `PreprocessingOperationResult` | 資料模型 | 中 | `Configure` 操作結果，啟動時配置流程的分岐依據 | Flow Spec 描述前處理配置失敗時的業務回應 |
| `PreprocessingResult` | 資料模型 | 高 | 前處理結果，決定是否可進入推論步驟 | Flow Spec 描述前處理成功/失敗後的流程走向 |
| `PreprocessingError` | 錯誤枚舉 | 中 | 前處理錯誤的業務回應依據 | Flow Spec 描述各錯誤的業務回應 |

---

## Project 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `IProjectProvider` | 介面 | 高 | 專案載入流程的核心介面，啟動/切換專案的觸發點 | Flow Spec 描述專案載入的觸發條件、成功/失敗後的業務序列 |
| `ProjectMetadata` | 資料模型 | 低 | 專案識別資訊，列舉可用專案時使用 | Flow Spec 引用「列舉專案清單」時對應此模型；不描述欄位格式 |
| `ProjectConfiguration` | 資料模型 | 高 | 聚合配置，載入後驅動硬體初始化與前處理配置的全部依據 | Flow Spec 描述載入後的配置分發序列；不描述配置格式 |
| `HardwareConfiguration` | 資料模型 | 高 | 硬體配置，啟動時初始化相機/通訊/光源的依據 | Flow Spec 描述依硬體配置初始化各裝置的流程序列 |
| `InspectionPipelineConfiguration` | 資料模型 | 高 | **Flow Spec 的核心輸入**：定義本專案有哪些檢測流程（以名稱識別）；相機/前處理/模型的組合由外部定義 | Flow Spec 描述各 Pipeline 的業務觸發條件與執行序列；組合規則的「外部定義」是 Open Question |
| `ProjectCameraConfiguration` | 資料模型 | 高 | 各相機的配置（識別碼 + 參數），啟動時套用至對應 `ICamera` 實例 | Flow Spec 描述配置套用的時機；不描述配置格式 |
| `ProjectLightingControllerConfiguration` | 資料模型 | 高 | 光源控制器配置，啟動時套用 | Flow Spec 描述光源配置套用的時機 |
| `ProjectModelConfiguration` | 資料模型 | 高 | 模型配置，載入專案後觸發 `IInspector.LoadModel` 的依據 | Flow Spec 描述模型載入的時機與失敗回應 |
| `StorageConfiguration` | 資料模型 | 高 | 存檔策略，決定推論後是否保留原始影像（依 Judgment 分別設定） | Flow Spec 描述推論後依 `StorageConfiguration` 決定是否存檔的流程 |
| `ProjectError` | 錯誤枚舉 | 中 | 專案載入錯誤的業務回應依據 | Flow Spec 描述各錯誤的業務回應（如：`ConfigurationInvalid` 時通知使用者） |
| `ProjectOperationResult` | 資料模型 | 中 | 專案載入操作結果，流程分岐依據 | Flow Spec 描述成功/失敗分岐 |

---

## Result 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `IResultRepository` | 介面 | 中 | 歷史結果查詢，結果查詢流程的能力來源 | Flow Spec 描述查詢流程的觸發條件與分頁語意；不描述持久化實作 |
| `WorkOrder` | 資料模型 | 高 | **工單是 Flow Spec 的核心業務概念**：工單識別碼貫穿整個作業流程（推論輸入、結果歸屬、統計查詢） | Flow Spec 必須定義工單的建立時機、生命週期與結束條件 |
| `InspectionStatistics` | 資料模型 | 中 | 累積統計，作業結果的彙總呈現依據 | Flow Spec 描述統計更新的觸發時機（每次推論後）；不描述統計欄位格式 |
| `DefectStatistics` | 資料模型 | 低 | 缺陷種類統計，附加診斷資訊 | Flow Spec 不描述，屬於結果呈現層責任 |
| `QueryCondition` | 資料模型 | 低 | 查詢條件，歷史查詢流程中的輸入格式 | Flow Spec 描述查詢觸發條件；不描述 QueryCondition 格式 |

---

## Alarm 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `IAlarmProvider` | 介面 | 中 | 警報查詢，業務流程中監視警報狀態的能力來源 | Flow Spec 描述何時需要查詢警報清單；不描述查詢實作 |
| `IAlarmReporter` | 介面 | 高 | 警報產生，業務流程中觸發新警報的入口 | Flow Spec 描述哪些業務事件 MUST 觸發 `Raise`（如：相機斷線、推論失敗超過閾值） |
| `IAlarmController` | 介面 | 高 | 警報清除，操作員清除警報的流程 | Flow Spec 描述操作員清除警報的觸發條件與後續流程 |
| `AlarmInfo` | 資料模型 | 中 | 警報資訊，警報觸發時的完整描述 | Flow Spec 描述觸發時需提供哪些資訊（AlarmCode、Level、Message）；不描述格式 |
| `AlarmLevel` | 枚舉 | 高 | **警報層級決定業務回應策略**：Info（繼續作業）/ Warning（繼續但注意）/ Error（暫停作業）/ Critical（停止系統） | Flow Spec 必須明確定義各層級的業務回應策略 |
| `AlarmState` | 枚舉 | 高 | `Active` / `Cleared` 生命週期，影響業務流程是否可繼續 | Flow Spec 描述警報狀態與業務流程的關聯（如：`Error` 層級警報 `Active` 時作業 MUST 暫停） |

---

## IO 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `IIOController` | 介面 | 高 | IO 通道讀寫，推論後輸出 OK/NG 信號的能力來源；Input 通道可作為業務觸發條件來源 | Flow Spec 描述推論結果後的 IO 輸出序列與 Input 通道觸發流程 |
| `IOChannelState` | 資料模型 | 中 | 通道即時狀態，Input 通道狀態可作為業務觸發條件 | Flow Spec 描述 IO 狀態作為觸發條件的時機 |
| `IODirection` | 枚舉 | 中 | Input / Output 通道方向，決定業務流程中的使用方式（Input = 接收觸發；Output = 發送結果） | Flow Spec 描述各方向通道的業務用途；不描述電位語意 |
| `IOError` | 錯誤枚舉 | 中 | IO 操作錯誤的業務回應依據 | Flow Spec 描述 IO 輸出失敗時的業務回應（是否觸發警報）|
| `IOOperationResult` | 資料模型 | 中 | IO 寫入結果，流程分岐依據 | Flow Spec 描述 IO 輸出成功/失敗的業務走向 |

---

## Lighting 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `ILightingController` | 介面 | 高 | 光源控制，取像前開啟光源、取像後（可選）關閉光源的業務序列 | Flow Spec 描述光源控制在取像流程中的位置與觸發時機；不描述通訊協定 |
| `LightingOptions` | 資料模型 | 低 | 光源參數，由 `ProjectLightingControllerConfiguration` 提供 | 配置依據，Flow Spec 描述套用時機；不描述格式 |
| `LightingTriggerMode` | 枚舉 | 高 | `Continuous`（常亮）/ `Strobe`（閃光）模式決定取像流程中光源控制的策略差異 | Flow Spec 描述兩種模式下取像流程的業務差異 |
| `LightingState` | 枚舉 | 高 | 光源連線狀態，影響是否可進行取像作業 | Flow Spec 描述各狀態對業務流程的影響（需 `Connected` 才可取像） |
| `LightingError` | 錯誤枚舉 | 中 | 光源錯誤的業務回應依據 | Flow Spec 描述各錯誤的業務回應（如：`NotConnected` 時觸發警報） |
| `LightingOperationResult` | 資料模型 | 中 | 光源操作結果，流程分岐依據 | Flow Spec 描述成功/失敗分岐 |
| `LightingBrightnessRange` | 資料模型 | 低 | 亮度合法範圍，配置驗證時使用 | 配置依據，Flow Spec 不描述 |

---

## Logging 模組

| 核心概念 | 類型 | 流程相關性 | Flow Spec 角色 | 邊界備註 |
|---------|------|----------|--------------|---------|
| `ILogger` | 介面 | 中 | 系統事件記錄，業務流程關鍵節點寫入日誌的能力來源 | Flow Spec 描述哪些流程節點 MUST / SHOULD 寫入日誌（由 cross-cutting-policy 統一定義） |
| `LogEntry` | 資料模型 | 低 | 日誌條目格式，查詢時使用 | Flow Spec 描述日誌查詢觸發條件；不描述 LogEntry 格式 |
| `LogLevel` | 枚舉 | 中 | 日誌層級，flow spec 的 cross-cutting-policy 需定義各流程節點的寫入層級 | Flow Spec 的 cross-cutting-policy 定義各事件對應的 LogLevel；不重複枚舉定義 |
