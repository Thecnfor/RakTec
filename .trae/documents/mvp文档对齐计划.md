# Rak 两周 MVP 文档对齐计划

## Summary

- 目标：在不写代码的前提下，把当前零散草稿收敛成一套可直接指导 2 周 MVP 开发的正式工程文档。
- 核心闭环：以“设备控制闭环”为唯一一号场景，固定链路为前端 → Go 云层 → Fake Runtime → MQTT 硬件 → 状态回传前端。
- 文档组织：采用“主文档 + 分文档”，其中 `d:\doc\Project\Rak生态\Rak\Tec\技术框架.md` 作为总览与索引，协议、接口、环境、物联标准、协作机制拆为独立 Markdown。
- 固定决策：MVP 协议栈直接定版为 HTTP/WebSocket、gRPC、MQTT、HTTP OTA；全链路统一使用 RakMessage 载荷模型。
- 交付对象：内部工程团队，重点服务总架构、Go 云层、AI Runtime、硬件四个角色的并行开发与联调。

## Current State Analysis

- `d:\doc\Project\Rak生态\Rak\Tec\技术框架.md` 当前为空，是最适合沉淀正式总览的位置，但尚未承担“统一入口”职责。
- `d:\doc\Project\Rak生态\Rak\Tec\dev\rak.md` 已有完整未来态架构图，覆盖前端、云层、智能层、边缘层、硬件层、记忆层与技能生命周期，适合作为高层蓝图来源，但内容偏大而全，不适合直接作为两周 MVP 规范。
- `d:\doc\Project\Rak生态\Rak\Tec\dev\quic.md` 已给出最接近 MVP 的接口收敛方案，包括 RakMessage、HTTP/gRPC/MQTT 分工、topic 规范、Fake Runtime 和最小闭环，是本轮接口文档的主要技术底稿。
- `d:\doc\Project\Rak生态\Rak\Tec\dev\tmp.md` 明确了 Rak 的产品叙事与长期愿景，适合作为“为什么这样分层”的背景说明，但需要从长期愿景中切出 MVP 范围。
- `d:\doc\Project\Rak生态\Rak\Tec\example\技术环境标准.ipynb` 是环境安装实验记录，包含 conda/pip 命令和报错输出，当前不具备正式标准文档质量。
- `d:\doc\Project\Rak生态\Rak\Tec\example\物联网标准.ipynb` 只有 MQTTS/WSS 连接示例，缺少 topic、消息格式、状态流和安全边界说明，而且示例关闭了证书校验，不适合直接升格为正式标准。
- `d:\doc\Project\Rak生态\Rak\Tec\example\RakMessage协议.ipynb` 与 `d:\doc\Project\Rak生态\Rak\Tec\example\云层统一接口.ipynb` 当前为空，说明最关键的协议与接口文档尚未真正落地。
- `d:\doc\Project\Rak生态\Rak\Tec\.gitignore` 忽略了 `dev/`，意味着 `dev` 更适合作为内部草稿来源，不应继续承担团队正式对齐文档角色。

## Assumptions & Decisions

- Audience：本轮文档默认面向内部工程团队，不面向投资人或外部合作方。
- In Scope：设备控制闭环、协议统一、接口对齐、消息格式统一、控制流成立、团队 owner 与联调顺序。
- Out of Scope：完整技能市场落地、完整 NTR/记忆系统实现、完整数据库设计、生产级安全合规细则、所有未来态能力一次写完。
- MVP Runtime：定义为 Fake Runtime，只承担最小调度与动作决策接口，不承诺完整 AI 自主能力。
- Protocol Freeze：MVP 直接固定协议栈，不保留“实现时再决定”的模糊空间。
- Documentation Strategy：正式规范使用 Markdown；现有 `dev/` 和 `example/*.ipynb` 作为素材来源，不再作为最终规范形态。
- Collaboration Strategy：文档中必须写清模块 owner、输入输出、依赖关系和联调顺序，避免角色边界重叠。

## Proposed Changes

### 1. 重写主文档 `d:\doc\Project\Rak生态\Rak\Tec\技术框架.md`

- What：把空白主文档改造成正式总入口。
- Why：当前没有统一入口，团队无法从一份文档快速理解 MVP 范围、链路、文档索引和负责人边界。
- How：
  - 用 2 周 MVP 目标取代泛化愿景表述。
  - 明确唯一主线是“设备控制闭环”。
  - 固定系统分层：前端、Go 云层、Fake Runtime、硬件/MQTT。
  - 放入统一控制流图、模块责任边界、正式子文档索引。
  - 放入团队分工摘要与联调顺序摘要。

### 2. 新建协议文档 `d:\doc\Project\Rak生态\Rak\Tec\docs\协议\RakMessage-v0.md`

- What：沉淀全系统统一消息模型。
- Why：`dev\quic.md` 已给出核心雏形，但目前没有正式协议文档承接，跨团队容易各自发明字段。
- How：
  - 定义 `type / trace_id / source / target / action / params / data / timestamp` 等字段。
  - 给出 `action / state / event` 三类消息的字段约束与使用场景。
  - 规定 trace 透传规则、可选字段规则、版本号写法和兼容性原则。
  - 提供前端请求、Runtime 下发、设备回传、错误上报四类样例。
  - 约束 HTTP、gRPC、MQTT 只是承载层，消息语义以 RakMessage 为准。

### 3. 新建接口文档 `d:\doc\Project\Rak生态\Rak\Tec\docs\接口\云层统一接口-v0.md`

