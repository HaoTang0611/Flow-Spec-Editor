# CameraState

**定位：語言無關 × 專案無關**

本文件定義相機連線狀態枚舉。

---

## 枚舉值

| 值 | 說明 |
|----|------|
| `Disconnected` | 未建立連線 |
| `Connecting` | 連線建立中 |
| `Connected` | 連線正常，可擷取影像 |
| `Error` | 連線中斷或影像擷取發生錯誤 |

## 約束

- `CameraState` 為相機驅動程式可觀察的連線狀態，MUST NOT 包含業務層的「運作中」語意
