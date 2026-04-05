# Runtime 文档

## 文档状态

- 状态：Draft v0
- Owner：AI 工程架构师
- 角色：Fake Runtime 负责人
- 主线：为设备控制闭环提供最小动作决策服务

## 角色定位

Runtime 在本轮只做最小决策服务，不承担完整智能中枢角色。

## 本轮职责

- 接收 Go 发起的 gRPC `Execute` 请求
- 基于输入状态或目标动作返回可执行动作
- 保持 `trace_id` 原样透传
- 输出可还原为 RakMessage 的 `action + params`
- 对失败场景返回明确错误码

## 本轮不负责

- 长期记忆
- 多步复杂规划
- 向量检索与完整 NTR
- 技能市场注册和分发
- MQTT 管理
- 前端推送

## 输入

| 字段 | 说明 |
| --- | --- |
| version | 固定为 `v0` |
| trace_id | 链路唯一标识 |
| source | 调用来源 |
| target | Runtime 实例 |
| state | 当前状态摘要 |
| available_actions | 允许动作集合 |
| action | 请求动作名，可为空 |
| params_json | 参数 JSON 字符串 |

## 输出

| 字段 | 说明 |
| --- | --- |
| version | 固定为 `v0` |
| trace_id | 原样透传 |
| action | 决策动作 |
| params_json | 决策后的参数 |
| status | `ok` 或 `error` |
| error_code | 失败码 |
| error_message | 失败说明 |

## 接口对接

### gRPC

- Service：`RuntimeService`
- Method：`Execute(ActionRequest) returns (ActionResponse)`

### 决策模式

#### 模式 1：动作确认

- 前端动作已明确
- Runtime 负责校验是否合法、是否需要补充参数

#### 模式 2：状态转动作

- Go 只提供状态摘要
- Runtime 从 `available_actions` 中选动作

## 本轮必须交付

- `Execute` 请求响应样例
- 固定动作选择逻辑
- 错误码表
- 联调时可复现的稳定返回结果

## 错误码

| error_code | 含义 |
| --- | --- |
| INVALID_STATE | 输入状态无效 |
| ACTION_NOT_ALLOWED | 动作不在允许集合中 |
| PARAMS_INVALID | 参数不合法 |
| DECISION_FAILED | 决策失败 |
| TIMEOUT | Runtime 处理超时 |

## 两周任务

### 第 1 周

- 冻结 gRPC 输入输出字段
- 明确动作确认和状态转动作两种模式
- 给 Go 提供稳定样例和错误码

### 第 2 周

- 与 Go 联调成功
- 跑通至少一个固定动作样例
- 对齐异常返回与超时处理

## 依赖对象

| 对接角色 | 依赖内容 |
| --- | --- |
| Go 云架构师 | gRPC 字段、状态摘要、可执行动作集合 |
| 前端 | 不直接对接 |
| 硬件工程师 | 不直接对接 |

## 联调顺序

1. 先和 Go 冻结 `ActionRequest`
2. 再冻结 `ActionResponse`
3. 再对齐错误码和超时处理
4. 最后参加端到端动作链路联调

## 验收标准

- 收到 Go 请求后能稳定返回 `ok` 或 `error`
- 返回动作必须在 `available_actions` 范围内
- 返回参数能被 Go 直接转成下游 RakMessage
- 所有联调返回都保留原始 `trace_id`

## 关联主文档

- [技术框架](./技术框架.md)
- [Runtime-MVP边界](./docs/运行时/Runtime-MVP边界.md)
- [云层统一接口-v0](./docs/接口/云层统一接口-v0.md)
- [RakMessage-v0](./docs/协议/RakMessage-v0.md)
- [MVP两周对齐表](./docs/协作/MVP两周对齐表.md)
