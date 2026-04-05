# Go 文档

## 文档状态

- 状态：Draft v0
- Owner：Go 云架构师
- 角色：Go 云层负责人
- 主线：设备控制闭环中枢

## 角色定位

Go 是本轮 MVP 的总转发与总汇聚节点，负责把前端请求、Runtime 决策和硬件状态收束成一条稳定控制流。

## 本轮职责

- 提供 HTTP API 入口
- 提供 WebSocket 状态推送
- 调用 Runtime gRPC 接口
- 通过 MQTT 向设备下发动作
- 汇总设备状态、事件、心跳
- 处理错误码、超时、幂等和链路 trace

## 本轮不负责

- 做前端页面交互
- 做 Runtime 内部决策逻辑
- 实现硬件设备驱动
- 设计长期记忆和智能系统

## 输入

| 来源 | 内容 |
| --- | --- |
| 前端 | HTTP 动作请求、设备状态查询 |
| Runtime | gRPC 动作结果、错误码 |
| 硬件 | MQTT 状态、事件、心跳、OTA 回执 |

## 输出

| 输出对象 | 内容 |
| --- | --- |
| 前端 | HTTP 响应、WebSocket 状态与错误推送 |
| Runtime | gRPC `Execute` 请求 |
| 硬件 | MQTT `cmd` `ota` 消息 |

## 接口对接

### 对前端

- `POST /api/v1/action/execute`
- `GET /api/v1/device/{device_id}`
- `POST /api/v1/device/ota`
- `GET /ws`

### 对 Runtime

- gRPC `RuntimeService.Execute`

请求关键字段：

- `version`
- `trace_id`
- `source`
- `target`
- `state`
- `available_actions`
- `action`
- `params_json`

### 对硬件

- `rak/{device_id}/cmd`
- `rak/{device_id}/state`
- `rak/{device_id}/event`
- `rak/{device_id}/heartbeat`
- `rak/{device_id}/ota`

## 主处理流程

1. 接收前端动作请求
2. 校验参数并补齐 `trace_id`
3. 调 Runtime 获取动作和参数
4. Runtime 返回成功后发布 MQTT 命令
5. 接收设备状态并写回链路状态
6. 通过 WebSocket 推送给前端

## 错误与状态责任

| 场景 | Go 责任 |
| --- | --- |
| 参数错误 | 直接返回前端 |
| Runtime 不可用 | 不下发设备命令，推送错误 |
| Runtime 决策失败 | 不下发设备命令，推送错误 |
| MQTT 发布失败 | 返回失败并推送错误事件 |
| 设备超时 | 标记失败态并推送错误 |
| 重复请求 | 基于 `trace_id` 做幂等处理 |

## 两周任务

### 第 1 周

- 冻结 HTTP API
- 冻结 WebSocket 推送结构
- 冻结 Runtime gRPC 请求响应
- 冻结 MQTT topic 和 payload 转发口径

### 第 2 周

- 串通端到端动作执行链
- 处理设备离线、超时、失败分支
- 和前端、Runtime、硬件做统一 `trace_id` 联调

## 依赖对象

| 对接角色 | 依赖内容 |
| --- | --- |
| 前端 | 请求体、响应体、推送体消费规则 |
| AI 工程架构师 | gRPC 字段、动作结果、错误码 |
| 硬件工程师 | MQTT topic、命令 payload、状态回传结构 |

## 联调顺序

1. 先和前端冻结 HTTP 与 WebSocket
2. 再和 Runtime 冻结 gRPC
3. 再和硬件冻结 MQTT
4. 最后做一条完整动作链的端到端验证

## 验收标准

- 能接收前端动作请求并返回受理结果
- 能调用 Runtime 并获得有效动作
- 能向设备发布 MQTT 命令
- 能接收设备状态并推送给前端
- 能正确处理 `INVALID_PARAMS` `DEVICE_OFFLINE` `RUNTIME_UNAVAILABLE` `DEVICE_TIMEOUT`

## 关联主文档

- [技术框架](./技术框架.md)
- [RakMessage-v0](./docs/协议/RakMessage-v0.md)
- [云层统一接口-v0](./docs/接口/云层统一接口-v0.md)
- [物联网接入标准](./docs/标准/物联网接入标准.md)
- [MVP两周对齐表](./docs/协作/MVP两周对齐表.md)
