# ESP 文档

## 文档状态

- 状态：Draft v0
- Owner：硬件工程师
- 角色：设备执行端负责人
- 主线：消费命令、执行动作、回传状态

## 角色定位

ESP 侧是设备控制闭环的执行末端，负责把 MQTT 命令变成真实动作，再把状态和事件回传云层。

## 本轮职责

- 接入 MQTT 云端
- 订阅 `cmd` 和 `ota`
- 解析 RakMessage JSON
- 执行动作
- 回传 `state` `event` `heartbeat`
- 保留原始 `trace_id`

## 本轮不负责

- 前端展示
- Go API 设计
- Runtime 决策逻辑
- 复杂业务编排

## 输入

| 来源 | 内容 |
| --- | --- |
| Go 云层 | MQTT `cmd` `ota` 消息 |

## 输出

| 输出对象 | 内容 |
| --- | --- |
| Go 云层 | MQTT `state` `event` `heartbeat` |

## 接口对接

### Topic

- `rak/{device_id}/cmd`
- `rak/{device_id}/state`
- `rak/{device_id}/event`
- `rak/{device_id}/heartbeat`
- `rak/{device_id}/ota`

### 命令处理要求

- 校验 `version`
- 识别 `action`
- 解析 `params`
- 保留 `trace_id`
- 执行完成后回传状态

### 状态回传最小字段

- `version`
- `type`
- `trace_id`
- `source`
- `target`
- `data.device_id`
- `data.status`

## 本轮最小动作集

- `shake_head`
- `wave_hand`
- `lock_open`
- `lock_close`
- `device_ota`

## 两周任务

### 第 1 周

- 完成 MQTT 连接
- 完成 `cmd` 订阅和 JSON 解析
- 完成最小动作执行框架
- 确认 `state` `event` `heartbeat` 回传格式

### 第 2 周

- 和 Go 联调命令下发与状态回传
- 跑通至少一个固定动作
- 对齐失败、超时、离线场景
- 配合 OTA 触发格式验证

## 依赖对象

| 对接角色 | 依赖内容 |
| --- | --- |
| Go 云架构师 | topic 规范、命令 payload、状态回传字段 |
| 前端 | 不直接对接 |
| AI 工程架构师 | 不直接对接 |

## 联调顺序

1. 先和 Go 冻结 topic
2. 再冻结命令 payload 和状态 payload
3. 再验证 `trace_id` 透传
4. 最后参加端到端冒烟链路

## 验收标准

- 能订阅 `rak/{device_id}/cmd`
- 能执行至少一个动作
- 能向 `rak/{device_id}/state` 回传完成状态
- 心跳和事件消息结构符合正式标准
- 回传消息保留原始 `trace_id`

## 关联主文档

- [技术框架](./技术框架.md)
- [物联网接入标准](./docs/标准/物联网接入标准.md)
- [云层统一接口-v0](./docs/接口/云层统一接口-v0.md)
- [RakMessage-v0](./docs/协议/RakMessage-v0.md)
- [MVP两周对齐表](./docs/协作/MVP两周对齐表.md)
