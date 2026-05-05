# CommunicationOptions

**定位：語言無關 × 專案無關**

本文件定義通訊完整配置，包含所有協定共用欄位與協定特定配置。

依賴：引用 Communication/CommunicationProtocol、Communication/SerialCommunicationOptions、Communication/TcpCommunicationOptions

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `ProtocolType` | `CommunicationProtocol` | 通訊協定類型，決定 `ProtocolOptions` 的實際型別 |
| `ConnectTimeoutMs` | 整數 | 連線建立超時時間（毫秒） |
| `ReceiveTimeoutMs` | 整數 | 單次接收預設超時時間（毫秒）；當 `Receive` 超時時間參數為 `-1` 時使用此值 |
| `ProtocolOptions` | `SerialCommunicationOptions` 或 `TcpCommunicationOptions` | 協定特定配置；型別 MUST 與 `ProtocolType` 對應 |

## 約束

- `ConnectTimeoutMs` MUST 大於 0
- `ReceiveTimeoutMs` MUST 大於等於 0；為 0 時表示非阻塞接收
- `ProtocolType` 為 `Serial` 時，`ProtocolOptions` MUST 為 `SerialCommunicationOptions`
- `ProtocolType` 為 `Tcp` 時，`ProtocolOptions` MUST 為 `TcpCommunicationOptions`
