# Front 文档

## 文档状态

- 状态：Draft v0
- Owner：你
- 角色：前端负责人
- 主线：设备控制闭环

## 角色定位

前端只负责把用户动作转成标准请求，把云层回传转成可见状态，不负责 Runtime 决策和硬件协议实现。

## 本轮职责

- 提供最小操作界面
- 发起动作执行请求
- 查询设备基础状态
- 接收 WebSocket 实时状态推送
- 展示执行结果、错误提示、设备在线状态

## 本轮不负责

- 设计 Runtime 决策逻辑
- 定义 MQTT topic 和设备协议
- 实现 gRPC 接口
- 管理设备驱动和 OTA 执行

## 输入

| 来源 | 内容 |
| --- | --- |
| 用户 | 动作触发、设备选择 |
| Go 云层 | HTTP 响应、WebSocket 状态推送、错误推送 |

## 输出

| 输出对象 | 内容 |
| --- | --- |
| Go 云层 | `POST /api/v1/action/execute` 请求体 |
| Go 云层 | `GET /api/v1/device/{device_id}` 查询请求 |
| 页面 UI | 设备状态、动作状态、错误提示 |

## 接口对接

### HTTP

- `POST /api/v1/action/execute`
- `GET /api/v1/device/{device_id}`

动作执行请求样例：

```json
{
  "version": "v0",
  "type": "action",
  "trace_id": "9f0e7b14-bf58-4b6a-9a2a-8f4e7f8e6001",
  "source": "frontend:web",
  "target": "runtime:default",
  "action": "shake_head",
  "params": {
    "device_id": "esp32-001",
    "speed": 100
  },
  "data": {},
  "timestamp": 1710000000
}
```

### WebSocket

- 通道：`GET /ws`
- 关注类型：`state` `event` `error`

状态推送最小消费字段：

- `trace_id`
- `type`
- `data.device_id`
- `data.status`

## 本轮页面任务

### 第 1 周

- 做最小动作入口页面
- 支持选择设备和动作参数
- 接 Go 的动作执行请求体
- 接 Go 的 WebSocket 状态推送体

### 第 2 周

- 联调执行状态与错误展示
- 跑通一次按钮触发到状态完成的闭环
- 补充离线、超时、失败展示

## 对接对象

| 对接角色 | 需要对齐的内容 |
| --- | --- |
| Go 云架构师 | HTTP 请求体、HTTP 响应体、WebSocket 推送体 |
| 硬件工程师 | 仅关注最终状态展示，不直接对接设备协议 |
| AI 工程架构师 | 不直接对接，经过 Go 云层 |

## 联调顺序

1. 先和 Go 对齐 `POST /api/v1/action/execute`
2. 再对齐 `GET /api/v1/device/{device_id}`
3. 再对齐 WebSocket `state` `error` 推送
4. 最后参与端到端冒烟联调

## 验收标准

- 能发起一次标准动作请求
- 能展示设备在线状态
- 能根据 `trace_id` 把执行中、完成、失败状态串起来
- 设备离线或超时时能看到明确错误提示

## 关联主文档

- [技术框架](./技术框架.md)
- [云层统一接口-v0](./docs/接口/云层统一接口-v0.md)
- [RakMessage-v0](./docs/协议/RakMessage-v0.md)
- [MVP两周对齐表](./docs/协作/MVP两周对齐表.md)
