# Principia · 通用 AI 数字个体全栈架构规范库

> **纯粹、自包含、权威、可直接驱动 Agent 工程开发的系统蓝图（System Blueprint & Architecture Principia）**
>
> - **后端核心运行时**：**Rust (`crates/*` Cargo Workspace，共 14 个核心 Crates)** —— `alia-server`、`alia-protocol`、`alia-core-character`、`alia-core-agent`、`alia-provider-inference`、`alia-storage`、`alia-audio-pipeline`、`alia-vision-pipeline`、`alia-plugin-protocol`、`alia-extension-host`、`alia-channel-gateway`、`alia-auth-service`、`alia-stream-kit`、`alia-trace-kit`。
> - **唯一内置前端控制台**：**`apps/webui` (`@alia/webui` + 7 个核心状态/音频/Trace 基础包，Vue 3 + TypeScript)** —— 通过 `REST + SSE` 实时同步本地持久状态。在 WebUI 面板内提供**工作台（运行概览、测试沙盒直接对话）**、**数字个体（身份与人格、记忆管理）**、**连接与能力（接入渠道、模型服务［含最多 6 路并发探活与日常对话/后台活动/内部推理/目标决策四维分工］、扩展管理、技能管理）**、**配置与运行（配置文件、自动化任务、数据与日志 Trace 时间轴、权限与审批、系统设置）**。
> - **通用可扩展边界**：除系统核心运行时与唯一内置 `apps/webui` 控制台外，一切外部渠道、呈现端与专业工具统一通过 `alia-extension-host`（专题 `17/18`）与 `alia-channel-gateway`（专题 `19`）以插件形式扩展接入。

---

## 一、核心产物导航

| 产物文件 | 内容与核心用途 |
|---|---|
| **[Principia-Architecture-Atlas.html](./Principia-Architecture-Atlas.html)** | **交互式全栈系统图纸（单文件离线可用）**。内嵌 **23 个正交架构专题、46 张可缩放矢量图（职责组织图 + 调用与数据流图）、479 个带状态归属与上下游检查器（Inspector）的符号节点、源码组织树、22 个工作空间依赖清单、24 条架构不变量、9 条运行次序与全局秒级检索**。 |
| **[Principia-Architecture-Specification.md](./Principia-Architecture-Specification.md)** | **全栈架构设计规范正文（23 章完整版）**。阐述正交分层、四维工程定位法则、契约即 Rust 符号规范、22 个工作空间依赖表、23 个专题的全量节点/状态/关系登记表、24 条不变量与 9 条运行次序。 |
| **[architecture-data.json](./architecture-data.json)** | **机器可读唯一架构事实源（SSOT）**（`sections: 23`, `nodes: 479`, `edges: 525`, `source_files: 231`, `workspaces: 22`, `findings: 24`, `sequences: 9`）。 |
| **[source-index.csv](./source-index.csv)** | **全量符号与源码路径索引表（UTF-8 BOM，479 行）**。 |
| **[Organization-Overview.svg](./Organization-Overview.svg) / [Organization-Expanded.svg](./Organization-Expanded.svg)** | **全局工程组织海报图与 23 专题展开总树（SVG）**。 |
| **[diagrams/](./diagrams/)** | **23 个专题的成对矢量图（`*-organization.svg` / `*-relations.svg`）与可编辑 Graphviz 图源（`*.dot`）**，共 92 个图表文件。 |
| **[AGENTS.md](./AGENTS.md) / [CLAUDE.md](./CLAUDE.md)** | **Agent 工程开发施工手册**。 |

---

## 二、23 个架构专题总览矩阵

| 编号 | 专题标识 (`id`) | 专题名称 | 节点数 | 关系数 | 架构分层归属 |
|---|---|---|---|---|---|
| `01` | `organization` | **全局组织与职责边界（Rust 核心运行时 + 唯一内置 WebUI 面板 + 插件扩展底座）** | 20 | 23 | 全局组织、本机宿主与唯一内置 WebUI 面板层 |
| `02` | `platform-hosts` | **应用宿主与启动装配（alia-server 本机宿主 + apps/webui 控制台初始化）** | 19 | 19 | 全局组织、本机宿主与唯一内置 WebUI 面板层 |
| `03` | `character` | **角色定义、设定文本与模块配置继承解析** | 9 | 10 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `04` | `runtime_host` | **会话宿主：宿主如何装配通用 Agent** | 24 | 27 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `05` | `runtime_schedule` | **调度器内部：按会话排队、提交、中断与收尾** | 24 | 28 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `06` | `runtime_domain` | **领域数据：Conversation、Turn、Round、Tool 与上下文桶** | 22 | 23 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `07` | `runtime_providers` | **推理边界：定义、配置、实例、协议与工具轮次** | 24 | 26 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `08` | `runtime_tools` | **工具与输出：注册定义、执行函数、流事件和表现标记** | 23 | 28 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `09` | `runtime_memory` | **记忆与技能：条目、决策、后台任务与真实边界** | 24 | 24 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `10` | `runtime_storage` | **会话数据所有权：窗口选择、本地记录、媒体与云同步** | 22 | 22 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `11` | `spark` | **主动反应旁路：Spark、Notebook与Character Orchestrator** | 20 | 23 | 角色配置、通用 Agent 调度、推理路由、记忆与存储层 |
| `12` | `vision` | **视觉感知的两条入口与并发边界** | 16 | 17 | 多模态视觉/听觉管线、VoiceController 与语义表达总线层 |
| `13` | `voice_host` | **语音宿主、控制总线与设备归属** | 20 | 23 | 多模态视觉/听觉管线、VoiceController 与语义表达总线层 |
| `14` | `voice_input` | **VoiceController内部：输入文档、插件与决策规则** | 23 | 28 | 多模态视觉/听觉管线、VoiceController 与语义表达总线层 |
| `15` | `voice_output` | **语音输出：生产者、合成、播放与打断** | 22 | 25 | 多模态视觉/听觉管线、VoiceController 与语义表达总线层 |
| `16` | `expression` | **流控制、内联动作标记解析与情绪语义信号分发** | 7 | 6 | 多模态视觉/听觉管线、VoiceController 与语义表达总线层 |
| `17` | `platform-extension-host` | **Extension Host 内部组织与状态所有权** | 26 | 31 | 通用插件宿主、Kit 契约、模块网关与后端服务层 |
| `18` | `platform-kits-protocol` | **Kit 契约、身份、通道与实现边界** | 25 | 24 | 通用插件宿主、Kit 契约、模块网关与后端服务层 |
| `19` | `platform-channel` | **模块 WebSocket 通道：连接、身份、路由和资源** | 23 | 30 | 通用插件宿主、Kit 契约、模块网关与后端服务层 |
| `20` | `platform-backend` | **Hosted Backend：产品资源、认证、同步与模型网关** | 26 | 31 | 通用插件宿主、Kit 契约、模块网关与后端服务层 |
| `21` | `lifetime` | **异步任务、资源生存期与事件流底座** | 23 | 13 | 异步生命周期底座与全链路执行观测层 |
| `22` | `observability` | **执行观测：生产、运输、采集、投影、回放** | 23 | 26 | 异步生命周期底座与全链路执行观测层 |
| `23` | `webui-console` | **项目唯一内置前端：WebUI 控制台面板（测试沙盒对话、配置中心与插件扩展管理）** | 14 | 18 | 全局组织、本机宿主与唯一内置 WebUI 面板层 |
