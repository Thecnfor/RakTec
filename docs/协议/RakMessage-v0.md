# RakMessage v0

## 文档状态

- 状态：Draft v0
- 目标：定义全系统统一消息结构
- 适用范围：HTTP、WebSocket、gRPC 业务载荷、MQTT

## 设计原则

- 协议可以不同，消息语义必须统一
- 所有跨模块消息都必须能映射到 RakMessage
- 字段名固定，避免每个角色重新发明消息结构
- 先满足设备控制闭环，后续能力在兼容前提下扩展

## 标准结构

```json
{
  "version": "v0",
  "type": "action",
  "trace_id": "uuid",
  "source": "frontend:web",
  "target": "runtime:default",
  "action": "shake_head",
  "params": {
    "speed": 100
  },
  "data": {},
  "timestamp": 1710000000
}
```

## 字段定义

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| version | string | 是 | 当前固定为 `v0` |
| type | string | 是 | `action` `state` `event` `error` 四选一 |
| trace_id | string | 是 | 单次链路唯一标识，前端生成或由 Go 补齐 |
| source | string | 是 | 消息来源，格式建议为 `模块:实例` |
| target | string | 是 | 消息目标，格式建议为 `模块:实例` 或 `device:{id}` |
| action | string | 否 | 当 `type=action` 时必填 |
| params | object | 否 | 动作参数，当 `type=action` 时建议存在 |
| data | object | 否 | 状态、事件、错误正文承载区 |
| timestamp | number | 是 | Unix 秒级时间戳 |

## type 约束

### action

- 用途：请求动作执行、动作决策返回、向设备下发命令
- 必填字段：`action`
- 建议字段：`params`
- `data` 可为空对象

### state

- 用途：设备状态回传、执行完成状态、运行中状态
- 必填字段：`data`
- `action` 可为空
- `params` 通常为空对象

### event

- 用途：门铃按下、设备上线、设备离线、告警、心跳等事件
- 必填字段：`data`
- `action` 一般为空

### error

- 用途：执行失败、设备离线、参数错误、超时
- 必填字段：`data.code` `data.message`
- `data.detail` 可选

## source 与 target 规范

| 场景 | source 示例 | target 示例 |
| --- | --- | --- |
| 前端发起执行 | `frontend:web` | `runtime:default` |
| Go 调 Runtime | `cloud:go-gateway` | `runtime:default` |
| Runtime 回 Go | `runtime:default` | `cloud:go-gateway` |
| Go 发设备命令 | `cloud:go-gateway` | `device:esp32-001` |
| 设备回传状态 | `device:esp32-001` | `cloud:go-gateway` |
| Go 推送前端 | `cloud:go-gateway` | `frontend:web` |

## trace 规则

- 同一次用户动作全链路必须复用同一个 `trace_id`
- 前端已有 `trace_id` 时，Go、Runtime、设备回传都必须透传
- 前端未生成 `trace_id` 时，由 Go 在入口创建
- WebSocket 推送与 MQTT 状态回传也必须带上原始 `trace_id`

## 时间戳规则

- `timestamp` 使用 Unix 秒级时间戳
- 每次消息创建时写入发送端本地时间
- 不要求所有节点严格时钟同步，但必须保留原始时间用于链路排查

## 最小动作集建议

| action | 说明 | params |
| --- | --- | --- |
| shake_head | 摇头动作 | `speed` |
| wave_hand | 挥手动作 | `speed` `duration_ms` |
| lock_open | 开锁 | `timeout_ms` |
| lock_close | 关锁 | `timeout_ms` |
| device_reboot | 设备重启 | 无或空对象 |

## 样例

### 1. 前端请求执行动作

```json
{
  "version": "v0",
  "type": "action",
  "trace_id": "9f0e7b14-bf58-4b6a-9a2a-8f4e7f8e6001",
  "source": "frontend:web",
  "target": "runtime:default",
  "action": "shake_head",
  "params": {
    "speed": 100
  },
  "data": {},
  "timestamp": 1710000000
}
```

### 2. Runtime 返回动作结果

```json
{
  "version": "v0",
  "type": "action",
  "trace_id": "9f0e7b14-bf58-4b6a-9a2a-8f4e7f8e6001",
  "source": "runtime:default",
  "target": "device:esp32-001",
  "action": "shake_head",
  "params": {
    "speed": 100
  },
  "data": {},
  "timestamp": 1710000001
}
```

### 3. 设备回传状态

```json
{
  "version": "v0",
  "type": "state",
  "trace_id": "9f0e7b14-bf58-4b6a-9a2a-8f4e7f8e6001",
  "source": "device:esp32-001",
  "target": "cloud:go-gateway",
  "action": "",
  "params": {},
  "data": {
    "status": "done",
    "device_id": "esp32-001"
  },
  "timestamp": 1710000003
}
```

### 4. 错误回传

```json
{
  "version": "v0",
  "type": "error",
  "trace_id": "9f0e7b14-bf58-4b6a-9a2a-8f4e7f8e6001",
  "source": "cloud:go-gateway",
  "target": "frontend:web",
  "action": "",
  "params": {},
  "data": {
    "code": "DEVICE_OFFLINE",
    "message": "目标设备当前离线",
    "detail": {
      "device_id": "esp32-001"
    }
  },
  "timestamp": 1710000004
}
```

## 兼容性规则

- 新增字段只能追加，不能修改既有字段语义
- `version` 升级前，`type` 和基础字段不可重命名
- 如果某通道无法原样承载 JSON 结构，必须先做等价映射，再恢复为 RakMessage 语义
- 文档未列出的字段默认不进入 v0 正式协议
