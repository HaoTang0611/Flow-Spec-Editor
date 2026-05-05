# Communication — 索引

**定位：語言無關 × 專案無關**

本模組定義協定無關的通訊通道能力契約，以及各協定的配置選項。

---

## 文件清單

| 文件 | 說明 |
|------|------|
| [ICommunicationChannel.md](ICommunicationChannel.md) | 通訊通道抽象介面（協定無關） |
| [CommunicationOptions.md](CommunicationOptions.md) | 通訊配置基礎選項（共用欄位） |
| [SerialCommunicationOptions.md](SerialCommunicationOptions.md) | 序列埠通訊配置選項（含 SerialStopBits、SerialParity 枚舉） |
| [TcpCommunicationOptions.md](TcpCommunicationOptions.md) | TCP 通訊配置選項 |
| [CommunicationState.md](CommunicationState.md) | 通訊通道連線狀態枚舉 |
| [CommunicationProtocol.md](CommunicationProtocol.md) | 通訊協定類型枚舉（Serial / Tcp） |
| [CommunicationError.md](CommunicationError.md) | 通訊操作錯誤識別碼枚舉與觸發條件 |
| [CommunicationOperationResult.md](CommunicationOperationResult.md) | 通訊操作回傳結果資料模型 |
| [CommunicationReceiveResult.md](CommunicationReceiveResult.md) | 通訊接收操作回傳結果資料模型 |
