# Deribit Starbase SBE 协议中文说明手册

## 1. 什么是 Starbase？
Starbase 是 Deribit 专为机构交易员和做市商（Market Maker）打造的高性能撮合引擎。它采用 **SBE (Simple Binary Encoding)** 替代传统的文本/JSON 协议，大幅降低了序列化与反序列化延迟。

## 2. 核心参数与头部结构
所有通过 TCP 发送的 SBE 消息均包含固定的 32 字节 Header：
- **Protocol ID**: `0xDB`（固定协议标识）
- **Message Type ID**: 消息类型（如 100 代表 `NewOrderRequest`，200 代表 `NewOrderResponse`）
- **Schema Version**: 建议协商设为最新版 Schema

## 3. LD4 机房托管与网络连接
- **托管位置**: Equinix LD4 (London)
- **支持架构**: AWS PrivateLink / Cross-Connect 物理直连
- **时间同步**: 支持 PTP (Precision Time Protocol)，时间戳精确至纳秒（Nanoseconds）
