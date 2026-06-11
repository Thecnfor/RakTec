# 仓库对齐：README 与 docs（Plan）

## Summary

目标：把 RakTec 中已经冻结的 MVP 边界、协议与字段口径，分发成 **三个工程仓库**（go-kernel / rak-esp / rak-runtime）各自能直接开工的 README + docs 结构，并且三方接口契约一致、互相可对齐。

范围：

* ✅ 对齐 Go 中枢（/home/xrak/Desktop/C4/go-kernel/）

* ✅ 对齐 ESP 硬件（/home/xrak/Desktop/C4/rak-esp/）

* ✅ 对齐 Runtime（/home/xrak/Desktop/C4/rak-runtime/）

* ❌ 暂不处理前端仓库与前端 README

硬性要求（来自需求）：

* 每个仓库的 README.md 明确：该角色的方向（按角色名，不写人名）、边界、向谁提供/依赖什么

* 每个仓库新增统一的 docs/：放“该角色应该看到的”架构说明 + 冻结协议（可细，但清楚）+ RakMessage v0 的字段统一设计（MVP 仅保留最需要字段）

* 三个仓库整体保持：简洁 + 高效 + 统一（目录结构、命名、文档口径一致）

* 删除无用内容（例如多余的 copy README、重复且无价值的文案）

## Current State Analysis（基于已探索的仓库现状）

RakTec（核心协调文档）：

