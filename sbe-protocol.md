# Deribit Starbase SBE 协议说明手册

Simple Binary Encoding (SBE) 是专为低延迟、高吞吐量的极速交易场景设计的二进制报文协议。相比传统的 JSON / FIX 协议，SBE 无需复杂的字符串解析，支持直接零拷贝（Zero-Copy）反序列化。

---

## 1. 报文标准结构

所有传输的二进制数据包均由 **Message Header（固定报头）** 和 **Message Body（消息体）** 构成：

| 字段名 (Field) | 数据类型 (Type) | 字节数 (Bytes) | 描述 (Description) |
| :--- | :--- | :--- | :--- |
| `blockLength` | uint16 | 2 | 消息主体的字节长度 |
| `templateId` | uint16 | 2 | 消息模板 ID（决定业务类型） |
| `schemaId` | uint16 | 2 | 协议 Schema 编号 |
| `version` | uint16 | 2 | 协议版本号 |

---

## 2. 核心模板 ID (Template IDs)

| Template ID | 消息名称 | 传输方向 | 说明 |
| :--- | :--- | :--- | :--- |
| `101` | `NewOrderSingle` | Client -> Server | 极速限价/市价报单 |
| `102` | `OrderCancelRequest` | Client -> Server | 撤单请求 |
| `201` | `OrderReport` | Server -> Client | 订单状态更新（已成交/已撤销） |
| `301` | `DepthUpdate` | Server -> Client | 深度行情推送 (L2 / L3) |

---

## 3. C++ 解析示例 (Zero-Copy)

```cpp
#include <iostream>
#include <cstdint>

#pragma pack(push, 1)
struct SbeHeader {
    uint16_t blockLength;
    uint16_t templateId;
    uint16_t schemaId;
    uint16_t version;
};
#pragma pack(pop)

void parse_packet(const char* buffer) {
    const auto* header = reinterpret_cast<const SbeHeader*>(buffer);
    std::cout << "Template ID: " << header->templateId << std::endl;
    std::cout << "Block Length: " << header->blockLength << " bytes" << std::endl;
}

---

---

# 4. 最佳实践指南

1. **字节对齐**：使用 `#pragma pack(push, 1)` 或语言原生的内存对齐控制，避免结构体填充（Padding）导致偏移错位。

2. **端序转换**：SBE 采用 Little-Endian（小端序）存储，x86 / ARM 架构主机可直接指针转换读取。

3. **TCP 粘包处理**：先读取前 8 字节 Header 确认 blockLength，再精确读取对应长度的 Body。
