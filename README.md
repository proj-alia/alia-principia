# Principia · 通用 AI 数字个体核心架构图谱与工程规范库

> **在原版 24 专题基础上完整移除可扩展功能（Live2D / VRM / 桌面移动壳 / 第三方平台与游戏 Bot），100% 保留原版全部核心能力，可直接驱动 Agent 工程开发的权威系统图纸（System Blueprint & Architecture Principia）**
>
> - **后端底层核心语言**：**Rust (`crates/*` Cargo Workspace，共 14 个核心 Crates)** —— 1:1 承载原版全部核心能力（本机宿主与后端 API `alia-server`、跨语言契约 `alia-protocol`、角色定义与配置继承 `alia-core-character`、通用 Agent 运行时 `alia-core-agent`、模型推理与分工路由 `alia-provider-inference`、本地持久状态存储 `alia-storage`、音频与语音管线 `alia-audio-pipeline`、视觉感知管线 `alia-vision-pipeline`、插件与 Kit 协议 `alia-plugin-protocol`、插件扩展宿主 `alia-extension-host`、模块与接入渠道网关 `alia-channel-gateway`、认证服务 `alia-auth-service`、异步生命周期底座 `alia-stream-kit`、全链路执行观测底座 `alia-trace-kit`）。
> - **项目唯一内置前端面板**：**`apps/webui` (`@alia/webui` + 6 个基础状态/音频/Trace 包，Vue 3 + TypeScript)** —— 项目本身只存在这一个 WebUI 控制台面板，通过 `REST + SSE` 与本机 Rust 实例实时同步本地持久状态。在 WebUI 面板内提供**测试沙盒（直接对话体验）**、**身份与记忆管理**、**模型服务（含最多 6 路并发探活与日常对话/后台活动/内部推理/目标决策四维使用分工）**、**接入渠道与扩展插件管理**、**自动化任务与权限审批**及**全链路 Trace 诊断**。
> - **万物皆插件扩展边界**：除上述核心能力与唯一内置 `apps/webui` 面板外，其他一切事物（外部聊天客户端、2D/3D 形象渲染、直播、游戏等）均只能通过 `alia-extension-host`（专题 `17/18`）与 `alia-channel-gateway`（专题 `19`）以插件形式配置和扩展。

---

## 一、核心产物导航

| 产物文件 | 内容与核心用途 |
|---|---|
| **[Principia-Architecture-Atlas.html](./Principia-Architecture-Atlas.html)** | **推荐首先打开（单文件离线交互式系统图纸）**。内嵌 **23 个核心架构专题、46 张可缩放矢量图（职责组织图 + 调用与数据流图）、479 个带状态归属与上下游检查器（Inspector）的代码符号节点、源码组织树、22 个纯净核心工作空间清单、24 条架构边界发现、9 条典型运行次序与全局秒级检索**。 |
| **[Principia-Architecture-Specification.md](./Principia-Architecture-Specification.md)** | **全景架构设计规范正文（23 章完整版）**。包含原版裁剪与保留对照表、四维读图法则、22 个工作空间依赖表、23 个专题的全量节点/状态/关系登记表、24 条边界发现与 9 条运行次序。 |
| **[architecture-data.json](./architecture-data.json)** | **机器可读唯一架构事实源（SSOT）**（`sections: 23`, `nodes: 479`, `edges: 525`, `source_files: 229`, `workspaces: 22`, `findings: 24`, `sequences: 9`）。 |
| **[source-index.csv](./source-index.csv)** | **全量符号与源码路径索引表（UTF-8 BOM，479 行）**。 |
| **[Organization-Overview.svg](./Organization-Overview.svg) / [Organization-Expanded.svg](./Organization-Expanded.svg)** | **全局工程组织海报图与全对象级展开总树（SVG）**。 |
| **[diagrams/](./diagrams/)** | **23 个专题的成对矢量图（`*-organization.svg` / `*-relations.svg`）与可编辑 Graphviz 图源（`*.dot`）**，共 92 个图表文件。 |
| **[AGENTS.md](./AGENTS.md) / [CLAUDE.md](./CLAUDE.md)** | **Agent 工程开发施工手册**。 |

---

## 二、23 个核心架构专题总览矩阵

| 编号 | 专题标识 (`id`) | 专题名称 | 节点数 | 关系数 | 对应原版关系 |
|---|---|---|---|---|---|
| `01` | `organization` | **全局组织与职责边界（原版核心能力保留 + 唯一内置 WebUI 面板 + 插件扩展底座）** | 20 | 23 | 原版核心保留 |
| `02` | `platform-hosts` | **应用宿主与启动装配（alia-server 本机宿主 + apps/webui 面板初始化）** | 19 | 19 | 原版核心保留 |
| `03` | `character` | **角色定义、设定文本与模块配置继承解析** | 9 | 10 | 原版核心保留 |
| `04` | `runtime_host` | **会话宿主：宿主如何装配通用 Agent** | 24 | 27 | 原版核心保留 |
| `05` | `runtime_schedule` | **调度器内部：按会话排队、提交、中断与收尾** | 24 | 28 | 原版核心保留 |
| `06` | `runtime_domain` | **领域数据：Conversation、Turn、Round、Tool 与上下文桶** | 22 | 23 | 原版核心保留 |
| `07` | `runtime_providers` | **推理边界：定义、配置、实例、协议与工具轮次** | 24 | 26 | 原版核心保留 |
| `08` | `runtime_tools` | **工具与输出：注册定义、执行函数、流事件和表现标记** | 23 | 28 | 原版核心保留 |
| `09` | `runtime_memory` | **记忆与技能：条目、决策、后台任务与真实边界** | 24 | 24 | 原版核心保留 |
| `10` | `runtime_storage` | **会话数据所有权：窗口选择、本地记录、媒体与云同步** | 22 | 22 | 原版核心保留 |
| `11` | `spark` | **主动反应旁路：Spark、Notebook与Character Orchestrator** | 20 | 23 | 原版核心保留 |
| `12` | `vision` | **视觉感知的两条入口与并发边界** | 16 | 17 | 原版核心保留 |
| `13` | `voice_host` | **语音宿主、控制总线与设备归属** | 20 | 23 | 原版核心保留 |
| `14` | `voice_input` | **VoiceController内部：输入文档、插件与决策规则** | 23 | 28 | 原版核心保留 |
| `15` | `voice_output` | **语音输出：生产者、合成、播放与打断** | 22 | 25 | 原版核心保留 |
| `16` | `expression` | **流控制、内联动作标记解析与情绪语义信号分发** | 7 | 6 | 原版核心保留 |
| `17` | `platform-extension-host` | **Extension Host 内部组织与状态所有权** | 26 | 31 | 原版核心保留 |
| `18` | `platform-kits-protocol` | **Kit 契约、身份、通道与实现边界** | 25 | 24 | 原版核心保留 |
| `19` | `platform-channel` | **模块 WebSocket 通道：连接、身份、路由和资源** | 23 | 30 | 原版核心保留 |
| `20` | `platform-backend` | **Hosted Backend：产品资源、认证、同步与模型网关** | 26 | 31 | 原版核心保留 |
| `21` | `lifetime` | **异步任务、资源生存期与事件流底座** | 23 | 13 | 原版核心保留 |
| `22` | `observability` | **执行观测：生产、运输、采集、投影、回放** | 23 | 26 | 原版核心保留 |
| `23` | `webui-console` | **项目唯一内置前端：WebUI 控制台面板（测试沙盒对话、配置中心与插件扩展管理）** | 14 | 18 | 原版核心保留 |