* 已提供 MVP 总入口与索引：[技术框架](file:///home/xrak/Desktop/C4/RakTec/技术框架.md)

* 已提供三角色入口文档：[go.md](file:///home/xrak/Desktop/C4/RakTec/go.md)、[runtime.md](file:///home/xrak/Desktop/C4/RakTec/runtime.md)、[esp.md](file:///home/xrak/Desktop/C4/RakTec/esp.md)

* 已提供冻结协议与字段口径：

  * [RakMessage-v0](file:///home/xrak/Desktop/C4/RakTec/docs/协议/RakMessage-v0.md)

  * [云层统一接口-v0](file:///home/xrak/Desktop/C4/RakTec/docs/接口/云层统一接口-v0.md)

  * [Runtime-MVP边界](file:///home/xrak/Desktop/C4/RakTec/docs/运行时/Runtime-MVP边界.md)

三个工程仓库现状：

* /go-kernel：只有空 README.md（0 行），暂无 docs/

* /rak-esp：README.md 只有标题，存在 README copy.md（疑似冗余），暂无 docs/

* /rak-runtime：README.md 只有标题，存在 README copy.md（疑似冗余），已存在 gRPC proto：[runtime.proto](file:///home/xrak/Desktop/C4/rak-runtime/protos/runtime.proto)，暂无 docs/

## Decisions & Assumptions（已确认/将采用）

* README 的 Owner：只写角色（Go 中枢 / ESP 硬件 / Runtime）

* 文档目录名：统一使用 docs/

* MVP 最小动作集：由我选择，并以“最短联调闭环”为准，剔除 OTA / reboot 等非首要动作

### MVP 最小动作集（本计划拟定）

为保证“动作请求 → gRPC → MQTT → 状态回传”的最短闭环，同时避免设备侧实现成本过高：

* `shake_head`（演示类，参数简单）

* `wave_hand`（演示类，可测试 params 传递）

* `lock_open`（业务类）

* `lock_close`（业务类）

不进入 MVP v0 开发必做字段/动作：

* `device_ota`（只保留接口占位，不进入必须实现的 RakMessage action）

* `device_reboot`（同上）

## Proposed Changes（具体要改什么：文件级别）

所有仓库统一“入口/结构/命名”，让新成员不迷路：

* 根 README.md：只放“角色方向 + 边界 + 对齐契约 + 快速入口链接”

* docs/：只放“该角色需要的”内容；重复信息用引用 RakTec 作为权威来源，但仍提供本角色可直接使用的最小字段/样例

### 1) go-kernel（Go 中枢）

#### 新增/修改文件

* 修改 [go-kernel/README.md](file:///home/xrak/Desktop/C4/go-kernel/README.md)

  * 写清楚：Go 中枢职责/不负责、对外接口（HTTP/WS）、对内接口（gRPC/MQTT）、错误与幂等等“对齐点”

  * 放 3 个“必须对齐”的链接到本仓库 docs（而不是长文堆在 README）

  * 明确：Go 是协议落地与汇聚点，所有字段语义以 RakTec 为权威来源

* 新增 docs 目录与最小文档集（统一命名）

  * 新增 go-kernel/docs/INDEX.md：本仓库 docs 入口与索引（统一模板）

  * 新增 go-kernel/docs/architecture.md：Go 处理链路与模块边界（不写实现细节）

  * 新增 go-kernel/docs/contracts.md：冻结接口契约（HTTP/WS/gRPC/MQTT 的“Go 视角”）

  * 新增 go-kernel/docs/rakmessage-mvp.md：RakMessage v0（MVP 必要字段子集）+ 三种消息（action/state/error）样例（只保留必需字段）

#### 关键对齐内容（Go 仓库需要明确）

* HTTP：`POST /api/v1/action/execute`（请求体是 RakMessage/action；响应只返回 accepted/trace\_id/device\_id/status）

* WebSocket：推送 body 为 RakMessage 的 `state/event/error`（MVP 先对齐 state + error）

* gRPC：以 runtime.proto 为准，但字段语义与 RakTec 的云层统一接口一致

* MQTT：topic 固定 `rak/{device_id}/cmd|state|event|heartbeat`，payload 统一 RakMessage（MVP 先对齐 cmd 与 state）

### 2) rak-esp（ESP 硬件）

#### 新增/修改/删除文件

* 修改 [rak-esp/README.md](file:///home/xrak/Desktop/C4/rak-esp/README.md)

  * 写清楚：设备端职责/不负责、最小动作集、必须透传 trace\_id、必须回传 state

  * 链接到 docs 入口与“MQTT + payload”冻结契约

* 删除冗余文件

  * 删除 rak-esp/README copy.md（若确认内容与 README.md 重复或无用；执行阶段先对比内容再删）

* 新增 docs 目录与最小文档集

  * 新增 rak-esp/docs/INDEX.md

  * 新增 rak-esp/docs/architecture.md：设备端处理流程（订阅 cmd → 执行动作 → 回传 state）

  * 新增 rak-esp/docs/mqtt-contracts.md：topic + payload 规范（MVP 只保留 cmd/state/error 的必要字段）

  * 新增 rak-esp/docs/rakmessage-mvp.md：RakMessage MVP 字段子集（从设备视角：cmd 解析、state 回传最小字段）

#### 关键对齐内容（ESP 仓库需要明确）

* 必须支持：订阅 `rak/{device_id}/cmd`，解析 RakMessage/action

* 必须回传：`rak/{device_id}/state`，payload RakMessage/state，带原 trace\_id

* MVP 不强制：event/heartbeat/ota，但 docs 里保留“占位接口说明”一段即可（不引入到必须字段与动作集）

### 3) rak-runtime（Runtime）

#### 新增/修改/删除文件

* 修改 [rak-runtime/README.md](file:///home/xrak/Desktop/C4/rak-runtime/README.md)

  * 写清楚：Fake Runtime 的职责/不负责、输入输出字段、错误码、与 Go 的对齐点

  * README 不写实现细节，把“接口契约”下放到 docs

* 删除冗余文件

  * 删除 rak-runtime/README copy.md（执行阶段先对比内容再删）

* 新增 docs 目录与最小文档集

  * 新增 rak-runtime/docs/INDEX.md

  * 新增 rak-runtime/docs/architecture.md：Fake Runtime 的决策模式与边界（与 RakTec 一致但更贴近实现）

  * 新增 rak-runtime/docs/grpc-contracts.md：runtime.proto 的语义解释 + 请求/响应样例 + 错误码

  * 新增 rak-runtime/docs/rakmessage-mvp.md：从 Runtime 输出到下游 RakMessage/action 的映射规则（只保留 MVP 必需字段）

#### 关键对齐内容（Runtime 仓库需要明确）

* gRPC proto 以仓库现有文件为权威：[runtime.proto](file:///home/xrak/Desktop/C4/rak-runtime/protos/runtime.proto)

* status=error 时 Go 不得下发 MQTT

* action 必须落在 available\_actions 内（防止 Runtime 输出“幽灵动作”）

* params\_json 必须可直接映射为 RakMessage.params（避免双方重新设计结构）

## Unification Rules（让三个仓库“简洁+统一”的约束）

* 统一目录：`docs/`

* 统一入口：`docs/INDEX.md`

* 统一命名：`architecture.md`、`contracts*.md`、`rakmessage-mvp.md`

* 统一文档结构（每个 docs 都包含）：

  * 一句话定位

  * 输入/输出（对齐字段）

  * 对齐依赖（需要谁提供什么）

  * MVP 必做 / 不做（清单式，避免长文）

  * 最小样例（1-2 个 JSON 或 proto 片段）

* 统一“权威来源”规则：细节以 RakTec 为主文档；工程仓库 docs 只保留该角色“开工必须信息”+ 本角色样例

## Verification（验收与自检方式）

执行完成后以以下方式自检：

* 三个仓库 README.md 都能在 60 秒内回答：

  * 我是谁（角色方向）/我不做什么

  * 我对外提供什么接口

  * 我依赖谁（Go/Runtime/ESP）以及依赖的字段口径

* 三个仓库 docs/INDEX.md 都能从索引跳转到：

  * architecture

  * contracts

  * rakmessage-mvp

* RakMessage MVP 子集在三仓库表述一致（字段名/必填规则一致）

* 删除 README copy.md 后仓库更干净，且信息不丢失（删除前先对比内容）

## Execution Steps（实现顺序）

1. 对比并清理冗余 README copy.md（确保无信息丢失）
2. 为三个仓库创建统一 docs/ 与最小文档集（按上面文件清单）
3. 重写三个仓库 README.md：统一模板 + 指向 docs/INDEX.md + 指向 RakTec 权威文档
4. 全量检查链接与字段一致性（尤其 trace\_id / type / action / params / data 的语义）

