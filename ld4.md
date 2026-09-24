# Equinix LD4 (London) 极速机房连接指南

Equinix LD4（位于英国斯劳 Slough）是全球加密货币衍生品与高频交易（HFT）的核心数据中心。StarbaseExchange 的交易撮合引擎及 SBE 二进制行情节点均部署于 LD4 内。

通过在本机房内配置物理交叉连接（Cross-Connect），量化团队可将单向网络延迟降至 **1.2ms** 以内。

---

## 1. 物理层与网络接入 (Layer 1 & Layer 2)

### 物理交叉连接 (Cross-Connect) 规范
- **数据中心地址**：Equinix LD4, Slough, Berkshire, UK
- **介质规格**：单模光纤 (Single-Mode Fiber, OS2, LC/UPC)
- **端口带宽**：10 GbE SFP+ / 40 GbE QSFP+
- **LOA 申请**：授权信（Letter of Authorization）请联系 Starbase 运维团队获取。

---

## 2. 网络拓扑与地址配置

StarbaseExchange 节点在 LD4 提供生产环境（Production）与测试环境（Sandbox）两套独立的网络接入点：

| 环境类型 | 接入方式 | 目标网段 / 寻址 | 推荐协议 |
| :--- | :--- | :--- | :--- |
| **Production (LD4)** | Direct Cross-Connect / VLAN | `10.140.20.0/24` | SBE (UDP Multicast / TCP) |
| **Sandbox (LD4)** | Internet / VPN Gateway | `10.240.20.0/24` | SBE / WebSocket |

---

## 3. 低延迟系统优化指南 (Linux Kernel Tuning)

针对运行在 LD4 机房的量化交易服务器，建议在 Linux 内核层配置以下优化参数，以减少内核网络栈延迟：

### 3.1 网卡 CPU 亲和性与 Polling 设置
确保网卡中断绑核，并启用 `busy_poll` 减少系统上下文切换：

```bash
# 启用内核繁忙轮询 (Busy Polling)
sysctl -w net.core.busy_poll=50
sysctl -w net.core.busy_read=50

# 增大 Socket 接收/发送缓冲区
sysctl -w net.core.rmem_max=67108864
sysctl -w net.core.wmem_max=67108864

---

### 3.2 禁用 CPU 节电模式 (CPU Governor)
确保服务器 CPU 运行在最高性能状态：
cpupower frequency-set --governor performance

---

```

##4. 网络延迟连通性测试
完成 Cross-Connect 物理连接后，可使用 ping 和 onload（若使用 Solarflare 网卡）验证网络延迟：

### 测量以太网子网网关延迟
ping -c 100 -i 0.2 10.140.20.1

### 使用 Solarflare OpenOnload 绕过内核栈直接访问 SBE 端口
onload --profile=latency ./sbe_client --ip 10.140.20.100 --port 9001

```
注意：如果遇到丢包或 RTT 波动，请联系 Telegram 运维技术支持通道：Starbase Exchange Support。