- What：统一前端、Go、Runtime、硬件之间的接口契约。
- Why：当前接口信息散落在 `dev\quic.md` 和零散想法中，无法直接作为多人并行开发标准。
- How：
  - 固定 HTTP 接口：动作执行、设备状态查询、OTA 触发。
  - 固定 WebSocket 推送：设备状态、执行结果、错误事件。
  - 固定 gRPC `RuntimeService.Execute` 的请求响应模型。
  - 固定 MQTT topic：命令、状态、事件、心跳、OTA 的命名规则。
  - 增加 ACK、错误码、超时、幂等、trace 透传、设备离线等边界约定。

### 4. 新建 Runtime 边界文档 `d:\doc\Project\Rak生态\Rak\Tec\docs\运行时\Runtime-MVP边界.md`

- What：给 AI 工程架构师一个清晰的“本轮只做什么”的边界说明。
- Why：当前 `dev\rak.md` 和 `dev\tmp.md` 中 Runtime 想象空间很大，若不收敛，MVP 会被长期架构吞没。
- How：
  - 明确 Fake Runtime 的职责：接收状态、根据规则返回动作、输出统一 RakMessage。
  - 明确不做的能力：长期记忆、复杂规划、多技能学习、完整 NTR。
  - 说明与未来真实 Runtime 的升级接口保持兼容。
  - 规定 Runtime 与 Go 的输入输出字段、错误返回、超时策略。

### 5. 新建环境文档 `d:\doc\Project\Rak生态\Rak\Tec\docs\标准\技术环境标准.md`

- What：把 notebook 里的实验安装记录清洗成工程可执行环境标准。
- Why：当前 `example\技术环境标准.ipynb` 混合安装记录与报错，不适合作为团队统一基线。
- How：
  - 按角色拆分最小环境：前端、Go 云层、AI Runtime、硬件联调。
  - 说明 Python 项目使用 `uv` 或 `conda`，不使用系统环境；涉及 PyTorch/CUDA 的 Runtime 方案优先保留 conda 路线。
  - 说明 Node 项目统一用 `pnpm`。
  - 区分“必需依赖”和“后续扩展依赖”，避免一次装满所有栈。
  - 给出最小验收方式，而不是直接保留 notebook 执行输出。

### 6. 新建物联网接入文档 `d:\doc\Project\Rak生态\Rak\Tec\docs\标准\物联网接入标准.md`

- What：把硬件接入、topic、状态回传和 OTA 相关约定正式化。
- Why：当前 `example\物联网标准.ipynb` 只有连接示例，不足以指导硬件工程师与 Go 云层对齐。
- How：
  - 规定设备 ID、topic 命名、上线/离线/心跳/状态/事件/命令的最小集合。
  - 用 RakMessage 统一命令与状态 payload。
  - 说明设备侧最小执行引擎责任：订阅命令、执行动作、上报状态。
  - 补充 OTA 触发与回执的控制流。
  - 将 notebook 中关闭 TLS 校验的示例降级为“调试示例”，不作为正式标准。

### 7. 新建协作文档 `d:\doc\Project\Rak生态\Rak\Tec\docs\协作\MVP两周对齐表.md`

- What：把团队角色、交付边界、接口 owner、联调顺序和验收口径写成明确清单。
- Why：用户明确要求“各部分文档对齐、接口对齐、格式对齐、控制流成立”，单有技术文档不足以推动并行执行。
- How：
  - 为四个角色定义 owner、输入、输出、依赖对象。
  - 列出第 1 周和第 2 周的文档/接口/联调目标。
  - 给出接口冻结点、冒烟联调点、最终验收点。
  - 形成“一人产出、谁消费、何时对齐”的责任链。

### 8. 补齐示例索引 `d:\doc\Project\Rak生态\Rak\Tec\example\README.md`

- What：为 `example/` 提供一个轻量索引，并重新定义它的角色。
- Why：当前 `example/` 同时混放实验 notebook 和“标准”命名文档，语义混乱。
- How：
  - 把 `example/` 定义为“最小样例与实验素材区”，不再承担正式规范主入口。
  - 标注每个 notebook 的用途、可信度、后续去向。
  - 引导读者从 `技术框架.md` 和 `docs/` 进入正式规范，从 `example/` 查看辅助示例。

## Execution Order

1. 先写 `技术框架.md`，形成总览、范围、链路和索引。
2. 再写 `RakMessage-v0.md`，冻结统一消息模型。
3. 再写 `云层统一接口-v0.md`，冻结跨角色接口。
4. 同步写 `Runtime-MVP边界.md` 与 `物联网接入标准.md`，锁定 AI/硬件边界。
5. 再清洗 `技术环境标准.md`，作为团队起环境依据。
6. 最后写 `MVP两周对齐表.md` 与 `example/README.md`，把责任和入口收口。

## Verification Steps

- 结构验证：确认 `技术框架.md` 能作为唯一入口，且每个正式子文档都能从主文档跳转。
- 一致性验证：逐项检查 RakMessage 字段、HTTP/gRPC/MQTT 命名、topic、trace 规则在所有文档中完全一致。
- 闭环验证：用一条设备控制链路逐文档走查，确保“前端发什么、Go 收什么、Runtime 回什么、硬件订阅什么、状态怎么回前端”没有断点。
- 边界验证：确认 Fake Runtime 的职责与非职责写清楚，避免 AI 范围膨胀。
- 协作验证：确认四个角色都能从文档中找到自己的 owner 边界、交付物、依赖接口和联调时序。
- 质量验证：清除 notebook 中不适合作为标准的内容表达，如安装报错、实验性依赖堆叠、禁用 TLS 校验等，正式文档只保留可执行规范。
