# MVP 两周对齐表

## 文档状态

- 状态：Draft v0
- 目标：把角色边界、交付物、联调顺序和验收口径拉齐

## 项目目标

两周内打通一条设备控制闭环：

前端发起动作 → Go 调 Runtime → MQTT 下发硬件 → 设备回传状态 → 前端看到最终结果

## 角色与 Owner

| 角色 | Owner | 本轮职责 |
| --- | --- | --- |
| 总架构师 | 你 | 前端、Runtime 总边界、硬件联调总控 |
| Go 云架构师 | Go 云架构师 | API 入口、gRPC 调 Runtime、MQTT 转发、WebSocket 推送 |
| AI 工程架构师 | AI 工程架构师 | Fake Runtime 定义与实现边界 |
| 硬件工程师 | 硬件工程师 | 设备执行引擎、MQTT 接入、状态回传 |

## 角色执行文档

| 角色 | 文档 |
| --- | --- |
| 前端负责人 | [front.md](../../front.md) |
| Go 云架构师 | [go.md](../../go.md) |
| AI 工程架构师 | [runtime.md](../../runtime.md) |
| 硬件工程师 | [esp.md](../../esp.md) |

## 模块交付物

| 模块 | 必交付物 | 交付给谁 |
| --- | --- | --- |
| 前端 | 动作请求体、状态展示结构、WebSocket 消费逻辑 | Go 云层 |
| Go 云层 | HTTP API、gRPC 调用口、MQTT topic/payload、状态推送体 | 前端、Runtime、硬件 |
| Runtime | gRPC `Execute` 输入输出样例、错误码 | Go 云层 |
| 硬件 | `cmd` 消费、`state` 回传样例、心跳样例 | Go 云层、前端 |

说明：

- 横向协作以本表为准
- 纵向执行以各自角色文档为准

## 依赖关系

| 角色 | 依赖前置 |
| --- | --- |
| 前端 | Go 提供 HTTP 和 WebSocket 结构 |
| Go 云层 | Runtime gRPC 结构、硬件 MQTT topic |
| Runtime | Go 定义请求字段和返回约定 |
| 硬件 | Go 定义 MQTT 命令和状态 payload |

## 接口冻结点

### 冻结点 1

- RakMessage v0 字段名
- `trace_id` 透传规则
- MQTT topic 命名

### 冻结点 2

- HTTP `POST /api/v1/action/execute`
- WebSocket 状态推送体
- gRPC `RuntimeService.Execute`

### 冻结点 3

- 设备执行状态字段
- 错误码和超时处理
- OTA 最小触发格式

## 两周节奏

### 第 1 周

| 项目 | 目标 |
| --- | --- |
| 文档 | 冻结主文档、协议、接口、Runtime 边界、物联网标准 |
| 前端 | 确认动作请求体与状态展示体 |
| Go | 确认 HTTP、WebSocket、gRPC、MQTT 四类接口 |
| Runtime | 确认 Fake Runtime 的输入输出 |
| 硬件 | 确认设备命令消费与状态回传格式 |

### 第 2 周

| 项目 | 目标 |
| --- | --- |
| 联调 | 跑通一次完整动作链路 |
| 稳定性 | 对齐错误码、离线、超时状态 |
| 验收 | 用统一 `trace_id` 完成一次端到端验证 |

## 联调顺序

1. 前端与 Go 先对齐 HTTP 请求和 WebSocket 推送
2. Go 与 Runtime 冻结 gRPC 请求响应
3. Go 与硬件冻结 MQTT topic 与 payload
4. 四方使用同一设备和同一动作做端到端联调

## 冒烟链路

推荐固定一条最小冒烟链路：

- 设备：`esp32-001`
- 动作：`shake_head`
- 输入端：前端按钮点击
- 输出端：前端显示 `done`

## 验收口径

- 前端、Go、Runtime、硬件四方都使用同一套 RakMessage 语义
- 四方都能输出和识别同一个 `trace_id`
- 至少完成一次动作下发和一次状态回传
- 设备离线时，前端能收到明确错误而不是无响应
- 文档可以直接作为联调时的唯一对照基线
