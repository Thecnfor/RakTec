# Runtime MVP 边界

## 文档状态

- 状态：Draft v0
- 目标：约束 AI Runtime 在两周 MVP 内的职责范围
- 适用对象：AI 工程架构师、Go 云架构师、总架构师

## 一句话定义

MVP Runtime 是一个最小决策服务，不是完整智能中枢。

它只负责根据当前状态和可选动作，返回一个可执行的动作结果，并保持与未来真实 Runtime 的接口兼容。

## 本轮必须负责

- 接收 Go 发起的 gRPC `Execute` 请求
- 根据输入状态返回动作名和参数
- 保持 `trace_id` 透传
- 将输出映射回 RakMessage `action + params` 语义
- 对无法决策的情况返回明确错误

## 本轮明确不负责

- 长期记忆系统
- 多轮复杂规划
- 向量检索与完整 NTR
- LoRA 技能切换
- 技能市场注册与分发
- 设备连接、MQTT 管理、前端推送

## 输入输出

### 输入

| 字段 | 说明 |
| --- | --- |
| version | 当前固定为 `v0` |
| trace_id | 链路唯一标识 |
| source | 调用来源，固定由 Go 传入 |
| target | 目标 Runtime 实例 |
| state | 当前状态摘要 |
| available_actions | 当前设备允许执行的动作集合 |
| action | 前端请求的动作名，可为空 |
| params_json | 动作参数 JSON 字符串 |

### 输出

| 字段 | 说明 |
| --- | --- |
| version | 当前固定为 `v0` |
| trace_id | 原样透传 |
| action | 决策后的动作 |
| params_json | 决策后的动作参数 |
| status | `ok` 或 `error` |
| error_code | 失败时必填 |
| error_message | 失败说明 |

## 行为模式

### 模式 1：动作确认

- 场景：前端已明确指定动作
- Runtime 负责校验、补充参数或拒绝不合法动作
- 示例：前端请求 `shake_head`，Runtime 输出 `shake_head + {"speed":100}`

### 模式 2：状态转动作

- 场景：Go 只提供当前状态，由 Runtime 自主选一个最优动作
- Runtime 必须从 `available_actions` 中选择，不得返回未声明动作
- 示例：`state=user_click_button` → `action=wave_hand`

## 错误处理

| error_code | 含义 |
| --- | --- |
| INVALID_STATE | 输入状态无效 |
| ACTION_NOT_ALLOWED | 返回动作不在允许集合中 |
| PARAMS_INVALID | 参数无法解析或不合法 |
| DECISION_FAILED | 内部决策失败 |
| TIMEOUT | Runtime 处理超时 |

## 与未来真实 Runtime 的兼容边界

- 当前 gRPC 接口保留 `state` 和 `available_actions`，未来可接 NTR 或记忆模块
- 当前 `params_json` 保持 JSON 字符串，后续可扩展为更复杂结构，但不改变字段含义
- 当前只要求同步返回一个动作，未来若支持多步计划，也必须先兼容单步动作模式

## 实现建议

- 本轮优先采用规则式或模板式 Fake Runtime
- 决策逻辑必须可预测、可复现、可联调
- 返回内容必须稳定，不使用随机性输出
- 所有失败都要以明确错误码返回，而不是静默失败

## 联调要求

- Go 调 Runtime 时必须传入 `trace_id`
- Runtime 回包后，Go 才允许下发 MQTT
- Runtime 返回 `status=error` 时，Go 直接向前端返回失败并推送错误事件
- AI 工程架构师提供的接口样例必须与 `RakMessage-v0` 和 `云层统一接口-v0` 保持一致
