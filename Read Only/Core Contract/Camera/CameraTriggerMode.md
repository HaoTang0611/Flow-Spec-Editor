# CameraTriggerMode

**定位：語言無關 × 專案無關**

本文件定義相機觸發模式枚舉，決定幀取像的主動方（Pull / Push），以及 `ICamera` 中 `GrabFrame` 與 `OnFrameArrived` 在各模式下的行為語意。

---

## 枚舉值

| 值 | 說明 | 主要幀傳遞路徑 |
|----|------|--------------|
| `Continuous` | 相機自主連續取像，不需外部觸發 | Push（`OnFrameArrived`） |
| `Software` | 由呼叫端顯式呼叫 `GrabFrame` 觸發單次取像 | Pull（`GrabFrame`） |
| `Hardware` | 由外部硬體訊號觸發取像 | Push（`OnFrameArrived`） |

---

## 各模式行為約束

### Continuous 模式

- `OnFrameArrived` MUST 為主要幀傳遞路徑；每幀到達時 MUST 觸發
- `GrabFrame` MAY 回傳緩衝區中最新可用的幀（語意為「取得當前最新幀」，不觸發新曝光）
- 若相機實作不支援在 `Continuous` 模式下呼叫 `GrabFrame`，MUST 回傳失敗並填入 `OperationNotSupportedInMode` 錯誤碼
- 呼叫端 MUST NOT 以 `GrabFrame` 作為 `Continuous` 模式下的主要幀接收路徑

### Software 模式

- `GrabFrame` MUST 觸發單次曝光並等待結果回傳；逾時時 MUST 回傳 `GrabTimeout`
- `OnFrameArrived` MAY 在 `GrabFrame` 成功後同步觸發，但不保證觸發
- 呼叫端 MUST NOT 以 `OnFrameArrived` 作為 `Software` 模式下的唯一幀接收路徑

### Hardware 模式

- `OnFrameArrived` MUST 為主要幀傳遞路徑；每次硬體訊號觸發後 MUST 觸發
- `GrabFrame` MUST 等待下一次硬體觸發後的幀並回傳；若逾時內無硬體訊號，MUST 回傳 `GrabTimeout`
- 呼叫端 MAY 使用 `GrabFrame` 以同步方式等待下一幀，但 MUST 同時容許以 `OnFrameArrived` 非同步接收

---

## 與常見取像模式的對應

| 常見名稱 | 對應 TriggerMode | 主要 API | 說明 |
|---------|------------------|----------|------|
| 單張取像 / Snap | `Software` | `GrabFrame` | 呼叫端主動要求相機曝光並取得一張影像 |
| Live / Streaming | `Continuous` | `OnFrameArrived` | 相機連續產生影像幀，呼叫端以事件方式接收 |
| 外部觸發取像 | `Hardware` | `OnFrameArrived`；必要時可用 `GrabFrame` 等待下一幀 | 由外部硬體訊號決定取像時機 |
