# Principia · 通用 AI 数字个体核心架构设计规范（原版核心能力保留 + 可扩展功能剥离 + 唯一内置 WebUI 面板）

> **定位声明**：本仓库是一套**纯粹、自包含、零历史包袱、可直接驱动 Agent 工程开发的权威架构规范库（System Blueprint & Architecture Principia）**。它严格以原版架构图谱为基准，**将其中一切属于“可扩展功能”的部分（Live2D、VRM、MMD/Spine/Tachie、口型/微动驱动、Tamagotchi 桌面壳、Pocket 移动壳及 Discord/Minecraft/Telegram/Satori 第三方集成）干净移除，100% 保留原版全部核心能力、细粒度节点与调用/数据流关系**。后端底层统一采用 **Rust Cargo Workspace (`crates/*`)**，前端仅保留**唯一内置的 `apps/webui` 控制台面板**（用于在 WebUI 中完成配置、在测试沙盒中直接对话体验、并通过插件与通道底座进行一切扩展）。全库共包含 **22 个纯净工作空间**、**23 个核心架构专题**、**479 个代码符号节点**、**525 条有向类型化关系边**、**24 条架构边界守则** 以及 **9 条端到端运行次序**。

---

## 产物清单与协同使用方式

| 产物文件 | 类型 | 核心用途 |
|---|---|---|
| [Principia-Architecture-Atlas.html](./Principia-Architecture-Atlas.html) | 单文件离线交互式图纸 | **首选工作台**。内嵌 23 个专题的职责组织图（`-organization.svg`）、调用与数据流图（`-relations.svg`）、节点登记表、右侧节点详情检查器（Inspector：查看导出符号、职责、状态归属、上游调用者、下游被调者）、源码组织树（`crates/*` + `apps/webui` + `packages/*`）、22 个工作空间依赖清单、24 条架构边界、9 条运行次序与全局秒级检索。 |
| [architecture-data.json](./architecture-data.json) | 机器可读唯一事实源（SSOT） | 供开发 Agent 与分析工具直接读取的结构化架构数据库，包含 `sections`（23）、`inventory`（22）、`findings`（24）、`sequences`（9）与统计元数据。 |
| [source-index.csv](./source-index.csv) | 全量符号与源码路径索引 | UTF-8 BOM 表格，按 `专题, 节点ID, 名称, 职责分组, 源码路径, 行, 符号, 职责, 状态, 架构分层, URL` 索引全部 479 个节点实例（跨 229 个核心源码文件）。 |
| [Organization-Overview.svg](./Organization-Overview.svg) / [Organization-Expanded.svg](./Organization-Expanded.svg) | 全局矢量总图 | 工程工作空间全量分组海报图与对象级展开总树。 |
| [diagrams/](./diagrams/) | 23 专题 `.dot` 与 `.svg` 图集 | 每个专题成对提供 `*-organization.dot/.svg`（职责分组树）与 `*-relations.dot/.svg`（调用/数据/事件/生命周期流），共 92 个图表文件。 |
| [Principia-Architecture-Specification.md](./Principia-Architecture-Specification.md) | 全景架构设计规范正文 | 本文档。阐述原版核心能力保留与可扩展功能剥离原则、Rust + 唯一内置 WebUI 面板架构、四维读图法则、契约即 Rust 符号（Contract-as-Rust-Symbol + Specta），以及 23 个专题的完整对象与关系细表。 |
| [AGENTS.md](./AGENTS.md) / [CLAUDE.md](./CLAUDE.md) | Agent 工程开发施工准则 | 规定编码 Agent 如何按图索骥、定位 Rust Crates 与 WebUI 包、遵守“核心保留 + 插件扩展”铁律。 |

---

## 第一部分：架构裁剪与核心保留准则 —— 如何在原版基础上剥离可扩展功能并保留全部核心能力？

### 1.1 原版 24 专题的裁剪与保留边界

| 处置类别 | 原版专题范围 | 工程处置与边界说明 |
|---|---|---|
| **整章移除的可扩展专题** | 原专题 `17 vrm`（VRM 3D 形象）<br>原专题 `22 platform-integrations`（Discord / Minecraft / Telegram / Satori） | 3D 形象渲染与具体第三方平台/游戏 Bot 均属于外部可扩展事物，从核心仓库中整章剔除，零内置。 |
| **剔除可扩展节点、保留核心能力的专题** | 原专题 `01 organization`、`02 platform-hosts`、`03 character`、`16 expression` | 1. 在 `01/02` 中移除 `Tamagotchi`、`Pocket`、`Live2D`、`VRM/Three`、`MMD/Spine/Tachie` 与第三方 Bot 节点，保留 `alia-server` 本机宿主与唯一内置的 `apps/webui` 面板；<br>2. 在 `03 character` 中**完整保留**角色卡（`AliaCard`）、设定文本与模块配置继承解析（`getModules`），移除 9 个绑定 Live2D/VRM 的模型读取与策略节点；<br>3. 在 `16 expression` 中**完整保留**流式输出控制与内联标记解析（`StreamingControl`、`createStreamingControlParser`、`normalizeActPayload`、`EmotionPayload`），移除 17 个 Live2D/VRM 内部渲染节点。 |
| **100% 完整保留的原版核心能力专题** | 原专题 `04 runtime_host`、`05 runtime_schedule`、`06 runtime_domain`、`07 runtime_providers`、`08 runtime_tools`、`09 runtime_memory`、`10 runtime_storage`、`11 spark`、`12 vision`、`13 voice_host`、`14 voice_input`、`15 voice_output`、`18 platform-extension-host`、`19 platform-kits-protocol`、`20 platform-channel`、`21 platform-backend`、`23 lifetime`、`24 observability` | 原版的所有核心调度、会话域、上下文注册表、模型服务与工具循环、长期记忆与 Recipe 技能、本地持久存储、Spark 与 Notebook 笔记、视觉与语音控制管线、`ExtensionHost` 插件宿主、`Kit` 契约、模块 WebSocket 通道网关、`stream-kit` 生命周期底座与 `trace-kit` 执行观测底座 **100% 原汁原味保留**，并落地于 Rust `crates/*`。 |
| **唯一内置前端面板专题** | 专题 `23 webui-console`（`apps/webui`） | 项目本身只存在一个内置 WebUI 控制台面板（`apps/webui`），通过 `REST + SSE` 与本机 Rust 实例实时同步。面板提供**工作台（运行概览、测试沙盒直接对话）**、**数字个体（身份与人格、记忆管理）**、**连接与能力（接入渠道、模型服务、扩展管理、技能管理）**、**配置与运行（配置文件、自动化任务、数据与日志、权限与审批、系统设置）**，所有其他功能只能通过插件扩展。 |

---

### 1.2 阅读与施工的四维架构法则

针对图中或表中任何一个模块与符号，开发 Agent 必须同时核对以下四维正交属性：

1. **代码归属（Physical Crate / Package & File）**：该符号位于 `crates/*` 的哪个 `.rs` 文件，还是 `apps/webui` / `packages/*` 的哪个 `.ts` / `.vue` 文件？
2. **调用与数据流（Call, Data & Event Flow）**：上游是谁、下游是谁？走的是同步/异步调用（`call`）、数据传递（`data`）、事件广播（`event`）、契约约束（`contract`）还是持有归属（`owns`）？
3. **状态与生命周期（State Ownership & Lifecycle）**：谁持有状态（Rust `Arc<AliaServerState>`、Tokio `Scope`、SQLx 持久层，还是前端 Pinia Store）？何时初始化、何时销毁？
4. **运行与部署边界（Process & Runtime Boundary）**：它运行在 Rust `alia-server` 本机主进程、插件扩展宿主（`ExtensionHost`），还是浏览器 `apps/webui` 控制台面板？

---

## 第二部分：工程工作空间总目录（22 个纯净核心 Workspaces 与内部依赖 DAG）

| 序号 | 生态分类 | 真实目录路径 | 工作空间包名 / Crate 名 | 内部依赖声明 (`Cargo.toml` / `workspace:`) |
|---|---|---|---|---|
| 01 | 唯一内置 WebUI 面板 (`apps/*`) | `apps/webui` | `@alia/webui` | `@alia/stage-ui` (dependencies), `@alia/stage-shared` (dependencies), `@alia/audio-web` (dependencies), `@alia/ui` (dependencies), `@alia/ui-trace` (dependencies) |
| 02 | Rust 核心 Crate (`crates/*`) | `crates/alia-server` | `alia-server` | `alia-protocol` (Cargo.toml), `alia-core-agent` (Cargo.toml), `alia-core-character` (Cargo.toml), `alia-provider-inference` (Cargo.toml), `alia-storage` (Cargo.toml), `alia-audio-pipeline` (Cargo.toml), `alia-vision-pipeline` (Cargo.toml), `alia-extension-host` (Cargo.toml), `alia-channel-gateway` (Cargo.toml), `alia-auth-service` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 03 | Rust 核心 Crate (`crates/*`) | `crates/alia-protocol` | `alia-protocol` | 无内部工作空间依赖（基础层） |
| 04 | Rust 核心 Crate (`crates/*`) | `crates/alia-core-character` | `alia-core-character` | `alia-protocol` (Cargo.toml) |
| 05 | Rust 核心 Crate (`crates/*`) | `crates/alia-core-agent` | `alia-core-agent` | `alia-protocol` (Cargo.toml), `alia-core-character` (Cargo.toml), `alia-stream-kit` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 06 | Rust 核心 Crate (`crates/*`) | `crates/alia-provider-inference` | `alia-provider-inference` | `alia-protocol` (Cargo.toml), `alia-core-agent` (Cargo.toml), `alia-stream-kit` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 07 | Rust 核心 Crate (`crates/*`) | `crates/alia-storage` | `alia-storage` | `alia-protocol` (Cargo.toml), `alia-core-character` (Cargo.toml), `alia-core-agent` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 08 | Rust 核心 Crate (`crates/*`) | `crates/alia-audio-pipeline` | `alia-audio-pipeline` | `alia-protocol` (Cargo.toml), `alia-core-agent` (Cargo.toml), `alia-provider-inference` (Cargo.toml), `alia-stream-kit` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 09 | Rust 核心 Crate (`crates/*`) | `crates/alia-vision-pipeline` | `alia-vision-pipeline` | `alia-protocol` (Cargo.toml), `alia-provider-inference` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 10 | Rust 核心 Crate (`crates/*`) | `crates/alia-plugin-protocol` | `alia-plugin-protocol` | `alia-protocol` (Cargo.toml) |
| 11 | Rust 核心 Crate (`crates/*`) | `crates/alia-extension-host` | `alia-extension-host` | `alia-plugin-protocol` (Cargo.toml), `alia-core-agent` (Cargo.toml), `alia-stream-kit` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 12 | Rust 核心 Crate (`crates/*`) | `crates/alia-channel-gateway` | `alia-channel-gateway` | `alia-plugin-protocol` (Cargo.toml), `alia-extension-host` (Cargo.toml), `alia-stream-kit` (Cargo.toml), `alia-trace-kit` (Cargo.toml) |
| 13 | Rust 核心 Crate (`crates/*`) | `crates/alia-auth-service` | `alia-auth-service` | `alia-protocol` (Cargo.toml), `alia-storage` (Cargo.toml) |
| 14 | Rust 核心 Crate (`crates/*`) | `crates/alia-stream-kit` | `alia-stream-kit` | 无内部工作空间依赖（基础层） |
| 15 | Rust 核心 Crate (`crates/*`) | `crates/alia-trace-kit` | `alia-trace-kit` | `alia-stream-kit` (Cargo.toml) |
| 16 | 前端核心与诊断包 (`packages/*`) | `packages/stage-shared` | `@alia/stage-shared` | 无内部工作空间依赖（基础层） |
| 17 | 前端核心与诊断包 (`packages/*`) | `packages/stage-ui` | `@alia/stage-ui` | `@alia/stage-shared` (dependencies), `@alia/audio-web` (dependencies), `@alia/ui` (dependencies) |
| 18 | 前端核心与诊断包 (`packages/*`) | `packages/audio-web` | `@alia/audio-web` | `@alia/stage-shared` (dependencies) |
| 19 | 前端核心与诊断包 (`packages/*`) | `packages/ui` | `@alia/ui` | 无内部工作空间依赖（基础层） |
| 20 | 前端核心与诊断包 (`packages/*`) | `packages/timeline-canvas` | `@alia/timeline-canvas` | 无内部工作空间依赖（基础层） |
| 21 | 前端核心与诊断包 (`packages/*`) | `packages/ui-timeline` | `@alia/ui-timeline` | `@alia/timeline-canvas` (dependencies) |
| 22 | 前端核心与诊断包 (`packages/*`) | `packages/ui-trace` | `@alia/ui-trace` | `@alia/ui-timeline` (dependencies), `@alia/stage-shared` (dependencies) |

---

## 第三部分：23 个核心架构专题全景展开（节点、符号、状态归属与调用/数据流）

### 专题 01：全局组织与职责边界（原版核心能力保留 + 唯一内置 WebUI 面板 + 插件扩展底座） (`organization`)

> **专题定位**：基于原版全局组织图，完整移除所有可扩展外围事物（Live2D、VRM、MMD/Spine/Tachie、Tamagotchi 桌面壳、Pocket 移动壳及 Discord/Minecraft/Telegram/Satori 专用集成），100% 保留通用 AI 核心能力（角色与配置继承、通用 Agent 运行时、模型推理与分工路由、记忆与存储、语音/视觉核心管线、ExtensionHost 插件宿主、ChannelGateway 通道网关、stream-kit 与 trace-kit），并收敛为唯一的内置 apps/webui 控制台面板。

- **配套矢量图纸**：[职责组织图 (`01-organization-organization.svg`)](./diagrams/01-organization-organization.svg) ｜ [调用与数据流图 (`01-organization-relations.svg`)](./diagrams/01-organization-relations.svg) ｜ [Graphviz 组织图源](./diagrams/01-organization-organization.dot) ｜ [Graphviz 关系图源](./diagrams/01-organization-relations.dot)
- **直接涉及源码范围**：`Cargo.toml`, `pnpm-workspace.yaml`, `apps/webui`, `crates/alia-server`, `crates/alia-core-agent`, `crates/alia-extension-host`, `crates/alia-channel-gateway`

#### 读图与工程边界守则
- 项目本身只存在一个内置前端面板（apps/webui）与一套 Rust 核心后端（crates/*），通过 REST + SSE 实时同步本地持久状态。
- 原版中的 2D/3D 形象渲染（Live2D / VRM）、独立聊天软件（IM / 独立 WebChat）、直播、游戏等均属于可扩展功能，已从核心架构中彻底移除，未来统一通过 ExtensionHost（专题 17/18）与 ChannelGateway（专题 19）以插件形式扩展。
- 为了利于项目落地与开发者直接体验，apps/webui 面板内部提供“测试沙盒”直接对话功能，并集中管理身份、记忆、接入渠道、模型服务、扩展、技能、配置文件、自动化任务、数据日志与权限审批。

#### 节点与状态归属登记表（共 20 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `O-WEBUI` | 应用宿主 (唯一内置面板) | **apps/webui · 项目唯一内置 WebUI 面板** | `@alia/webui` | `apps/webui/package.json:1` | 项目唯一的内置前端面板：提供工作台（运行概览、测试沙盒直接对话）、数字个体（身份与记忆配置）、连接与能力（接入渠道、模型服务、扩展与技能管理）、配置与运行（配置文件、自动化、日志与权限审批）。 | 浏览器控制台视图与实时同步投影态 | WebUI 面板前端 |
| `O-UI` | 共享前端核心层 | **stage-ui · 前端核心状态编排** | `@alia/stage-ui` | `packages/stage-ui/package.json:1` | 连接 Rust 服务端 REST/SSE/WS，编排会话、模型服务配置、记忆、语音控制、渠道与插件状态（已剥离所有 2D/3D 形象渲染引擎）。 | Pinia 状态 Store | WebUI 面板前端 |
| `O-SHARED` | 共享前端核心层 | **stage-shared · Specta 跨语言契约** | `@alia/stage-shared` | `packages/stage-shared/package.json:1` | 由 Rust crates/alia-protocol 编译期自动导出的 TypeScript 类型契约与状态同步模型。 | 纯类型与序列化契约 | WebUI 面板前端 |
| `O-PRIMITIVE` | 共享前端核心层 | **ui · WebUI 基础组件原语** | `@alia/ui` | `packages/ui/package.json:1` | WebUI 控制台面板的基础表单、卡片、抽屉、弹窗、状态徽标与代码/文本展示原语。 | 无全局业务状态 | WebUI 面板前端 |
| `O-TRACEUI` | 共享前端核心层 | **ui-trace / ui-timeline / timeline-canvas** | `@alia/ui-trace` | `packages/ui-trace/package.json:1` | WebUI 内置的全链路执行观测时间轴与离线 .jsonl 日志回放面板。 | 时间轴视口与选中 Span 快照 | WebUI 面板前端 |
| `O-AGENT` | 运行时与契约 (Rust) | **alia-core-agent · 通用交互运行时** | `crate::alia_core_agent` | `crates/alia-core-agent/Cargo.toml:1` | 原版核心运行时：会话串行队列、generation 代际并发屏障、ContextRegistry、流式输出与内联标记切分、工具轮次、记忆/技能、Spark/Notebook 与 VoiceController。 | Rust 强一致会话运行时状态 | Rust 核心实现 |
| `O-CHAR` | 运行时与契约 (Rust) | **alia-core-character · 角色与继承契约** | `crate::alia_core_character` | `crates/alia-core-character/Cargo.toml:1` | 原版角色核心：定义角色卡（AliaCard）、人设提示词与模块配置继承规则（get_modules）；已剥离具体 Live2D/VRM 渲染绑定。 | 角色卡配置与模块继承解析契约 | Rust 核心实现 |
| `O-PROVIDER` | 运行时与契约 (Rust) | **alia-provider-inference · 推理与任务路由** | `crate::alia_provider_inference` | `crates/alia-provider-inference/Cargo.toml:1` | 原版推理边界：定义 Provider 配置校验、批量并发探活测试、按任务分工路由（日常对话/后台活动/内部推理/目标决策）与流式推理。 | 服务商连接池与健康度状态 | Rust 核心实现 |
| `O-PROTOCOL` | 运行时与契约 (Rust) | **alia-protocol · 跨语言核心协议** | `crate::alia_protocol` | `crates/alia-protocol/Cargo.toml:1` | 定义前后端与跨进程共享的事件、会话、配置与协议数据结构，经 specta 自动导出 TS 绑定。 | 编译期强类型协议契约 | Rust 核心实现 |
| `O-AUDIO` | 多模态感知与语音核心 | **audio-web · 浏览器音频适配** | `@alia/audio-web` | `packages/audio-web/package.json:1` | 浏览器端麦克风 PCM 拾音、重采样与 TTS 音频流播放队列，供 WebUI 测试沙盒语音交互使用。 | WebAudio 上下文与播放队列 | WebUI 面板前端 |
| `O-PIPE` | 多模态感知与语音核心 | **alia-audio-pipeline · 语音处理管线** | `crate::alia_audio_pipeline` | `crates/alia-audio-pipeline/Cargo.toml:1` | 原版语音管线：音频采集转写、VAD 切窗、声纹门控、流式控制标记解析与 TTS 合成调度。 | 音频流管线与取消令牌 | Rust 核心实现 |
| `O-VISION` | 多模态感知与语音核心 | **alia-vision-pipeline · 视觉感知管线** | `crate::alia_vision_pipeline` | `crates/alia-vision-pipeline/Cargo.toml:1` | 原版视觉感知：聊天附图感知、工具读图与并发去重边界。 | 图像描述缓存与单飞锁 | Rust 核心实现 |
| `O-PLUGIN` | 扩展与通道底座 (支撑一切外部插件) | **alia-extension-host · 插件扩展宿主** | `crate::alia_extension_host` | `crates/alia-extension-host/Cargo.toml:1` | 原版 Extension Host：负责插件清单校验、加载器、Session 管理、Provider/Tool/Context/Widget 动态注册事务与权限门禁。 | ExtensionSession 与动态租约表 | Rust 核心实现 |
| `O-KIT-PROTO` | 扩展与通道底座 (支撑一切外部插件) | **alia-plugin-protocol · 插件与 Kit 协议** | `crate::alia_plugin_protocol` | `crates/alia-plugin-protocol/Cargo.toml:1` | 原版 extension-protocol 与 plugin-protocol：定义扩展身份、权限领域、KitContract 与事件路由契约。 | 跨插件边界强类型契约 | Rust 核心实现 |
| `O-ROUTER` | 扩展与通道底座 (支撑一切外部插件) | **alia-channel-gateway · 模块与接入渠道网关** | `crate::alia_channel_gateway` | `crates/alia-channel-gateway/Cargo.toml:1` | 原版 server-runtime / server-sdk：提供标准 WebSocket 模块通道、认证宣告、事件路由与资源桥，一切外部聊天渠道、外部形象客户端、直播或游戏均通过此网关扩展接入。 | 已鉴权模块与外部渠道注册表 | Rust 核心实现 |
| `O-API` | 服务端与持久状态 (Rust) | **alia-server · 本机实例与后端 API 服务** | `crate::alia_server` | `crates/alia-server/Cargo.toml:1` | 原版 Hosted Backend API + 本机宿主：基于 Axum 托管 apps/webui，提供 REST + SSE 实时状态同步、角色/会话/配置 API 并装配全部核心服务。 | Arc<AliaServerState> 单实例权威 | Rust 核心实现 |
| `O-STORAGE` | 服务端与持久状态 (Rust) | **alia-storage · 本地持久状态存储** | `crate::alia_storage` | `crates/alia-storage/Cargo.toml:1` | 原版会话与记忆持久层：基于 SQLx (SQLite-vec / Postgres) 持久化会话、Turn/Round、长期记忆、Notebook 笔记、Recipe 技能与配置。 | SQLx 连接池与事务写锁 | Rust 核心实现 |
| `O-AUTH` | 服务端与持久状态 (Rust) | **alia-auth-service · 身份认证与授权服务** | `crate::alia_auth_service` | `crates/alia-auth-service/Cargo.toml:1` | 原版 Auth Server：提供本机实例访问授权、Token/JWKS 校验与扩展通道鉴权。 | 认证会话与策略表 | Rust 核心实现 |
| `O-STREAM` | 通用底座 (Rust) | **alia-stream-kit · 异步生命周期底座** | `crate::alia_stream_kit` | `crates/alia-stream-kit/Cargo.toml:1` | 原版 stream-kit：提供结构化并发 Task、Outcome、Scope 资源生存期、Latest、BoundedQueue 与 Broadcast 原语。 | Tokio 作用域任务树与流通道 | Rust 核心实现 |
| `O-TRACE` | 通用底座 (Rust) | **alia-trace-kit · 全链路执行观测底座** | `crate::alia_trace_kit` | `crates/alia-trace-kit/Cargo.toml:1` | 原版 trace-kit：提供 TraceSink、TraceSpan、TraceRecord、IOTraceHub 汇聚与 .jsonl 离线录制导出。 | 单调时钟追踪缓冲与订阅流 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 23 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `O-WEBUI` (apps/webui · 项目唯一内置 WebUI 面板) | `O-UI` (stage-ui · 前端核心状态编排) | 包依赖 (`dependency`) | 挂载核心状态编排与测试沙盒 |
| `O-WEBUI` (apps/webui · 项目唯一内置 WebUI 面板) | `O-PRIMITIVE` (ui · WebUI 基础组件原语) | 包依赖 (`dependency`) | 使用基础UI原语构建控制台 |
| `O-WEBUI` (apps/webui · 项目唯一内置 WebUI 面板) | `O-TRACEUI` (ui-trace / ui-timeline / timeline-canvas) | 持有 / 归属 (`owns`) | 内嵌数据与日志Trace时间轴面板 |
| `O-UI` (stage-ui · 前端核心状态编排) | `O-SHARED` (stage-shared · Specta 跨语言契约) | 契约 / 配置 (`contract`) | 使用Specta导出的跨语言契约 |
| `O-SHARED` (stage-shared · Specta 跨语言契约) | `O-PROTOCOL` (alia-protocol · 跨语言核心协议) | 契约 / 配置 (`contract`) | 编译期同步Rust协议定义 |
| `O-UI` (stage-ui · 前端核心状态编排) | `O-API` (alia-server · 本机实例与后端 API 服务) | 调用 (`call`) | REST + SSE 实时状态同步与会话调用 |
| `O-UI` (stage-ui · 前端核心状态编排) | `O-AUDIO` (audio-web · 浏览器音频适配) | 调用 (`call`) | 管理WebUI语音测试拾音与播放 |
| `O-API` (alia-server · 本机实例与后端 API 服务) | `O-WEBUI` (apps/webui · 项目唯一内置 WebUI 面板) | 持有 / 归属 (`owns`) | 托管唯一内置WebUI控制台面板 |
| `O-API` (alia-server · 本机实例与后端 API 服务) | `O-AUTH` (alia-auth-service · 身份认证与授权服务) | 调用 (`call`) | 校验本机实例与通道访问授权 |
| `O-API` (alia-server · 本机实例与后端 API 服务) | `O-AGENT` (alia-core-agent · 通用交互运行时) | 持有 / 归属 (`owns`) | 持有并调度单实例通用Agent运行时 |
| `O-AGENT` (alia-core-agent · 通用交互运行时) | `O-CHAR` (alia-core-character · 角色与继承契约) | 包依赖 (`dependency`) | 读取角色设定与模块配置继承规则 |
| `O-AGENT` (alia-core-agent · 通用交互运行时) | `O-PROVIDER` (alia-provider-inference · 推理与任务路由) | 调用 (`call`) | 按任务分工发起流式推理与工具轮次 |
| `O-AGENT` (alia-core-agent · 通用交互运行时) | `O-STORAGE` (alia-storage · 本地持久状态存储) | 调用 (`call`) | 读写本地持久会话、记忆、笔记与Recipe |
| `O-AGENT` (alia-core-agent · 通用交互运行时) | `O-PIPE` (alia-audio-pipeline · 语音处理管线) | 调用 (`call`) | 协调语音输入转写、流控标记与TTS输出 |
| `O-AGENT` (alia-core-agent · 通用交互运行时) | `O-VISION` (alia-vision-pipeline · 视觉感知管线) | 调用 (`call`) | 协调附图读取与视觉去重 |
| `O-API` (alia-server · 本机实例与后端 API 服务) | `O-PLUGIN` (alia-extension-host · 插件扩展宿主) | 持有 / 归属 (`owns`) | 装配ExtensionHost插件沙箱宿主 |
| `O-PLUGIN` (alia-extension-host · 插件扩展宿主) | `O-KIT-PROTO` (alia-plugin-protocol · 插件与 Kit 协议) | 契约 / 配置 (`contract`) | 遵循KitContract与插件权限契约 |
| `O-PLUGIN` (alia-extension-host · 插件扩展宿主) | `O-AGENT` (alia-core-agent · 通用交互运行时) | 调用 (`call`) | 动态注册/撤回插件Provider、Tool与Context |
| `O-API` (alia-server · 本机实例与后端 API 服务) | `O-ROUTER` (alia-channel-gateway · 模块与接入渠道网关) | 持有 / 归属 (`owns`) | 挂载模块与外部接入渠道WebSocket网关 |
| `O-ROUTER` (alia-channel-gateway · 模块与接入渠道网关) | `O-KIT-PROTO` (alia-plugin-protocol · 插件与 Kit 协议) | 契约 / 配置 (`contract`) | 遵循统一模块通道与事件路由协议 |
| `O-AGENT` (alia-core-agent · 通用交互运行时) | `O-STREAM` (alia-stream-kit · 异步生命周期底座) | 包依赖 (`dependency`) | 基于Scope与Task管理异步生命周期 |
| `O-AGENT` (alia-core-agent · 通用交互运行时) | `O-TRACE` (alia-trace-kit · 全链路执行观测底座) | 事件 (`event`) | 写入Agent与Voice全链路TraceSpan |
| `O-TRACE` (alia-trace-kit · 全链路执行观测底座) | `O-TRACEUI` (ui-trace / ui-timeline / timeline-canvas) | 数据流 (`data`) | 推送实时诊断流与导出JSONL录制 |

---

### 专题 02：应用宿主与启动装配（alia-server 本机宿主 + apps/webui 面板初始化） (`platform-hosts`)

> **专题定位**：保留原版双端启动装配核心：Rust 端 alia-server 装配应用配置、模块通道网关、内置 HTTP/SSE 服务、原生语音/视觉管线、Trace Hub 与 ExtensionHost 插件宿主；前端唯一内置的 apps/webui 面板按 StartupResourcesStore 依次初始化认证、角色卡、会话、核心服务与音频路由。

- **配套矢量图纸**：[职责组织图 (`02-platform-hosts-organization.svg`)](./diagrams/02-platform-hosts-organization.svg) ｜ [调用与数据流图 (`02-platform-hosts-relations.svg`)](./diagrams/02-platform-hosts-relations.svg) ｜ [Graphviz 组织图源](./diagrams/02-platform-hosts-organization.dot) ｜ [Graphviz 关系图源](./diagrams/02-platform-hosts-relations.dot)
- **直接涉及源码范围**：`crates/alia-server/src/bootstrap.rs`, `apps/webui/src/App.vue`, `packages/stage-ui/src/stores/startup-resources.ts`

#### 读图与工程边界守则
- 已从原版启动链路中移除 Electron 桌面多窗口（Widget/主窗口/字幕窗口）与 2D/3D 形象模型预加载步骤。
- 本机实例启动后直接通过内置 HTTP/SSE 服务托管 apps/webui 控制台面板，并挂载 ExtensionHost 与模块通道服务供外部插件扩展。

#### 节点与状态归属登记表（共 19 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `H01` | WebUI 面板宿主 | **WebUI 面板根入口** | `App.vue` | `apps/webui/src/App.vue:1` | 装配认证、角色、会话、模型、音频和服务。不是只包含界面模板。 | Vue app scope | WebUI 面板前端 |
| `H02` | 共享启动状态 | **启动资源状态表** | `useStartupResourcesStore` | `packages/stage-ui/src/stores/startup-resources.ts:14` | 维护 queued/loading/ready/failed/skipped；用于启动覆盖层和错误呈现。 | resources[] | WebUI 面板前端 |
| `H03` | WebUI 启动步骤 | **认证初始化** | `authStore.initialize` | `apps/webui/src/App.vue:144` | 先恢复用户认证状态；注册认证后的产品初始化逻辑。 | 本图未单独列出；按相连对象与源码确认。 | WebUI 面板前端 |
| `H05` | WebUI 启动步骤 | **角色卡初始化** | `startup.run(card)` | `apps/webui/src/App.vue:146` | 初始化当前角色卡并接入认证后设置。 | 本图未单独列出；按相连对象与源码确认。 | WebUI 面板前端 |
| `H06` | WebUI 启动步骤 | **会话初始化** | `chatStore.initialize(syncedPinia)` | `apps/webui/src/App.vue:152` | 初始化聊天编排及跨窗口同步集成。 | 本图未单独列出；按相连对象与源码确认。 | WebUI 面板前端 |
| `H07` | WebUI 启动步骤 | **服务初始化** | `startup.run(services)` | `apps/webui/src/App.vue:153` | 初始化角色编排器和上下文桥等应用服务。 | 本图未单独列出；按相连对象与源码确认。 | WebUI 面板前端 |
| `H09` | WebUI 启动步骤 | **音频设备 / 路由** | `settingsAudioDeviceStore.initialize / router.isReady` | `apps/webui/src/App.vue:160` | 音频设备初始化；路由独立等待；模型显示是否完成另有状态。 | 本图未单独列出；按相连对象与源码确认。 | WebUI 面板前端 |
| `H10` | WebUI 生命周期 | **卸载清理** | `onUnmounted` | `apps/webui/src/App.vue:182` | dispose chat、character orchestrator、context bridge，停止作用域业务。 | 本图未单独列出；按相连对象与源码确认。 | WebUI 面板前端 |
| `H11` | Rust 服务端主进程 (alia-server) | **Rust 服务端主进程 (alia-server)** | `injeca container` | `crates/alia-server/src/bootstrap.rs:280` | 拥有原生窗口、服务和共享外部资源的装配。 | 主进程 service container | Rust 核心实现 |
| `H12` | Rust 服务端主进程 (alia-server) | **应用配置 / i18n** | `configs:app / libs:i18n` | `crates/alia-server/src/bootstrap.rs:283` | 持久应用配置供国际化、更新、窗口模式等服务读取。 | globalAppConfig | Rust 核心实现 |
| `H13` | Rust 宿主平台服务 | **模块通道服务** | `modules:channel-server` | `crates/alia-server/src/bootstrap.rs:310` | 嵌入 server-runtime，并管理监听配置和生命周期。 | Server 实例 | Rust 核心实现 |
| `H14` | Rust 宿主平台服务 | **内置 HTTP 服务** | `modules:alia-http-server` | `crates/alia-server/src/bootstrap.rs:315` | 为本机产品功能提供 HTTP 服务管理。与模块 WebSocket 通道分开。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `H15` | Rust 宿主平台服务 | **原生语音 / 视觉** | `appleSpeechTranscription / appleVision` | `crates/alia-server/src/bootstrap.rs:323` | 接入平台特有识别能力；由窗口服务依赖。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `H16` | Rust 宿主平台服务 | **MCP 服务管理** | `modules:mcp-manager` | `crates/alia-server/src/bootstrap.rs:331` | 维护 MCP 接入所需的主进程能力。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `H17` | Rust 宿主平台服务 | **Trace Hub / 录制** | `ioTraceHub / ioTraceRecording` | `crates/alia-server/src/bootstrap.rs:335` | Hub 聚合记录，录制服务写入记录并在 stop 时关闭订阅。 | trace collector / recording service | Rust 核心实现 |
| `H19` | Rust 扩展宿主装配 | **扩展宿主核心** | `new ExtensionHost` | `crates/alia-server/src/bootstrap.rs:385` | 创建 SDK 宿主，指定 runtime=rust-host、ALIA 版本。 | ExtensionHost | Rust 核心实现 |
| `H20` | Rust 扩展宿主装配 | **内置 Kit 注册** | `createBuiltInExtensionKits` | `crates/alia-server/src/bootstrap.rs:386` | 把 Widget/Widget/Tool 能力接到宿主。stop 时撤销注册。 | registration leases | Rust 核心实现 |
| `H21` | Rust 扩展宿主装配 | **扩展应用服务** | `setupExtensionHost` | `crates/alia-server/src/bootstrap.rs:394` | 装配清单目录、配置、启停、资源服务及管理 RPC。stop 先于依赖的 Kit。 | ExtensionHostService | Rust 核心实现 |
| `H24` | Rust 宿主生命周期 | **正式启动** | `injeca.start` | `crates/alia-server/src/bootstrap.rs:566` | 按照注册依赖构建服务；不能把声明顺序理解成所有功能严格顺序运行。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 19 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `H01` (WebUI 面板根入口) | `H02` (启动资源状态表) | 调用 (`call`) | 启动 / 记录状态 |
| `H01` (WebUI 面板根入口) | `H03` (认证初始化) | 调用 (`call`) | 初始化先后 |
| `H05` (角色卡初始化) | `H06` (会话初始化) | 调用 (`call`) | 初始化先后 |
| `H06` (会话初始化) | `H07` (服务初始化) | 调用 (`call`) | 初始化先后 |
| `H01` (WebUI 面板根入口) | `H10` (卸载清理) | 生命周期 (`lifecycle`) | scope 销毁 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H12` (应用配置 / i18n) | 持有 / 归属 (`owns`) | 注册 / 装配 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H13` (模块通道服务) | 持有 / 归属 (`owns`) | 注册 / 装配 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H14` (内置 HTTP 服务) | 持有 / 归属 (`owns`) | 注册 / 装配 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H15` (原生语音 / 视觉) | 持有 / 归属 (`owns`) | 注册 / 装配 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H16` (MCP 服务管理) | 持有 / 归属 (`owns`) | 注册 / 装配 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H17` (Trace Hub / 录制) | 持有 / 归属 (`owns`) | 注册 / 装配 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H19` (扩展宿主核心) | 持有 / 归属 (`owns`) | 注册 / 装配 |
| `H19` (扩展宿主核心) | `H20` (内置 Kit 注册) | 调用 (`call`) | 宿主供注册 |
| `H20` (内置 Kit 注册) | `H21` (扩展应用服务) | 调用 (`call`) | Kit 已就绪 |
| `H13` (模块通道服务) | `H21` (扩展应用服务) | 调用 (`call`) | 通道依赖 |
| `H11` (Rust 服务端主进程 (alia-server)) | `H24` (正式启动) | 生命周期 (`lifecycle`) | 触发构建 |
| `H21` (扩展应用服务) | `H20` (内置 Kit 注册) | 生命周期 (`lifecycle`) | 先停扩展，再撤 Kit |
| `H03` (认证初始化) | `H05` (角色卡初始化) | 调用 (`call`) | 初始化先后 |
| `H07` (服务初始化) | `H09` (音频设备 / 路由) | 调用 (`call`) | 初始化先后 |

---

### 专题 03：角色定义、设定文本与模块配置继承解析 (`character`)

> **专题定位**：保留原版角色定义与模块配置继承核心能力：AliaCard 持有角色设定文本（System Prompt）与角色模块覆盖（chat / consciousness / speech / vision），由 getModules(characterId) 实现“角色显式覆盖优先、未配置项自动回退全局默认 Store”的继承解析。

- **配套矢量图纸**：[职责组织图 (`03-character-organization.svg`)](./diagrams/03-character-organization.svg) ｜ [调用与数据流图 (`03-character-relations.svg`)](./diagrams/03-character-relations.svg) ｜ [Graphviz 组织图源](./diagrams/03-character-organization.dot) ｜ [Graphviz 关系图源](./diagrams/03-character-relations.dot)
- **直接涉及源码范围**：`crates/alia-core-character/src/inheritance.rs`, `crates/alia-core-character/src/card.rs`, `crates/alia-core-character/src/avatar-model.rs`, `packages/stage-ui/src/services/avatar-model.ts`, `packages/stage-ui/src/stores/display-models.ts`

#### 读图与工程边界守则
- 已从原版专题 03 中移除与 Live2D/VRM 模型文件读取及 Live2DControlPolicy 相关的 9 个可扩展形象节点，仅保留纯粹的角色身份与模块继承核心。
- 角色设定与记忆均持久保留在后端，当在 WebUI 切换底层推理模型时，角色的身份设定与模块继承规则保持不变。

#### 节点与状态归属登记表（共 9 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `character_store` | 角色配置 | **AliaCard Store** | `useAliaCardStore` | `crates/alia-core-character/src/inheritance.rs:191` | 角色增删、选择、配置解析、同步的业务入口 | cards / activeCardId / ready | Rust 核心实现 |
| `character_card` | 角色配置 | **AliaCard** | `AliaCard` | `crates/alia-core-character/src/card.rs:99` | Character Card 加 ALIA 扩展 | extensions.alia | Rust 核心实现 |
| `character_persona` | 角色配置 | **设定文本** | `resolveSystemPrompt` | `crates/alia-core-character/src/inheritance.rs:140` | 拼接 systemPrompt、description、personality、scenario | 文本配置 | Rust 核心实现 |
| `character_extension` | 角色配置 | **AliaExtension** | `AliaExtension` | `crates/alia-core-character/src/card.rs:13` | 保存 contextWindow、wakeWords、avatarModels、modules、agents | 随角色卡保存 | Rust 核心实现 |
| `character_modules` | 角色配置 | **角色模块覆盖** | `AliaExtension.modules` | `crates/alia-core-character/src/card.rs:28` | 保存 consciousness / vision / speech 选择 | provider / model / voice_id | Rust 核心实现 |
| `character_defaults` | 继承解析 | **全局默认 Store** | `readModuleDefaults` | `crates/alia-core-character/src/inheritance.rs:265` | 从功能 Store 读取默认选择 | Consciousness / Vision / Speech | Rust 核心实现 |
| `character_resolve` | 继承解析 | **getModules(characterId)** | `getModules` | `crates/alia-core-character/src/inheritance.rs:1312` | 按会话角色解析有效配置 | 不改 activeCardId | Rust 核心实现 |
| `character_voiceinherit` | 继承解析 | **resolveSpeechModule** | `resolveSpeechModule` | `crates/alia-core-character/src/inheritance.rs:1344` | 同 provider 且同 model 才继承默认 voice | 角色显式 voice 优先 | Rust 核心实现 |
| `character_prompt` | 继承解析 | **getSystemPrompt** | `getSystemPrompt` | `crates/alia-core-character/src/inheritance.rs:1372` | 读取指定角色的最终身份提示词 | 激活角色可含 ACT 控制说明 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 10 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `character_store` (AliaCard Store) | `character_card` (AliaCard) | 持有 / 归属 (`owns`) | 持有 |
| `character_card` (AliaCard) | `character_persona` (设定文本) | 持有 / 归属 (`owns`) | 包含设定 |
| `character_card` (AliaCard) | `character_extension` (AliaExtension) | 持有 / 归属 (`owns`) | 包含扩展 |
| `character_extension` (AliaExtension) | `character_modules` (角色模块覆盖) | 持有 / 归属 (`owns`) | 包含模块配置 |
| `character_store` (AliaCard Store) | `character_resolve` (getModules(characterId)) | 调用 (`call`) | 提供解析 |
| `character_modules` (角色模块覆盖) | `character_resolve` (getModules(characterId)) | 契约 / 配置 (`contract`) | 显式覆盖 |
| `character_defaults` (全局默认 Store) | `character_resolve` (getModules(characterId)) | 契约 / 配置 (`contract`) | 回退默认 |
| `character_resolve` (getModules(characterId)) | `character_voiceinherit` (resolveSpeechModule) | 调用 (`call`) | 解析 speech |
| `character_store` (AliaCard Store) | `character_prompt` (getSystemPrompt) | 调用 (`call`) | 提供提示词 |
| `character_persona` (设定文本) | `character_prompt` (getSystemPrompt) | 数据流 (`data`) | 设定内容 |

---

### 专题 04：会话宿主：宿主如何装配通用 Agent (`runtime_host`)

> **专题定位**：useChatStore 是宿主装配点。角色、会话、记忆、工具和媒体适配在此合成四个核心 ports，core-agent 不拥有 Rust SQLx (SQLite + sqlite-vec)、Vue 组件或具体 Provider 配置。

- **配套矢量图纸**：[职责组织图 (`04-runtime-host-organization.svg`)](./diagrams/04-runtime-host-organization.svg) ｜ [调用与数据流图 (`04-runtime-host-relations.svg`)](./diagrams/04-runtime-host-relations.svg) ｜ [Graphviz 组织图源](./diagrams/04-runtime-host-organization.dot) ｜ [Graphviz 关系图源](./diagrams/04-runtime-host-relations.dot)
- **直接涉及源码范围**：`packages/stage-ui/src/stores/chat.ts`, `crates/alia-core-character/src/inheritance.rs`, `packages/stage-ui/src/stores/modules/consciousness.ts`, `packages/stage-ui/src/stores/memory.ts`, `packages/stage-ui/src/stores/recipes.ts`, `packages/stage-ui/src/stores/ai/chat-llm/tools.ts`, `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs`, `crates/alia-core-agent/src/contracts/context-port.rs`, `crates/alia-core-agent/src/contracts/stream-port.rs`, `crates/alia-core-agent/src/contracts/llm-port.rs`, `packages/stage-ui/src/stores/chat/session-store.ts`, `packages/stage-ui/src/stores/chat/context-store.ts`, `packages/stage-ui/src/stores/chat/stream-store.ts`, `packages/stage-ui/src/stores/ai/chat-llm/llm.ts`

#### 读图与工程边界守则
- 角色身份由 session 的 character member 决定；选择界面上的另一张卡不会把排队请求改为另一个角色。
- startSend 的 accepted 表示用户消息已持久化；done 表示生成结束。二者是不同承诺。
- resolveStep 每次模型请求读取同一角色的当前配置，因此修改该角色和切换所选角色产生不同结果。
- 当前 runtime 实际使用 ChatOrchestratorSessionPort 的 commit 操作；contracts/session-port.ts 中的较旧 AgentSessionPort 不能代替它画成现行持久化边界。

#### 节点与状态归属登记表（共 24 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `runtime_host_store` | Stage 装配 | **useChatStore · 产品会话装配** | `export const useChatStore` | `packages/stage-ui/src/stores/chat.ts:261` | 组合产品 Store、运行时及宿主适配器。 | 局部requests、trace spans与UI镜像 | WebUI 面板前端 |
| `runtime_host_init` | Stage 装配 | **initialize / dispose · leader 生命周期** | `async function initialize(` | `packages/stage-ui/src/stores/chat.ts:350` | 注册同步运行时和leader订阅；角色任务与流的执行职责随leader清理。 | leader监听与作用域 | WebUI 面板前端 |
| `runtime_host_send` | Stage 入口 | **startSend · 请求去重** | `function startSend(` | `packages/stage-ui/src/stores/chat.ts:1378` | 以sessionId/messageId为key复用请求；等待Recipe与memory首次读取后提交。 | requests Map；每个请求有AbortController | WebUI 面板前端 |
| `runtime_host_prepare` | Stage 入口 | **prepareSend · 会话请求准备** | `async function prepareSend(` | `packages/stage-ui/src/stores/chat.ts:1274` | 加载session、取所属角色、Provider、附件、工具、场景/后台标记。 | 本次提交的选项快照 | WebUI 面板前端 |
| `runtime_host_persona` | 身份与知识 | **personaOf · session→character** | `function personaOf(` | `packages/stage-ui/src/stores/chat.ts:289` | 从会话的character members解析角色，缺失时使用fallback。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_host_prompt` | 身份与知识 | **sessionSystemPrompt · 身份+记忆索引** | `function sessionSystemPrompt(` | `packages/stage-ui/src/stores/chat.ts:299` | 拼合角色提示词、记忆索引、卡片规则；任务session加入Recipe步骤。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_host_card` | 身份与知识 | **useAliaCardStore · 角色与有效配置** | `export let useAliaCardStore` | `crates/alia-core-character/src/inheritance.rs:191` | 提供角色提示词和getModules继承解析。 | 角色卡及角色覆盖配置 | Rust 核心实现 |
| `runtime_host_conscious` | 身份与知识 | **useConsciousnessStore · 聊天模型选择** | `export const useConsciousnessStore` | `packages/stage-ui/src/stores/modules/consciousness.ts:11` | 提供聊天Provider实例、模型和采样配置。 | 全局默认聊天配置 | WebUI 面板前端 |
| `runtime_host_memory` | 身份与知识 | **useMemoryStore · 长期记忆入口** | `export const useMemoryStore` | `packages/stage-ui/src/stores/memory.ts:46` | 为指定persona提供索引、条目读取与修改。 | 设备记忆镜像；持久化由repo负责 | WebUI 面板前端 |
| `runtime_host_recipes` | 身份与知识 | **useRecipesStore · 可用技能** | `export const useRecipesStore` | `packages/stage-ui/src/stores/recipes.ts:54` | 提供enabled且approved的Recipe，供关键词、工具与后台任务使用。 | Recipe镜像与proposal开关 | WebUI 面板前端 |
| `runtime_host_tools` | 能力接入 | **useLlmToolsStore · 工具注册表** | `export const useLlmToolsStore` | `packages/stage-ui/src/stores/ai/chat-llm/tools.ts:48` | 持有可序列化定义与本地执行函数；解析模型可用工具。 | definitions ref与executors Map | WebUI 面板前端 |
| `runtime_host_vision` | 能力接入 | **chatVision / 图像适配** | `const visionSelection = chatVision.selectionOf` | `packages/stage-ui/src/stores/chat.ts:697` | 按首步模型能力决定附图是否改为描述；工具图片可另行读取。 | 失败读取缓存；描述写回session | WebUI 面板前端 |
| `runtime_host_audio` | 能力接入 | **prepareTextOnlyAudioContext · 音频请求副本** | `const prepareTextOnlyAudioContext = async` | `packages/stage-ui/src/stores/chat.ts:760` | 不支持音频的目标模型接收ASR文本；保留已存录音和可复用转写。 | liveTranscripts与每条音频转写缓存 | WebUI 面板前端 |
| `runtime_host_resolve` | 能力接入 | **resolveStep · 每步解析有效配置** | `const resolveStep = async () =>` | `packages/stage-ui/src/stores/chat.ts:1322` | 为每一次模型调用重新读取同一character的模型、Provider与系统提示词。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_host_runtime` | Core 边界 | **createChatOrchestratorRuntime** | `const runtime = createChatOrchestratorRuntime` | `packages/stage-ui/src/stores/chat.ts:1040` | 把四个port及上下文、生命周期回调注入可复用调度器。 | 运行时对象由该Store持有 | WebUI 面板前端 |
| `runtime_host_sessionport` | Core 边界 | **ChatOrchestratorSessionPort · 持久化提交接口** | `pub trait ChatOrchestratorSessionPort` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:223` | commitUserMessage/commitAssistantMessage 与只读history/generation。 | 宿主承诺落盘后resolve | Rust 核心实现 |
| `runtime_host_contextport` | Core 边界 | **AgentContextPort · 上下文接口** | `pub trait AgentContextPort` | `crates/alia-core-agent/src/contracts/context-port.rs:3` | ingest/snapshot/reset；runtime依赖其中ingest和snapshot。 | registry由宿主facade持有 | Rust 核心实现 |
| `runtime_host_streamport` | Core 边界 | **AgentForegroundStreamPort · 可见流接口** | `pub trait AgentForegroundStreamPort` | `crates/alia-core-agent/src/contracts/stream-port.rs:3` | patch/reset只投影前台session，不代替完整turn状态。 | 前台UI镜像 | Rust 核心实现 |
| `runtime_host_llmport` | Core 边界 | **AgentLLMPort · 模型流接口** | `pub trait AgentLLMPort` | `crates/alia-core-agent/src/contracts/llm-port.rs:7` | stream(model, provider, Conversation, options)返回完成Promise。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_host_sessions` | 宿主实现 | **useChatSessionStore · 会话owner** | `export const useChatSessionStore` | `packages/stage-ui/src/stores/chat/session-store.ts:69` | 提供history读取、提交、generation、hidden/task/session meta。 | sessionMessages/sessionMetas/sessionGenerations | WebUI 面板前端 |
| `runtime_host_contexts` | 宿主实现 | **useChatContextStore · 上下文facade** | `export const useChatContextStore` | `packages/stage-ui/src/stores/chat/context-store.ts:27` | 将事件送入核心registry并维护只读镜像。 | activeContextsMirror/contextHistoryMirror | WebUI 面板前端 |
| `runtime_host_streams` | 宿主实现 | **useChatStreamStore · 流显示** | `export const useChatStreamStore` | `packages/stage-ui/src/stores/chat/stream-store.ts:10` | 维护本地activeTurns及remoteMessages，按所选session显示。 | 每个窗口的流状态 | WebUI 面板前端 |
| `runtime_host_adapter` | 宿主实现 | **streamWithStageAdapters · 媒体/追踪适配** | `async function streamWithStageAdapters(` | `packages/stage-ui/src/stores/chat.ts:661` | 展开附件bytes、适配图片音频、构造工具/请求追踪，再调用LLM Store。 | 本轮trace和toolSpans | WebUI 面板前端 |
| `runtime_host_llmstore` | 宿主实现 | **useLLM.stream · 工具与兼容适配** | `export const useLLM` | `packages/stage-ui/src/stores/ai/chat-llm/llm.ts:28` | 解析完整工具表，缓存model兼容性，交给core streamFrom。 | toolsCompatibility/contentArrayCompatibility | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 27 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `runtime_host_store` (useChatStore · 产品会话装配) | `runtime_host_init` (initialize / dispose · leader 生命周期) | 持有 / 归属 (`owns`) | 管理作用域 |
| `runtime_host_store` (useChatStore · 产品会话装配) | `runtime_host_send` (startSend · 请求去重) | 持有 / 归属 (`owns`) | 暴露入口 |
| `runtime_host_send` (startSend · 请求去重) | `runtime_host_prepare` (prepareSend · 会话请求准备) | 调用 (`call`) | 准备 |
| `runtime_host_send` (startSend · 请求去重) | `runtime_host_memory` (useMemoryStore · 长期记忆入口) | 调用 (`call`) | ready() |
| `runtime_host_send` (startSend · 请求去重) | `runtime_host_recipes` (useRecipesStore · 可用技能) | 调用 (`call`) | ready() |
| `runtime_host_prepare` (prepareSend · 会话请求准备) | `runtime_host_persona` (personaOf · session→character) | 调用 (`call`) | 确定请求角色 |
| `runtime_host_persona` (personaOf · session→character) | `runtime_host_sessions` (useChatSessionStore · 会话owner) | 调用 (`call`) | characterMembersOf |
| `runtime_host_prepare` (prepareSend · 会话请求准备) | `runtime_host_card` (useAliaCardStore · 角色与有效配置) | 调用 (`call`) | getModules(characterId) |
| `runtime_host_prepare` (prepareSend · 会话请求准备) | `runtime_host_conscious` (useConsciousnessStore · 聊天模型选择) | 调用 (`call`) | 解析Provider实例 |
| `runtime_host_prepare` (prepareSend · 会话请求准备) | `runtime_host_resolve` (resolveStep · 每步解析有效配置) | 持有 / 归属 (`owns`) | 创建每步回调 |
| `runtime_host_prompt` (sessionSystemPrompt · 身份+记忆索引) | `runtime_host_card` (useAliaCardStore · 角色与有效配置) | 调用 (`call`) | getSystemPrompt |
| `runtime_host_prompt` (sessionSystemPrompt · 身份+记忆索引) | `runtime_host_memory` (useMemoryStore · 长期记忆入口) | 调用 (`call`) | indexFor(persona) |
| `runtime_host_prompt` (sessionSystemPrompt · 身份+记忆索引) | `runtime_host_recipes` (useRecipesStore · 可用技能) | 调用 (`call`) | 任务步骤 |
| `runtime_host_prepare` (prepareSend · 会话请求准备) | `runtime_host_tools` (useLlmToolsStore · 工具注册表) | 调用 (`call`) | 工具引用 |
| `runtime_host_send` (startSend · 请求去重) | `runtime_host_runtime` (createChatOrchestratorRuntime) | 调用 (`call`) | submit(text, options, sessionId) |
| `runtime_host_runtime` (createChatOrchestratorRuntime) | `runtime_host_sessionport` (ChatOrchestratorSessionPort · 持久化提交接口) | 调用 (`call`) | 依赖 |
| `runtime_host_runtime` (createChatOrchestratorRuntime) | `runtime_host_contextport` (AgentContextPort · 上下文接口) | 调用 (`call`) | 依赖 |
| `runtime_host_runtime` (createChatOrchestratorRuntime) | `runtime_host_streamport` (AgentForegroundStreamPort · 可见流接口) | 调用 (`call`) | 依赖 |
| `runtime_host_runtime` (createChatOrchestratorRuntime) | `runtime_host_llmport` (AgentLLMPort · 模型流接口) | 调用 (`call`) | 依赖 |
| `runtime_host_sessionport` (ChatOrchestratorSessionPort · 持久化提交接口) | `runtime_host_sessions` (useChatSessionStore · 会话owner) | 调用 (`call`) | 宿主实现 |
| `runtime_host_contextport` (AgentContextPort · 上下文接口) | `runtime_host_contexts` (useChatContextStore · 上下文facade) | 调用 (`call`) | 宿主实现 |
| `runtime_host_streamport` (AgentForegroundStreamPort · 可见流接口) | `runtime_host_streams` (useChatStreamStore · 流显示) | 调用 (`call`) | 宿主实现 |
| `runtime_host_llmport` (AgentLLMPort · 模型流接口) | `runtime_host_adapter` (streamWithStageAdapters · 媒体/追踪适配) | 调用 (`call`) | 宿主实现 |
| `runtime_host_adapter` (streamWithStageAdapters · 媒体/追踪适配) | `runtime_host_vision` (chatVision / 图像适配) | 调用 (`call`) | 处理图片 |
| `runtime_host_adapter` (streamWithStageAdapters · 媒体/追踪适配) | `runtime_host_audio` (prepareTextOnlyAudioContext · 音频请求副本) | 调用 (`call`) | 按请求处理音频 |
| `runtime_host_adapter` (streamWithStageAdapters · 媒体/追踪适配) | `runtime_host_llmstore` (useLLM.stream · 工具与兼容适配) | 调用 (`call`) | stream |
| `runtime_host_runtime` (createChatOrchestratorRuntime) | `runtime_host_prompt` (sessionSystemPrompt · 身份+记忆索引) | 调用 (`call`) | 读取身份 |

---

### 专题 05：调度器内部：按会话排队、提交、中断与收尾 (`runtime_schedule`)

> **专题定位**：每个session有独立FIFO队列和active send。不同会话可以同时生成；同一会话的后续输入等待前一轮。generation是防止已重置会话被旧结果写回的版本屏障。

- **配套矢量图纸**：[职责组织图 (`05-runtime-schedule-organization.svg`)](./diagrams/05-runtime-schedule-organization.svg) ｜ [调用与数据流图 (`05-runtime-schedule-relations.svg`)](./diagrams/05-runtime-schedule-relations.svg) ｜ [Graphviz 组织图源](./diagrams/05-runtime-schedule-organization.dot) ｜ [Graphviz 关系图源](./diagrams/05-runtime-schedule-relations.dot)
- **直接涉及源码范围**：`crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs`, `crates/alia-core-agent/src/runtime/agent-hooks.rs`

#### 读图与工程边界守则
- activeSendSessionId是可见状态镜像，不能据此误画成全局只能执行一个请求；authoritative activeSends是Map。
- 取消针对(sessionId,turnId)。清空或删除session同时使generation失效，拒绝旧队列项。
- 正常完成才记录generatedTurn；中断已有可见内容时保留interrupted历史，不能伪装成completed生成记录。

#### 节点与状态归属登记表（共 24 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `runtime_schedule_runtime` | 运行时 | **ChatOrchestratorRuntime** | `pub fn createChatOrchestratorRuntime` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:523` | 调度、上下文组合、流处理和持久化提交。 | 下列Map与队列归此实例 | Rust 核心实现 |
| `runtime_schedule_submit` | 入口与队列 | **submit · 捕获所有权** | `pub fn submit(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1469` | 捕获目标session与generation，复制请求资源，返回accepted/done。 | QueuedSend | Rust 核心实现 |
| `runtime_schedule_queued` | 入口与队列 | **QueuedSend · 排队项** | `interface QueuedSend {` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:184` | 请求文本、options、sessionId、generation和Promise出口。 | 该队列项生命周期 | Rust 核心实现 |
| `runtime_schedule_queue` | 入口与队列 | **queueFor(sessionId)** | `pub fn queueFor(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1414` | 懒创建该session的串行工作队列；空闲后删除。 | 每session一条FIFO | Rust 核心实现 |
| `runtime_schedule_queues` | 入口与队列 | **sendQueues Map** | `let sendQueues = new Map` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:531` | sessionId→createQueue<QueuedSend>；允许跨session并行。 | runtime拥有 | Rust 核心实现 |
| `runtime_schedule_pending` | 入口与队列 | **pendingQueuedSends** | `let pendingQueuedSends:` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:544` | 提供等待数量和可序列化队列快照。 | 排队项列表 | Rust 核心实现 |
| `runtime_schedule_active` | 执行与取消 | **activeSends Map** | `let activeSends = new Map` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:526` | sessionId→AbortController、turnId、streaming message。 | 每个活跃session的执行状态 | Rust 核心实现 |
| `runtime_schedule_perform` | 执行与取消 | **performSend · 执行一轮** | `pub async fn performSend(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:697` | 组织提交、上下文、流事件、最终历史和清理。 | 本轮buildingMessage、slices、usage | Rust 核心实现 |
| `runtime_schedule_guard` | 执行与取消 | **generation + abort 屏障** | `let isStaleGeneration =` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:758` | 旧generation或已abort时停止继续写入/渲染。 | 只读取session当前generation | Rust 核心实现 |
| `runtime_schedule_commituser` | 提交边界 | **commitUserMessage** | `let receipt = await deps.session.commitUserMessage` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:877` | 用户输入先持久化；重复messageId返回existing，不再生成。 | 宿主落盘结果 | Rust 核心实现 |
| `runtime_schedule_accepted` | 提交边界 | **accepted.resolve · 持久化回执** | `accepted.resolve({ sessionId` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:878` | 只在用户消息已保存后告诉调用者accepted。 | 与生成完成分离的Promise | Rust 核心实现 |
| `runtime_schedule_compose` | 请求上下文 | **buildContext** | `pub fn buildContext(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:624` | 恢复结构化turn、时间、notice来源、回复引用及invoked skills。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_schedule_trim` | 请求上下文 | **trimConversationHistory** | `pub fn trimConversationHistory(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:646` | 按对话turn限制发送历史；超限保留ceil(limit*0.9)；不删存储历史。 | 使用historyCuts缓存边界 | Rust 核心实现 |
| `runtime_schedule_cuts` | 请求上下文 | **historyCuts Map** | `let historyCuts = new Map` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:534` | 保存每session的limit/generation/start锚点，减少重复扫描。 | runtime临时状态 | Rust 核心实现 |
| `runtime_schedule_contexts` | 请求上下文 | **ingestRuntimeContexts** | `pub fn ingestRuntimeContexts(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:608` | 在构建请求前获取宿主上下文Provider。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_schedule_model` | 生成与显示 | **deps.llm.stream** | `deps.llm.stream(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1108` | 把Conversation与选项交给宿主LLM适配器。 | 正在执行的transport | Rust 核心实现 |
| `runtime_schedule_parser` | 生成与显示 | **useLlmmarkerParser** | `let parser = useLlmmarkerParser` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:913` | 分离字面文本和控制标记，再送显示及语音hook。 | 跨chunk解析缓冲 | Rust 核心实现 |
| `runtime_schedule_update` | 生成与显示 | **updateStream** | `pub fn updateStream(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:589` | 更新该session的active message，仅为前台session patch UI。 | 本轮stream snapshot | Rust 核心实现 |
| `runtime_schedule_generated` | 提交边界 | **onGeneratedTurn** | `onGeneratedTurn: (turn)` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1121` | 保存协议层完成的AssistantTurn，包含所有settled rounds。 | 临时generatedTurn | Rust 核心实现 |
| `runtime_schedule_commitassistant` | 提交边界 | **commitAssistantMessage** | `await deps.session.commitAssistantMessage(sessionId, { ...finalAssistant` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1292` | 完整生成和hook收尾后提交助手消息。 | 宿主持久化完整记录 | Rust 核心实现 |
| `runtime_schedule_partial` | 提交边界 | **storeInterruptedAssistant** | `let storeInterruptedAssistant =` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:773` | 中断时只保留已可见的局部回复并标interrupted。 | 局部历史；无完整generationTranscript | Rust 核心实现 |
| `runtime_schedule_cancel` | 执行与取消 | **cancelTurn** | `pub fn cancelTurn(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1528` | abort匹配的活跃轮次，并取消messageId匹配的待执行项。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_schedule_cancelqueue` | 执行与取消 | **cancelPendingSends** | `pub fn cancelPendingSends(` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1501` | abort指定session或全部活跃请求，拒绝相应待执行项，并释放historyCuts。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_schedule_hooks` | 生成与显示 | **createChatHooks** | `pub fn createChatHooks` | `crates/alia-core-agent/src/runtime/agent-hooks.rs:6` | 生命周期、字面token、special token、完成事件订阅。 | hook数组；clearHooks释放 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 28 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `runtime_schedule_runtime` (ChatOrchestratorRuntime) | `runtime_schedule_queues` (sendQueues Map) | 持有 / 归属 (`owns`) | 持有 |
| `runtime_schedule_runtime` (ChatOrchestratorRuntime) | `runtime_schedule_active` (activeSends Map) | 持有 / 归属 (`owns`) | 持有 |
| `runtime_schedule_runtime` (ChatOrchestratorRuntime) | `runtime_schedule_cuts` (historyCuts Map) | 持有 / 归属 (`owns`) | 持有 |
| `runtime_schedule_runtime` (ChatOrchestratorRuntime) | `runtime_schedule_hooks` (createChatHooks) | 持有 / 归属 (`owns`) | 创建 |
| `runtime_schedule_submit` (submit · 捕获所有权) | `runtime_schedule_queued` (QueuedSend · 排队项) | 调用 (`call`) | 创建 |
| `runtime_schedule_queued` (QueuedSend · 排队项) | `runtime_schedule_pending` (pendingQueuedSends) | 调用 (`call`) | 登记 |
| `runtime_schedule_submit` (submit · 捕获所有权) | `runtime_schedule_queue` (queueFor(sessionId)) | 调用 (`call`) | 选择session队列 |
| `runtime_schedule_queue` (queueFor(sessionId)) | `runtime_schedule_queues` (sendQueues Map) | 调用 (`call`) | 索引 |
| `runtime_schedule_queue` (queueFor(sessionId)) | `runtime_schedule_perform` (performSend · 执行一轮) | 调用 (`call`) | FIFO执行 |
| `runtime_schedule_perform` (performSend · 执行一轮) | `runtime_schedule_active` (activeSends Map) | 调用 (`call`) | 登记本轮 |
| `runtime_schedule_perform` (performSend · 执行一轮) | `runtime_schedule_guard` (generation + abort 屏障) | 调用 (`call`) | 多处检查 |
| `runtime_schedule_perform` (performSend · 执行一轮) | `runtime_schedule_contexts` (ingestRuntimeContexts) | 调用 (`call`) | 取运行上下文 |
| `runtime_schedule_perform` (performSend · 执行一轮) | `runtime_schedule_commituser` (commitUserMessage) | 调用 (`call`) | 保存输入 |
| `runtime_schedule_commituser` (commitUserMessage) | `runtime_schedule_accepted` (accepted.resolve · 持久化回执) | 调用 (`call`) | 返回存储回执 |
| `runtime_schedule_perform` (performSend · 执行一轮) | `runtime_schedule_trim` (trimConversationHistory) | 调用 (`call`) | 选历史 |
| `runtime_schedule_trim` (trimConversationHistory) | `runtime_schedule_cuts` (historyCuts Map) | 调用 (`call`) | 读写锚点 |
| `runtime_schedule_trim` (trimConversationHistory) | `runtime_schedule_compose` (buildContext) | 数据流 (`data`) | 保留历史 |
| `runtime_schedule_compose` (buildContext) | `runtime_schedule_model` (deps.llm.stream) | 数据流 (`data`) | Conversation |
| `runtime_schedule_model` (deps.llm.stream) | `runtime_schedule_parser` (useLlmmarkerParser) | 数据流 (`data`) | StreamEvent文本 |
| `runtime_schedule_parser` (useLlmmarkerParser) | `runtime_schedule_hooks` (createChatHooks) | 调用 (`call`) | literal / special |
| `runtime_schedule_parser` (useLlmmarkerParser) | `runtime_schedule_update` (updateStream) | 调用 (`call`) | 追加内容 |
| `runtime_schedule_model` (deps.llm.stream) | `runtime_schedule_generated` (onGeneratedTurn) | 调用 (`call`) | settled turn |
| `runtime_schedule_generated` (onGeneratedTurn) | `runtime_schedule_commitassistant` (commitAssistantMessage) | 数据流 (`data`) | 正常提交 |
| `runtime_schedule_cancel` (cancelTurn) | `runtime_schedule_active` (activeSends Map) | 调用 (`call`) | abort匹配turn |
| `runtime_schedule_cancel` (cancelTurn) | `runtime_schedule_pending` (pendingQueuedSends) | 调用 (`call`) | 取消匹配messageId |
| `runtime_schedule_cancelqueue` (cancelPendingSends) | `runtime_schedule_active` (activeSends Map) | 调用 (`call`) | 同时abort活跃请求 |
| `runtime_schedule_cancelqueue` (cancelPendingSends) | `runtime_schedule_queues` (sendQueues Map) | 调用 (`call`) | 拒绝待执行项 |
| `runtime_schedule_guard` (generation + abort 屏障) | `runtime_schedule_partial` (storeInterruptedAssistant) | 调用 (`call`) | 有可见内容时保存 |

---

### 专题 06：领域数据：Conversation、Turn、Round、Tool 与上下文桶 (`runtime_domain`)

> **专题定位**：这是数据所有权图。Conversation持有Turn，只有AssistantTurn持有GenerationRound，工具调用及其结果只归属某一个round；原生continuation是协议与scope受限的恢复材料。

- **配套矢量图纸**：[职责组织图 (`06-runtime-domain-organization.svg`)](./diagrams/06-runtime-domain-organization.svg) ｜ [调用与数据流图 (`06-runtime-domain-relations.svg`)](./diagrams/06-runtime-domain-relations.svg) ｜ [Graphviz 组织图源](./diagrams/06-runtime-domain-organization.dot) ｜ [Graphviz 关系图源](./diagrams/06-runtime-domain-relations.dot)
- **直接涉及源码范围**：`crates/alia-core-agent/src/messages/types.rs`, `crates/alia-core-agent/src/runtime/context-registry.rs`, `crates/alia-core-agent/src/messages/preview.rs`

#### 读图与工程边界守则
- 一个用户交互回合不等于一个模型请求；工具轮次可以使同一个AssistantTurn包含多个GenerationRound。
- SystemTurn.authority明确区分system/developer/context，应用提供的数据不自动获得指令权威。
- runId对应真实调度执行；tool callId是Provider关联键；ALIA invocationId用于区分可能重复的Provider callId。
- ContextRegistry默认400条限制的是ingest history；append-self活跃桶没有在此代码中自动按400裁剪。

#### 节点与状态归属登记表（共 22 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `runtime_domain_conversation` | 会话结构 | **Conversation** | `pub trait Conversation {` | `crates/alia-core-agent/src/messages/types.rs:25` | 按顺序持有turns。 | turns: Turn[] | Rust 核心实现 |
| `runtime_domain_turn` | 会话结构 | **Turn union** | `pub enum Turn =` | `crates/alia-core-agent/src/messages/types.rs:30` | UserTurn / AssistantTurn / SystemTurn互斥联合。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_user` | 会话结构 | **UserTurn** | `pub trait UserTurn {` | `crates/alia-core-agent/src/messages/types.rs:33` | 用户内容、附件和结构化context。 | id + content | Rust 核心实现 |
| `runtime_domain_system` | 会话结构 | **SystemTurn** | `pub trait SystemTurn {` | `crates/alia-core-agent/src/messages/types.rs:40` | 带明确authority的指令或应用上下文。 | id + authority + content | Rust 核心实现 |
| `runtime_domain_assistant` | 生成结构 | **AssistantTurn** | `pub trait AssistantTurn {` | `crates/alia-core-agent/src/messages/types.rs:48` | 一次已完成助手执行；失败/取消不产生completed turn。 | id/runId/status/rounds | Rust 核心实现 |
| `runtime_domain_round` | 生成结构 | **GenerationRound** | `pub trait GenerationRound {` | `crates/alia-core-agent/src/messages/types.rs:62` | 一次模型调用及它请求的全部工具执行。 | content/toolInvocations/continuation/projectionIssues | Rust 核心实现 |
| `runtime_domain_model` | 生成结构 | **modelCall** | `modelCall?: { model:` | `crates/alia-core-agent/src/messages/types.rs:65` | 记录该round的model、finishReason和usage；导入历史可缺失。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_content` | 生成结构 | **RoundContent** | `pub enum RoundContent =` | `crates/alia-core-agent/src/messages/types.rs:59` | 文本/拒绝/context或工具invocationId引用，保留输出顺序。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_invocation` | 工具结构 | **ToolInvocation** | `pub trait ToolInvocation {` | `crates/alia-core-agent/src/messages/types.rs:75` | 工具名、参数、callId和执行结果有且只有一个owner。 | id/callId/name/arguments/execution | Rust 核心实现 |
| `runtime_domain_execution` | 工具结构 | **ToolExecution** | `pub enum ToolExecution` | `crates/alia-core-agent/src/messages/types.rs:85` | pending / succeeded / failed / cancelled；缺结果不能算成功。 | 状态及输出segments | Rust 核心实现 |
| `runtime_domain_continuation` | 协议恢复 | **ProviderContinuation** | `pub enum ProviderContinuation =` | `crates/alia-core-agent/src/messages/types.rs:154` | 保留Chat Message[]或Responses ItemParam[]；禁止跨scope盲重播。 | protocol + scope + native data | Rust 核心实现 |
| `runtime_domain_issues` | 协议恢复 | **projectionIssues** | `projectionIssues: string[]` | `crates/alia-core-agent/src/messages/types.rs:71` | 记录不能可移植投影的原生内容，不丢弃原始payload。 | 每round的问题数组 | Rust 核心实现 |
| `runtime_domain_input` | 内容段 | **InputSegment** | `pub enum InputSegment =` | `crates/alia-core-agent/src/messages/types.rs:112` | text/image/audio/file输入；文件来源互斥。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_context` | 内容段 | **ContextSegment** | `pub enum ContextSegment =` | `crates/alia-core-agent/src/messages/types.rs:91` | instruction/tagged text/domain event/state/history/summary/reference/runtime context。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_citation` | 内容段 | **Citation** | `pub trait Citation {` | `crates/alia-core-agent/src/messages/types.rs:126` | URL、title和文字偏移，独立于可朗读文字。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_registry` | 运行上下文 | **ContextRegistry** | `pub fn createContextRegistry` | `crates/alia-core-agent/src/runtime/context-registry.rs:99` | 拥有writer分桶和有界ingest history。 | currentActiveContexts + currentContextHistory | Rust 核心实现 |
| `runtime_domain_bucket` | 运行上下文 | **currentActiveContexts** | `let currentActiveContexts =` | `crates/alia-core-agent/src/runtime/context-registry.rs:103` | writer sourceKey→ContextMessage[]，与持久会话历史分开。 | Map；runtime内存 | Rust 核心实现 |
| `runtime_domain_ingest` | 运行上下文 | **ingest / writer语义** | `pub fn ingest(envelope:` | `crates/alia-core-agent/src/runtime/context-registry.rs:106` | replace-self替换自己的桶；append-self追加自己的桶。 | 复制输入后保存 | Rust 核心实现 |
| `runtime_domain_snapshot` | 运行上下文 | **snapshot** | `pub fn snapshot()` | `crates/alia-core-agent/src/runtime/context-registry.rs:149` | 返回克隆，消费者不能改内部桶。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_remove` | 运行上下文 | **removeWriter** | `removeWriter: sourceKey =>` | `crates/alia-core-agent/src/runtime/context-registry.rs:161` | 模块离开时清除该来源活跃桶。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_domain_history` | 运行上下文 | **currentContextHistory** | `let currentContextHistory:` | `crates/alia-core-agent/src/runtime/context-registry.rs:104` | 保留有界写入观察记录，默认400；不等于长期记忆。 | registry诊断记录 | Rust 核心实现 |
| `runtime_domain_preview` | 显示投影 | **renderConversationPreview** | `pub fn renderConversationPreview` | `crates/alia-core-agent/src/messages/preview.rs:24` | 为hook/调试生成纯文本投影；media变标签，不泄露native continuation。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 23 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `runtime_domain_conversation` (Conversation) | `runtime_domain_turn` (Turn union) | 持有 / 归属 (`owns`) | owns ordered[] |
| `runtime_domain_turn` (Turn union) | `runtime_domain_user` (UserTurn) | 调用 (`call`) | variant |
| `runtime_domain_turn` (Turn union) | `runtime_domain_system` (SystemTurn) | 调用 (`call`) | variant |
| `runtime_domain_turn` (Turn union) | `runtime_domain_assistant` (AssistantTurn) | 调用 (`call`) | variant |
| `runtime_domain_assistant` (AssistantTurn) | `runtime_domain_round` (GenerationRound) | 持有 / 归属 (`owns`) | owns rounds[] |
| `runtime_domain_round` (GenerationRound) | `runtime_domain_model` (modelCall) | 持有 / 归属 (`owns`) | metadata |
| `runtime_domain_round` (GenerationRound) | `runtime_domain_content` (RoundContent) | 持有 / 归属 (`owns`) | ordered content[] |
| `runtime_domain_round` (GenerationRound) | `runtime_domain_invocation` (ToolInvocation) | 持有 / 归属 (`owns`) | owns toolInvocations[] |
| `runtime_domain_invocation` (ToolInvocation) | `runtime_domain_execution` (ToolExecution) | 持有 / 归属 (`owns`) | one result owner |
| `runtime_domain_content` (RoundContent) | `runtime_domain_invocation` (ToolInvocation) | 调用 (`call`) | invocationId引用 |
| `runtime_domain_round` (GenerationRound) | `runtime_domain_continuation` (ProviderContinuation) | 持有 / 归属 (`owns`) | scope受限恢复数据 |
| `runtime_domain_round` (GenerationRound) | `runtime_domain_issues` (projectionIssues) | 持有 / 归属 (`owns`) | 投影问题 |
| `runtime_domain_user` (UserTurn) | `runtime_domain_input` (InputSegment) | 持有 / 归属 (`owns`) | 附件/文字 |
| `runtime_domain_system` (SystemTurn) | `runtime_domain_context` (ContextSegment) | 持有 / 归属 (`owns`) | 有authority的上下文 |
| `runtime_domain_content` (RoundContent) | `runtime_domain_citation` (Citation) | 持有 / 归属 (`owns`) | 文本来源偏移 |
| `runtime_domain_registry` (ContextRegistry) | `runtime_domain_bucket` (currentActiveContexts) | 持有 / 归属 (`owns`) | 拥有活跃桶 |
| `runtime_domain_registry` (ContextRegistry) | `runtime_domain_history` (currentContextHistory) | 持有 / 归属 (`owns`) | 拥有有界观察记录 |
| `runtime_domain_ingest` (ingest / writer语义) | `runtime_domain_bucket` (currentActiveContexts) | 调用 (`call`) | replace/append self |
| `runtime_domain_ingest` (ingest / writer语义) | `runtime_domain_history` (currentContextHistory) | 调用 (`call`) | 记录更新 |
| `runtime_domain_bucket` (currentActiveContexts) | `runtime_domain_snapshot` (snapshot) | 数据流 (`data`) | clone |
| `runtime_domain_remove` (removeWriter) | `runtime_domain_bucket` (currentActiveContexts) | 调用 (`call`) | 删除指定writer |
| `runtime_domain_snapshot` (snapshot) | `runtime_domain_context` (ContextSegment) | 数据流 (`data`) | 组装runtime-context |
| `runtime_domain_conversation` (Conversation) | `runtime_domain_preview` (renderConversationPreview) | 调用 (`call`) | 独立显示投影 |

---

### 专题 07：推理边界：定义、配置、实例、协议与工具轮次 (`runtime_providers`)

> **专题定位**：Provider定义与配置持久化分层；core只收到GenerationProvider。generation()先选协议，再把同一Conversation投影成Chat消息或Responses Items。

- **配套矢量图纸**：[职责组织图 (`07-runtime-providers-organization.svg`)](./diagrams/07-runtime-providers-organization.svg) ｜ [调用与数据流图 (`07-runtime-providers-relations.svg`)](./diagrams/07-runtime-providers-relations.svg) ｜ [Graphviz 组织图源](./diagrams/07-runtime-providers-organization.dot) ｜ [Graphviz 关系图源](./diagrams/07-runtime-providers-relations.dot)
- **直接涉及源码范围**：`crates/alia-provider-inference/src/types.rs`, `crates/alia-provider-inference/src/providers/registry.rs`, `packages/stage-ui/src/stores/providers/config.ts`, `packages/stage-ui/src/stores/providers/provider.ts`, `crates/alia-provider-inference/src/model-catalog.rs`, `crates/alia-core-agent/src/types/llm.rs`, `crates/alia-core-agent/src/runtime/llm-service.rs`, `crates/alia-core-agent/src/runtime/chat-completions.rs`, `crates/alia-core-agent/src/runtime/responses.rs`, `crates/alia-core-agent/src/messages/chat-completions.rs`, `crates/alia-core-agent/src/runtime/generation.rs`, `crates/alia-core-agent/src/runtime/request-context.rs`, `crates/alia-core-agent/src/runtime/request-switch.rs`, `packages/stage-ui/src/stores/ai/chat-llm/llm.ts`

#### 读图与工程边界守则
- 源码两种adapter均使用stopWhen:not(hasToolCall())，没有在adapter中设置固定轮次上限；README所写Responses最多十步与本提交源码不一致。
- resolveStep改变provider/endpoint/model/conversation scope或protocol时，用RequestSwitch携带partialTurn重新开流，已完成rounds仍属于同一AssistantTurn。
- 自动HTTP重试只发生于首个stream event之前，且408/429/5xx；已有输出或工具副作用后不自动整轮重放。
- 图片适配首步决定；音频prepareConversation逐请求决定。两者当前并未完全统一。

#### 节点与状态归属登记表（共 24 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `runtime_providers_definition` | Provider 定义 | **ProviderDefinition** | `pub trait ProviderDefinition<` | `crates/alia-provider-inference/src/types.rs:266` | 静态定义schema、factory、capabilities、extra与validators。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_registry` | Provider 定义 | **createProviderRegistry** | `pub fn createProviderRegistry` | `crates/alia-provider-inference/src/providers/registry.rs:30` | 按definition id注册、查找Provider定义。 | definitions Map | Rust 核心实现 |
| `runtime_providers_config` | Stage 配置与实例 | **useProviderConfigStore** | `export const useProviderConfigStore` | `packages/stage-ui/src/stores/providers/config.ts:50` | 保存provider实例id、definitionId、config和同步状态。 | 可序列化配置；凭证在配置边界 | WebUI 面板前端 |
| `runtime_providers_runtime` | Stage 配置与实例 | **useProviderStore** | `export const useProviderStore` | `packages/stage-ui/src/stores/providers/provider.ts:106` | 模型发现、验证和实例缓存；不把runtime对象塞入持久配置。 | providerRuntimeState与实例缓存 | WebUI 面板前端 |
| `runtime_providers_instance` | Stage 配置与实例 | **getProviderInstance** | `async function getProviderInstance<` | `packages/stage-ui/src/stores/providers/provider.ts:1022` | 用定义factory与当前config创建或复用实例。 | instance cache；配置变化dispose | WebUI 面板前端 |
| `runtime_providers_catalog` | Provider 定义 | **listModelCatalog** | `export pub async fn listModelCatalog` | `crates/alia-provider-inference/src/model-catalog.rs:26` | 规范化模型发现结果；含能力、默认model与availability。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_validate` | Stage 配置与实例 | **validateProviderConfig** | `async function validateProviderConfig(` | `packages/stage-ui/src/stores/providers/provider.ts:269` | 运行config/runtime验证器并记录状态。 | 验证状态/重验证循环 | WebUI 面板前端 |
| `runtime_providers_genprovider` | Core 请求契约 | **GenerationProvider** | `pub trait GenerationProvider` | `crates/alia-provider-inference/src/types.rs:28` | 协议中立的generation(model, options)能力。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_genrequest` | Core 请求契约 | **GenerationRequest** | `pub enum GenerationRequest` | `crates/alia-provider-inference/src/types.rs:23` | discriminated union：chat-completions或responses及配置。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_resolve` | Core 请求契约 | **resolveStep** | `resolveStep?:` | `crates/alia-core-agent/src/types/llm.rs:47` | 每次模型请求解析最新model/provider/systemPrompt/tools/headers。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_stream` | Core 执行 | **streamFrom** | `export pub async fn streamFrom(` | `crates/alia-core-agent/src/runtime/llm-service.rs:279` | 维护跨RequestSwitch的completedRounds与usage。 | 同一assistant turn的已完成rounds | Rust 核心实现 |
| `runtime_providers_once` | Core 执行 | **streamOnce** | `pub async fn streamOnce(` | `crates/alia-core-agent/src/runtime/llm-service.rs:75` | 解析目标能力、适配上下文、合并工具、选择协议adapter并等待消费者。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_media` | Core 请求契约 | **prepareConversation** | `if (options?.prepareConversation)` | `crates/alia-core-agent/src/runtime/llm-service.rs:87` | 适配当前目标模型的媒体请求副本，不改历史。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_chat` | 双协议适配 | **streamChatCompletions** | `pub fn streamChatCompletions(` | `crates/alia-core-agent/src/runtime/chat-completions.rs:18` | Conversation→Chat Message[]→alia-provider-inference stream_text。 | 本SDK流scope与steps | Rust 核心实现 |
| `runtime_providers_responses` | 双协议适配 | **streamResponses** | `pub fn streamResponses(` | `crates/alia-core-agent/src/runtime/responses.rs:175` | Conversation→native ItemParam[]；store:false；可选provider web_search。 | 本SDK流scope与steps | Rust 核心实现 |
| `runtime_providers_chatproject` | 双协议适配 | **conversationToChatMessages** | `pub fn conversationToChatMessages(` | `crates/alia-core-agent/src/messages/chat-completions.rs:168` | 协议投影；可重播同scope native continuation。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_respject` | 双协议适配 | **renderConversation** | `pub fn renderConversation(` | `crates/alia-core-agent/src/runtime/responses.rs:87` | 直接投影Responses Items，不先降为Chat格式。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_generation` | 轮次与恢复 | **createGeneration** | `pub fn createGeneration<` | `crates/alia-core-agent/src/runtime/generation.rs:49` | 以SDK input快照确定round边界，保存native与portable结果。 | starts/models数组 | Rust 核心实现 |
| `runtime_providers_step` | 轮次与恢复 | **prepareStep** | `pub fn prepareStep(` | `crates/alia-core-agent/src/runtime/generation.rs:61` | 保存每次模型调用开始的input长度及model。 | round起点offset | Rust 核心实现 |
| `runtime_providers_complete` | 轮次与恢复 | **complete** | `pub async fn complete(` | `crates/alia-core-agent/src/runtime/generation.rs:67` | await items和steps，生成AssistantTurn内所有GenerationRound。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_scope` | 轮次与恢复 | **createContinuationScope** | `pub fn createContinuationScope(` | `crates/alia-core-agent/src/runtime/request-context.rs:47` | 标识provider identity、endpoint、model、conversation；凭证和headers不属于scope。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_switch` | 轮次与恢复 | **RequestSwitch** | `pub struct RequestSwitch` | `crates/alia-core-agent/src/runtime/request-switch.rs:5` | 携带next配置与partialTurn结束旧SDK流，重新投影续跑。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_providers_retry` | 失败边界 | **streamWithTransientRetry** | `pub async fn streamWithTransientRetry(` | `crates/alia-core-agent/src/runtime/llm-service.rs:240` | 首事件前的瞬态HTTP失败最多三次重试；取消也取消等待。 | attempt与consumerNotified | Rust 核心实现 |
| `runtime_providers_compat` | 失败边界 | **tools / contentArray兼容缓存** | `const toolsCompatibility =` | `packages/stage-ui/src/stores/ai/chat-llm/llm.ts:29` | 按协议/endpoint/model记忆兼容失败；content数组可安全退化重试一次。 | 两张Map；已有tool-call后禁止整轮重试 | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 26 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `runtime_providers_registry` (createProviderRegistry) | `runtime_providers_definition` (ProviderDefinition) | 持有 / 归属 (`owns`) | 收录 |
| `runtime_providers_config` (useProviderConfigStore) | `runtime_providers_runtime` (useProviderStore) | 数据流 (`data`) | 配置值 |
| `runtime_providers_runtime` (useProviderStore) | `runtime_providers_validate` (validateProviderConfig) | 调用 (`call`) | 检查配置 |
| `runtime_providers_runtime` (useProviderStore) | `runtime_providers_instance` (getProviderInstance) | 调用 (`call`) | 取得可执行实例 |
| `runtime_providers_instance` (getProviderInstance) | `runtime_providers_definition` (ProviderDefinition) | 调用 (`call`) | factory |
| `runtime_providers_definition` (ProviderDefinition) | `runtime_providers_catalog` (listModelCatalog) | 调用 (`call`) | extra发现 |
| `runtime_providers_instance` (getProviderInstance) | `runtime_providers_genprovider` (GenerationProvider) | 调用 (`call`) | 暴露推理能力 |
| `runtime_providers_genprovider` (GenerationProvider) | `runtime_providers_genrequest` (GenerationRequest) | 数据流 (`data`) | generation(model) |
| `runtime_providers_stream` (streamFrom) | `runtime_providers_once` (streamOnce) | 调用 (`call`) | 执行/续跑 |
| `runtime_providers_once` (streamOnce) | `runtime_providers_resolve` (resolveStep) | 调用 (`call`) | 每请求读取 |
| `runtime_providers_resolve` (resolveStep) | `runtime_providers_genrequest` (GenerationRequest) | 数据流 (`data`) | 更新请求目标 |
| `runtime_providers_once` (streamOnce) | `runtime_providers_media` (prepareConversation) | 调用 (`call`) | 协议投影前适配 |
| `runtime_providers_once` (streamOnce) | `runtime_providers_chat` (streamChatCompletions) | 调用 (`call`) | protocol=chat-completions |
| `runtime_providers_once` (streamOnce) | `runtime_providers_responses` (streamResponses) | 调用 (`call`) | protocol=responses |
| `runtime_providers_chat` (streamChatCompletions) | `runtime_providers_chatproject` (conversationToChatMessages) | 调用 (`call`) | 投影 |
| `runtime_providers_responses` (streamResponses) | `runtime_providers_respject` (renderConversation) | 调用 (`call`) | 投影 |
| `runtime_providers_chat` (streamChatCompletions) | `runtime_providers_generation` (createGeneration) | 调用 (`call`) | 保存steps |
| `runtime_providers_responses` (streamResponses) | `runtime_providers_generation` (createGeneration) | 调用 (`call`) | 保存steps |
| `runtime_providers_generation` (createGeneration) | `runtime_providers_step` (prepareStep) | 持有 / 归属 (`owns`) | 记录边界 |
| `runtime_providers_generation` (createGeneration) | `runtime_providers_complete` (complete) | 持有 / 归属 (`owns`) | 组合rounds |
| `runtime_providers_chat` (streamChatCompletions) | `runtime_providers_scope` (createContinuationScope) | 调用 (`call`) | 校验 |
| `runtime_providers_responses` (streamResponses) | `runtime_providers_scope` (createContinuationScope) | 调用 (`call`) | 校验 |
| `runtime_providers_scope` (createContinuationScope) | `runtime_providers_switch` (RequestSwitch) | 调用 (`call`) | 变化时抛出 |
| `runtime_providers_switch` (RequestSwitch) | `runtime_providers_stream` (streamFrom) | 数据流 (`data`) | 携partialTurn续跑 |
| `runtime_providers_retry` (streamWithTransientRetry) | `runtime_providers_once` (streamOnce) | 调用 (`call`) | 安全重试 |
| `runtime_providers_compat` (tools / contentArray兼容缓存) | `runtime_providers_stream` (streamFrom) | 数据流 (`data`) | 兼容性选项 |

---

### 专题 08：工具与输出：注册定义、执行函数、流事件和表现标记 (`runtime_tools`)

> **专题定位**：工具定义、执行器、模型返回的调用、展示切片是不同对象。模型可见工具由多个来源合并，结果走协议adapter与统一StreamEvent，再经parser/categorizer分流。

- **配套矢量图纸**：[职责组织图 (`08-runtime-tools-organization.svg`)](./diagrams/08-runtime-tools-organization.svg) ｜ [调用与数据流图 (`08-runtime-tools-relations.svg`)](./diagrams/08-runtime-tools-relations.svg) ｜ [Graphviz 组织图源](./diagrams/08-runtime-tools-organization.dot) ｜ [Graphviz 关系图源](./diagrams/08-runtime-tools-relations.dot)
- **直接涉及源码范围**：`packages/stage-ui/src/stores/ai/chat-llm/tools.ts`, `packages/stage-ui/src/stores/ai/chat-llm/tool-resolver.ts`, `packages/stage-ui/src/stores/ai/chat-llm/tool-call-summary.ts`, `packages/stage-ui/src/stores/ai/chat-llm/tool-images.ts`, `packages/stage-ui/src/stores/chat.ts`, `crates/alia-core-agent/src/types/llm.rs`, `crates/alia-core-agent/src/runtime/llm-marker-parser.rs`, `crates/alia-core-agent/src/runtime/response-categoriser.rs`, `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs`, `crates/alia-core-agent/src/runtime/agent-hooks.rs`, `packages/stage-ui/src/stores/ai/chat-llm/streaming-control.ts`

#### 读图与工程边界守则
- 工具定义可同步，但execute函数仅存在注册它的本地Map；丢失执行器时返回unavailable，不把函数序列化。
- resolveLlmTools按名称去重，后加入的active runtime注册优先；并为调用包装summary及可选图片描述。
- 模型的文字、推理片段、tool-call与特殊控制标记有不同出口；表情不是通过一个持久情绪对象自动产生。
- STICKER marker在runtime消费，只有已知catalog id可用且每回复至多一个，不作为语音文字输出。

#### 节点与状态归属登记表（共 23 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `runtime_tools_store` | 注册层 | **useLlmToolsStore** | `export const useLlmToolsStore` | `packages/stage-ui/src/stores/ai/chat-llm/tools.ts:48` | 维护工具定义和本地执行器。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_defs` | 注册层 | **ToolDefinition[]** | `const tools = ref<ToolDefinition[]>` | `packages/stage-ui/src/stores/ai/chat-llm/tools.ts:49` | 可序列化schema、id、defaultActive、requiresExplicitSelection。 | 响应式定义列表 | WebUI 面板前端 |
| `runtime_tools_exec` | 注册层 | **executors Map** | `const executors = new Map` | `packages/stage-ui/src/stores/ai/chat-llm/tools.ts:50` | id→execute函数，本地真实能力。 | 不可序列化函数引用 | WebUI 面板前端 |
| `runtime_tools_active` | 选择层 | **activeTools** | `const activeTools = computed` | `packages/stage-ui/src/stores/ai/chat-llm/tools.ts:63` | 筛选默认启用且不需显式选择的工具。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_selected` | 选择层 | **getToolsByNames** | `function getToolsByNames(` | `packages/stage-ui/src/stores/ai/chat-llm/tools.ts:68` | 按模型名称顺序解析用户/历史显式选取的工具。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_resolve` | 选择层 | **resolveLlmTools** | `export async function resolveLlmTools(` | `packages/stage-ui/src/stores/ai/chat-llm/tool-resolver.ts:153` | 并行取得来源、按name合并去重，再包装。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_mcp` | 来源层 | **MCP / builtInTools** | `resolveToolSource(options.builtInTools ?? mcp)` | `packages/stage-ui/src/stores/ai/chat-llm/tool-resolver.ts:162` | 默认内建来源为mcp()。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_spark` | 来源层 | **Spark command工具** | `async function resolveSparkCommandTools(` | `packages/stage-ui/src/stores/ai/chat-llm/tool-resolver.ts:107` | 生成发往模块网络的spark:command；当前删除hallucinated destinations后广播。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_web` | 来源层 | **Web search工具** | `async function resolveWebSearchTools(` | `packages/stage-ui/src/stores/ai/chat-llm/tool-resolver.ts:131` | 只在搜索配置已就绪时提供服务工具。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_custom` | 来源层 | **request-scoped tools** | `async function resolveCustomTools(` | `packages/stage-ui/src/stores/ai/chat-llm/tool-resolver.ts:93` | 当次发送注入memory/recipe/wakeword等工具。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_summary` | 工具包装 | **withCallSummary** | `export function withCallSummary(` | `packages/stage-ui/src/stores/ai/chat-llm/tool-call-summary.ts:67` | 给每个工具调用追加模型描述摘要并在实际执行前去除额外字段。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_images` | 工具包装 | **withDescribedImages** | `export function withDescribedImages(` | `packages/stage-ui/src/stores/ai/chat-llm/tool-images.ts:96` | 需要时把工具结果图像变成描述，普通多模态模型可保留图像。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_run` | 工具包装 | **withRunTools** | `function withRunTools(` | `packages/stage-ui/src/stores/chat.ts:635` | 按session persona注入记忆和Recipe工具，主会话与任务能力边界分开。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_tools_events` | 生成结果 | **StreamEvent union** | `pub enum StreamEvent` | `crates/alia-core-agent/src/types/llm.rs:17` | text/reasoning/tool call/result/error/citation/finish等统一事件。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_tools_parser` | 生成结果 | **useLlmmarkerParser** | `pub fn useLlmmarkerParser(` | `crates/alia-core-agent/src/runtime/llm-marker-parser.rs:184` | 跨chunk识别literal与special token。 | 流缓冲与任务 | Rust 核心实现 |
| `runtime_tools_category` | 生成结果 | **createStreamingCategorizer** | `pub fn createStreamingCategorizer(` | `crates/alia-core-agent/src/runtime/response-categoriser.rs:214` | 把可说文字与reasoning片段分离。 | 累计文本位置与标签状态 | Rust 核心实现 |
| `runtime_tools_literal` | 生成结果 | **onLiteral → speech** | `onLiteral: async (literal)` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:914` | 过滤可说文字，写content/text slices并发literal hook。 | buildingMessage可见文字 | Rust 核心实现 |
| `runtime_tools_special` | 生成结果 | **onSpecial** | `onSpecial: async (special)` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:956` | 非sticker特殊标记进入special hook。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_tools_sticker` | 生成结果 | **STICKER catalog查验** | `if (/^<\\|STICKER\b/.test(special))` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:960` | 只接受请求catalog中的id，存入ChatSlicesSticker。 | 本回复stickerEmitted | Rust 核心实现 |
| `runtime_tools_slices` | 显示与下游 | **buildingMessage.slices** | `let buildingMessage: StreamingAssistantMessage` | `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:763` | UI显示文字、工具调用、sticker、卡片；不等于完整native continuation。 | 实时展示投影 | Rust 核心实现 |
| `runtime_tools_hooks` | 显示与下游 | **token hooks** | `pub fn createChatHooks` | `crates/alia-core-agent/src/runtime/agent-hooks.rs:6` | 把literal/special/stream end等事件交给语音或其它订阅者。 | 订阅列表 | Rust 核心实现 |
| `runtime_tools_controls` | 显示与下游 | **useLlmStreamingControlStore** | `export const useLlmStreamingControlStore` | `packages/stage-ui/src/stores/ai/chat-llm/streaming-control.ts:23` | 把special token交给已注册dispatcher，记录处理结果。 | 控制分发/观察订阅 | WebUI 面板前端 |
| `runtime_tools_rerun` | 显示与下游 | **rerunToolCall** | `async function rerunToolCall(` | `packages/stage-ui/src/stores/chat.ts:1520` | 在指定历史调用上重跑工具；由ALIA调用身份定位。 | 替换调用结果并使依赖native轮次失效 | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 28 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `runtime_tools_store` (useLlmToolsStore) | `runtime_tools_defs` (ToolDefinition[]) | 持有 / 归属 (`owns`) | 拥有 |
| `runtime_tools_store` (useLlmToolsStore) | `runtime_tools_exec` (executors Map) | 持有 / 归属 (`owns`) | 拥有 |
| `runtime_tools_defs` (ToolDefinition[]) | `runtime_tools_active` (activeTools) | 数据流 (`data`) | 默认筛选 |
| `runtime_tools_defs` (ToolDefinition[]) | `runtime_tools_selected` (getToolsByNames) | 数据流 (`data`) | 显式选择 |
| `runtime_tools_exec` (executors Map) | `runtime_tools_active` (activeTools) | 数据流 (`data`) | 补execute |
| `runtime_tools_exec` (executors Map) | `runtime_tools_selected` (getToolsByNames) | 数据流 (`data`) | 补execute |
| `runtime_tools_active` (activeTools) | `runtime_tools_resolve` (resolveLlmTools) | 数据流 (`data`) | 最后合并/同名优先 |
| `runtime_tools_selected` (getToolsByNames) | `runtime_tools_custom` (request-scoped tools) | 数据流 (`data`) | 请求引用 |
| `runtime_tools_mcp` (MCP / builtInTools) | `runtime_tools_resolve` (resolveLlmTools) | 数据流 (`data`) | 内建来源 |
| `runtime_tools_spark` (Spark command工具) | `runtime_tools_resolve` (resolveLlmTools) | 数据流 (`data`) | 命令来源 |
| `runtime_tools_web` (Web search工具) | `runtime_tools_resolve` (resolveLlmTools) | 数据流 (`data`) | 搜索来源 |
| `runtime_tools_run` (withRunTools) | `runtime_tools_custom` (request-scoped tools) | 数据流 (`data`) | 按当前session注入 |
| `runtime_tools_custom` (request-scoped tools) | `runtime_tools_resolve` (resolveLlmTools) | 数据流 (`data`) | 请求来源 |
| `runtime_tools_resolve` (resolveLlmTools) | `runtime_tools_summary` (withCallSummary) | 调用 (`call`) | 包装 |
| `runtime_tools_summary` (withCallSummary) | `runtime_tools_images` (withDescribedImages) | 调用 (`call`) | 可选包装 |
| `runtime_tools_images` (withDescribedImages) | `runtime_tools_events` (StreamEvent union) | 数据流 (`data`) | 模型工具执行返回统一事件 |
| `runtime_tools_events` (StreamEvent union) | `runtime_tools_parser` (useLlmmarkerParser) | 数据流 (`data`) | text delta |
| `runtime_tools_parser` (useLlmmarkerParser) | `runtime_tools_literal` (onLiteral → speech) | 调用 (`call`) | literal |
| `runtime_tools_literal` (onLiteral → speech) | `runtime_tools_category` (createStreamingCategorizer) | 调用 (`call`) | speech过滤 |
| `runtime_tools_parser` (useLlmmarkerParser) | `runtime_tools_special` (onSpecial) | 调用 (`call`) | special |
| `runtime_tools_special` (onSpecial) | `runtime_tools_sticker` (STICKER catalog查验) | 调用 (`call`) | 先处理sticker |
| `runtime_tools_sticker` (STICKER catalog查验) | `runtime_tools_slices` (buildingMessage.slices) | 调用 (`call`) | 保存合法sticker |
| `runtime_tools_literal` (onLiteral → speech) | `runtime_tools_slices` (buildingMessage.slices) | 调用 (`call`) | 保存文本 |
| `runtime_tools_events` (StreamEvent union) | `runtime_tools_slices` (buildingMessage.slices) | 调用 (`call`) | 工具call/result |
| `runtime_tools_literal` (onLiteral → speech) | `runtime_tools_hooks` (token hooks) | 调用 (`call`) | literal hook |
| `runtime_tools_special` (onSpecial) | `runtime_tools_hooks` (token hooks) | 调用 (`call`) | special hook |
| `runtime_tools_hooks` (token hooks) | `runtime_tools_controls` (useLlmStreamingControlStore) | 数据流 (`data`) | control dispatcher |
| `runtime_tools_rerun` (rerunToolCall) | `runtime_tools_exec` (executors Map) | 调用 (`call`) | 定位执行器 |

---

### 专题 09：记忆与技能：条目、决策、后台任务与真实边界 (`runtime_memory`)

> **专题定位**：长期记忆是索引+事实条目。Recipe是模型遵循的说明或选择题；background Recipe在独立隐藏session执行，并以notice把结果交回父会话。

- **配套矢量图纸**：[职责组织图 (`09-runtime-memory-organization.svg`)](./diagrams/09-runtime-memory-organization.svg) ｜ [调用与数据流图 (`09-runtime-memory-relations.svg`)](./diagrams/09-runtime-memory-relations.svg) ｜ [Graphviz 组织图源](./diagrams/09-runtime-memory-organization.dot) ｜ [Graphviz 关系图源](./diagrams/09-runtime-memory-relations.dot)
- **直接涉及源码范围**：`packages/stage-ui/src/stores/memory.ts`, `packages/stage-ui/src/types/memory.ts`, `packages/stage-ui/src/tools/memory.ts`, `crates/alia-storage/src/repos/memory.repo.rs`, `packages/stage-ui/src/database/storage.ts`, `crates/alia-core-agent/src/runtime/recipe.rs`, `packages/stage-ui/src/stores/recipes.ts`, `crates/alia-storage/src/repos/recipes.repo.rs`, `packages/stage-ui/src/tools/use-recipe.ts`, `packages/stage-ui/src/tools/judge-recipe.ts`, `packages/stage-ui/src/tools/propose-recipe.ts`, `packages/stage-ui/src/stores/chat.ts`, `packages/stage-ui/src/types/chat-session.ts`, `crates/alia-core-agent/src/lib.rs`

#### 读图与工程边界守则
- memory.enabled关掉时不注入索引与记忆工具。模型只写/忘自己的persona条目；general条目由用户在Settings修改。
- write/forget在repo的navigator.locks临界区读并写，避免多窗口旧镜像恢复已删除条目。
- Recipe模型提案默认approved:false/enabled:false；owner写入的recipe默认approved:true。
- 实际ChatSessionTaskStatus只有running/done/failed/interrupted。ADR的armed/stopped描述与源码不一致。
- Automation/checkAutomations/isAutoRunRecipe/builtIn_armRecipe/modelTimed只在README或ADR被声明；当前apps/packages源码未检出。必须另标文档愿景，不能接入已实现实线图。

#### 节点与状态归属登记表（共 24 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `runtime_memory_memstore` | 长期记忆 | **useMemoryStore** | `export const useMemoryStore` | `packages/stage-ui/src/stores/memory.ts:46` | 提供persona过滤、条目操作、跨窗口reload。 | entries镜像与enabled | WebUI 面板前端 |
| `runtime_memory_entry` | 长期记忆 | **MemoryEntry** | `export interface MemoryEntry` | `packages/stage-ui/src/types/memory.ts:2` | id/description/body/persona/updatedAt，一条事实一个记录。 | 持久条目 | WebUI 面板前端 |
| `runtime_memory_index` | 长期记忆 | **indexFor(personaId)** | `function indexFor(` | `packages/stage-ui/src/stores/memory.ts:79` | 只列本人+general的id与description，正文按工具读取。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_memory_memtools` | 长期记忆 | **createMemoryTools** | `export async function createMemoryTools(` | `packages/stage-ui/src/tools/memory.ts:54` | builtIn_readMemory/writeMemory/forgetMemory绑定运行persona。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_memory_memrepo` | 长期记忆 | **memoryRepo** | `export let memoryRepo` | `crates/alia-storage/src/repos/memory.repo.rs:34` | 每次写入锁住read-modify-write；删除写tombstone。 | IDB memory/entries；UUIDv7 | Rust 核心实现 |
| `runtime_memory_storage` | 持久化 | **Rust SQLx (SQLite + sqlite-vec) storage mounts** | `storage.mount('memory'` | `packages/stage-ui/src/database/storage.ts:26` | memory和recipes独立store，Blob/media另存。 | 设备持久化 | WebUI 面板前端 |
| `runtime_memory_broadcast` | 持久化 | **BroadcastChannel alia:memory** | `name: 'alia:memory'` | `packages/stage-ui/src/stores/memory.ts:49` | 变更后通知其他窗口重读；不是跨设备云同步。 | 同源窗口通知 | WebUI 面板前端 |
| `runtime_memory_recipe` | 技能定义 | **Recipe** | `pub trait Recipe {` | `crates/alia-core-agent/src/runtime/recipe.rs:42` | instructions或decision；keywords/background；enabled/approved。 | 可序列化技能 | Rust 核心实现 |
| `runtime_memory_recstore` | 技能定义 | **useRecipesStore** | `export const useRecipesStore` | `packages/stage-ui/src/stores/recipes.ts:54` | owner新增、模型提案、批准、启停与usable列表。 | records镜像/proposalsEnabled | WebUI 面板前端 |
| `runtime_memory_recrepo` | 持久化 | **recipesRepo** | `export let recipesRepo` | `crates/alia-storage/src/repos/recipes.repo.rs:46` | 保存逐Recipe记录、只更新所改字段、删除留tombstone。 | IDB recipes/recipes | Rust 核心实现 |
| `runtime_memory_usable` | 技能选择 | **usableRecipes** | `pub fn usableRecipes(` | `crates/alia-core-agent/src/runtime/recipe.rs:74` | 只选enabled && approved。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_memory_keyword` | 技能选择 | **matchKeywordRecipes** | `pub fn matchKeywordRecipes(` | `crates/alia-core-agent/src/runtime/recipe.rs:84` | 忽略大小写查关键词，效果类似owner slash command。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_memory_use` | 技能选择 | **builtIn_useRecipe** | `export async function createUseRecipeTool(` | `packages/stage-ui/src/tools/use-recipe.ts:122` | 描述中只列名称用途；工具返回步骤或启动后台session。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_memory_decision` | 决策技能 | **RecipeDecision + judgeDecision** | `pub fn judgeDecision(` | `crates/alia-core-agent/src/runtime/recipe.rs:114` | 模型选yes/no或编号，代码按action映射执行；无置信度或unsure分支。 | 无独立持久状态；按调用范围生存 | Rust 核心实现 |
| `runtime_memory_judge` | 决策技能 | **builtIn_judge** | `export async function createJudgeRecipeTool(` | `packages/stage-ui/src/tools/judge-recipe.ts:107` | 把回答映射到stay-quiet、hint、recipe或正常回复。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_memory_propose` | 决策技能 | **builtIn_proposeRecipe** | `export async function createProposeRecipeTool(` | `packages/stage-ui/src/tools/propose-recipe.ts:111` | 创建待owner审批的模型Recipe建议。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_memory_start` | 后台执行 | **startRecipe** | `async function startRecipe(` | `packages/stage-ui/src/stores/chat.ts:491` | 新建该persona的hidden task session，history初始为空。 | session meta含parentSessionId/recipeId | WebUI 面板前端 |
| `runtime_memory_background` | 后台执行 | **runBackgroundTask** | `async function runBackgroundTask(` | `packages/stage-ui/src/stores/chat.ts:461` | background:true提交，独立session并行，无声音。 | 任务状态running→done/failed | WebUI 面板前端 |
| `runtime_memory_status` | 后台执行 | **ChatSessionTaskStatus** | `export type ChatSessionTaskStatus` | `packages/stage-ui/src/types/chat-session.ts:13` | 源码仅running/done/failed/interrupted。 | 持久session.task.status | WebUI 面板前端 |
| `runtime_memory_notice` | 后台执行 | **notifyConversation** | `async function notifyConversation(` | `packages/stage-ui/src/stores/chat.ts:424` | 任务结果作为带来源notice送给父会话，主Agent可说或保持安静。 | history保留notice来源 | WebUI 面板前端 |
| `runtime_memory_stop` | 后台执行 | **stopBackgroundTask** | `async function stopBackgroundTask(` | `packages/stage-ui/src/stores/chat.ts:522` | 改为interrupted，abort请求，已停止任务不回报结果。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_memory_prune` | 后台执行 | **pruneFinishedTasks** | `async function pruneFinishedTasks(` | `packages/stage-ui/src/stores/chat.ts:548` | 只保留最近若干finished task session；当前常量3。 | 删除旧hidden session | WebUI 面板前端 |
| `runtime_memory_orphan` | 后台执行 | **interruptOrphanedTasks** | `async function interruptOrphanedTasks(` | `packages/stage-ui/src/stores/chat.ts:560` | 新leader把前leader遗留running任务标interrupted，不恢复执行。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_memory_automationdocs` | 文档与实现差异 | **Automation文档声明 · 当前未检出实现** | `## Automations` | `crates/alia-core-agent/src/lib.rs:97` | README声明trigger/conditions/cooldown及checkAutomations；该提交源码没有这些符号。 | 文档边界，勿画已实现调用 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 24 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `runtime_memory_memstore` (useMemoryStore) | `runtime_memory_entry` (MemoryEntry) | 持有 / 归属 (`owns`) | 维护镜像 |
| `runtime_memory_memstore` (useMemoryStore) | `runtime_memory_index` (indexFor(personaId)) | 调用 (`call`) | 生成可见索引 |
| `runtime_memory_index` (indexFor(personaId)) | `runtime_memory_memtools` (createMemoryTools) | 数据流 (`data`) | 按id读取正文 |
| `runtime_memory_memtools` (createMemoryTools) | `runtime_memory_memstore` (useMemoryStore) | 调用 (`call`) | persona限制读写 |
| `runtime_memory_memstore` (useMemoryStore) | `runtime_memory_memrepo` (memoryRepo) | 调用 (`call`) | 落盘 |
| `runtime_memory_memrepo` (memoryRepo) | `runtime_memory_storage` (Rust SQLx (SQLite + sqlite-vec) storage mounts) | 调用 (`call`) | 写条目/tombstone |
| `runtime_memory_memstore` (useMemoryStore) | `runtime_memory_broadcast` (BroadcastChannel alia:memory) | 调用 (`call`) | publish变更 |
| `runtime_memory_recstore` (useRecipesStore) | `runtime_memory_recipe` (Recipe) | 持有 / 归属 (`owns`) | 维护列表 |
| `runtime_memory_recstore` (useRecipesStore) | `runtime_memory_recrepo` (recipesRepo) | 调用 (`call`) | 逐字段落盘 |
| `runtime_memory_recrepo` (recipesRepo) | `runtime_memory_storage` (Rust SQLx (SQLite + sqlite-vec) storage mounts) | 调用 (`call`) | 写recipe记录 |
| `runtime_memory_recstore` (useRecipesStore) | `runtime_memory_usable` (usableRecipes) | 调用 (`call`) | 筛选 |
| `runtime_memory_usable` (usableRecipes) | `runtime_memory_keyword` (matchKeywordRecipes) | 数据流 (`data`) | owner文字触发 |
| `runtime_memory_usable` (usableRecipes) | `runtime_memory_use` (builtIn_useRecipe) | 数据流 (`data`) | 模型可见技能 |
| `runtime_memory_keyword` (matchKeywordRecipes) | `runtime_memory_decision` (RecipeDecision + judgeDecision) | 调用 (`call`) | 决策型 |
| `runtime_memory_decision` (RecipeDecision + judgeDecision) | `runtime_memory_judge` (builtIn_judge) | 数据流 (`data`) | 选项映射动作 |
| `runtime_memory_propose` (builtIn_proposeRecipe) | `runtime_memory_recstore` (useRecipesStore) | 调用 (`call`) | unapproved提案 |
| `runtime_memory_use` (builtIn_useRecipe) | `runtime_memory_start` (startRecipe) | 调用 (`call`) | background技能 |
| `runtime_memory_keyword` (matchKeywordRecipes) | `runtime_memory_start` (startRecipe) | 调用 (`call`) | background关键词 |
| `runtime_memory_start` (startRecipe) | `runtime_memory_background` (runBackgroundTask) | 调用 (`call`) | 独立session执行 |
| `runtime_memory_background` (runBackgroundTask) | `runtime_memory_status` (ChatSessionTaskStatus) | 调用 (`call`) | 更新状态 |
| `runtime_memory_background` (runBackgroundTask) | `runtime_memory_notice` (notifyConversation) | 调用 (`call`) | 完成结果 |
| `runtime_memory_stop` (stopBackgroundTask) | `runtime_memory_status` (ChatSessionTaskStatus) | 调用 (`call`) | interrupted |
| `runtime_memory_orphan` (interruptOrphanedTasks) | `runtime_memory_status` (ChatSessionTaskStatus) | 调用 (`call`) | interrupted |
| `runtime_memory_background` (runBackgroundTask) | `runtime_memory_prune` (pruneFinishedTasks) | 调用 (`call`) | 收尾清理 |

---

### 专题 10：会话数据所有权：窗口选择、本地记录、媒体与云同步 (`runtime_storage`)

> **专题定位**：当前窗口选中哪段对话、哪个角色拥有对话、执行中的生成、设备历史、云端文本副本分别有自己的owner。云同步不等于同步完整Agent执行状态。

- **配套矢量图纸**：[职责组织图 (`10-runtime-storage-organization.svg`)](./diagrams/10-runtime-storage-organization.svg) ｜ [调用与数据流图 (`10-runtime-storage-relations.svg`)](./diagrams/10-runtime-storage-relations.svg) ｜ [Graphviz 组织图源](./diagrams/10-runtime-storage-organization.dot) ｜ [Graphviz 关系图源](./diagrams/10-runtime-storage-relations.dot)
- **直接涉及源码范围**：`packages/stage-ui/src/stores/chat/session-store.ts`, `packages/stage-ui/src/types/chat-session.ts`, `crates/alia-storage/src/repos/chat-sessions.repo.rs`, `packages/stage-ui/src/database/storage.ts`, `crates/alia-storage/src/repos/chat-assets.repo.rs`, `packages/stage-ui/src/stores/chat/session-locality.ts`, `packages/stage-ui/src/libs/chat-sync/wire-message.ts`, `packages/stage-ui/src/libs/chat-sync/cloud-mapper.ts`, `packages/stage-ui/src/libs/chat-sync/ws-client.ts`

#### 读图与工程边界守则
- activeSessionId放在window-local selection store；共享会话数据不应强迫不同窗口同时选择同一session。
- 同一个messageId的user commit幂等；assistant storeMessage按id替换。generation清理阻止旧请求写回。
- scene与background task session staysLocal；notice/proactive、中断、音频输入及对应回复留本地。普通图片消息可同步文本部分，云端不恢复媒体或原生执行记录。
- 本地media用content hash和session ownership保留；fork之后删除原session不会删除副本仍引用的asset。
- 云同步的outbox落盘，重试沿用messageId；tombstone防止离线删除的会话被重新导入。

#### 节点与状态归属登记表（共 22 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `runtime_storage_store` | 会话所有者 | **useChatSessionStore** | `export const useChatSessionStore` | `packages/stage-ui/src/stores/chat/session-store.ts:69` | session生命周期与本地/云同步owner。 | messages/metas/generations/index | WebUI 面板前端 |
| `runtime_storage_selection` | 会话所有者 | **useChatSessionSelectionStore** | `const useChatSessionSelectionStore` | `packages/stage-ui/src/stores/chat/session-store.ts:63` | 选择的session属于单窗口。 | activeSessionId | WebUI 面板前端 |
| `runtime_storage_messages` | 会话所有者 | **sessionMessages** | `const sessionMessages = ref<` | `packages/stage-ui/src/stores/chat/session-store.ts:88` | sessionId→本地完整history。 | generationTranscript/media引用/可见中断 | WebUI 面板前端 |
| `runtime_storage_meta` | 会话所有者 | **ChatSessionMeta** | `export interface ChatSessionMeta` | `packages/stage-ui/src/types/chat-session.ts:22` | character members、bindings、hidden、parent、recipe与task元数据。 | 会话元数据 | WebUI 面板前端 |
| `runtime_storage_generations` | 会话所有者 | **sessionGenerations** | `const sessionGenerations = ref<` | `packages/stage-ui/src/stores/chat/session-store.ts:90` | 单调代数标记，reset/清理后拒绝旧执行。 | 每session generation | WebUI 面板前端 |
| `runtime_storage_commit` | 本地持久化 | **commitUserMessage** | `async function commitUserMessage(` | `packages/stage-ui/src/stores/chat/session-store.ts:329` | 相同id不重复插入，等待存储后回执。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_storage_storemsg` | 本地持久化 | **storeMessage** | `async function storeMessage(` | `packages/stage-ui/src/stores/chat/session-store.ts:361` | 按id存储/替换助手消息，避免重试重复追加。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_storage_load` | 本地持久化 | **loadSession** | `async function loadSession(` | `packages/stage-ui/src/stores/chat/session-store.ts:453` | 按需读取session；合并已加载数据与正在变化的历史。 | loadingSessions去重 | WebUI 面板前端 |
| `runtime_storage_repo` | 本地持久化 | **chatSessionsRepo** | `export let chatSessionsRepo` | `crates/alia-storage/src/repos/chat-sessions.repo.rs:44` | 按user存index，按session存完整record，同时retain media。 | IDB local:chat/* | Rust 核心实现 |
| `runtime_storage_idb` | 本地持久化 | **storage local mount** | `storage.mount('local'` | `packages/stage-ui/src/database/storage.ts:12` | unstorage Rust SQLx (SQLite + sqlite-vec) driver；持久chat/索引/队列。 | alia-local | WebUI 面板前端 |
| `runtime_storage_assets` | 媒体所有权 | **chatAssetsRepo** | `export let chatAssetsRepo` | `crates/alia-storage/src/repos/chat-assets.repo.rs:26` | content hash媒体blob与session owners；最后owner释放才删。 | 独立chat-assets数据库 | Rust 核心实现 |
| `runtime_storage_retain` | 媒体所有权 | **retainAssets** | `pub async fn retainAssets(` | `crates/alia-storage/src/repos/chat-sessions.repo.rs:34` | 新引用加入session ownership。 | ownedAssets per-window Map | Rust 核心实现 |
| `runtime_storage_fork` | 媒体所有权 | **forkSession** | `async function forkSession(` | `packages/stage-ui/src/stores/chat/session-store.ts:1875` | 复制指定历史到新session并保留资源引用。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_storage_local` | 同步过滤 | **staysLocal** | `export function staysLocal(` | `packages/stage-ui/src/stores/chat/session-locality.ts:7` | 按scene/task元数据界定设备本地session。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_storage_eligible` | 同步过滤 | **isCloudSyncableMessage** | `export function isCloudSyncableMessage(` | `packages/stage-ui/src/libs/chat-sync/wire-message.ts:80` | 排除tool/system/error、notice/proactive、中断和音频输入；图片消息仅文本部分同步。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_storage_mapper` | 云端副本 | **createCloudChatMapper** | `export function createCloudChatMapper(` | `packages/stage-ui/src/libs/chat-sync/cloud-mapper.ts:138` | 本地session与云chat的创建、列举、删除等API映射。 | cloud id映射 | WebUI 面板前端 |
| `runtime_storage_reconcile` | 云端副本 | **reconcileCloudSessions** | `async function reconcileCloudSessions(` | `packages/stage-ui/src/stores/chat/session-store.ts:941` | 比较本地/远端元数据，建立映射或导入，并尊重tombstone。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |
| `runtime_storage_outbox` | 云端副本 | **ChatSendOutboxEntry** | `pub trait ChatSendOutboxEntry` | `crates/alia-storage/src/repos/chat-sessions.repo.rs:18` | messageId/sessionId/cloudChatId/text/attempts/lastError。 | 设备持久发送队列 | Rust 核心实现 |
| `runtime_storage_drain` | 云端副本 | **drainOutbox** | `async function drainOutbox()` | `packages/stage-ui/src/stores/chat/session-store.ts:1487` | 按会话发送待同步文本；重试不换messageId。 | attempts；失败队列保留 | WebUI 面板前端 |
| `runtime_storage_tombstone` | 云端副本 | **getTombstones / 删除墓碑** | `async getTombstones(` | `crates/alia-storage/src/repos/chat-sessions.repo.rs:109` | 离线删除会话留标记，防reconcile把云旧记录导回。 | 按user的cloudChatId集合 | Rust 核心实现 |
| `runtime_storage_ws` | 云端副本 | **createChatWsClient** | `export function createChatWsClient(` | `packages/stage-ui/src/libs/chat-sync/ws-client.ts:207` | 云聊天新消息事件、认证状态与重连。 | ws lifecycle | WebUI 面板前端 |
| `runtime_storage_merge` | 云端副本 | **mergeCloudMessagesIntoLocal** | `export function mergeCloudMessagesIntoLocal(` | `packages/stage-ui/src/libs/chat-sync/wire-message.ts:202` | 按稳定消息身份合并云端文本，保留本地更丰富记录。 | 无独立持久状态；按调用范围生存 | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 22 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `runtime_storage_store` (useChatSessionStore) | `runtime_storage_selection` (useChatSessionSelectionStore) | 调用 (`call`) | 读取窗口选择 |
| `runtime_storage_store` (useChatSessionStore) | `runtime_storage_messages` (sessionMessages) | 持有 / 归属 (`owns`) | 拥有 |
| `runtime_storage_store` (useChatSessionStore) | `runtime_storage_meta` (ChatSessionMeta) | 持有 / 归属 (`owns`) | 拥有 |
| `runtime_storage_store` (useChatSessionStore) | `runtime_storage_generations` (sessionGenerations) | 持有 / 归属 (`owns`) | 拥有 |
| `runtime_storage_commit` (commitUserMessage) | `runtime_storage_messages` (sessionMessages) | 调用 (`call`) | 幂等追加 |
| `runtime_storage_storemsg` (storeMessage) | `runtime_storage_messages` (sessionMessages) | 调用 (`call`) | 按id存储 |
| `runtime_storage_commit` (commitUserMessage) | `runtime_storage_repo` (chatSessionsRepo) | 调用 (`call`) | await落盘 |
| `runtime_storage_storemsg` (storeMessage) | `runtime_storage_repo` (chatSessionsRepo) | 调用 (`call`) | await落盘 |
| `runtime_storage_load` (loadSession) | `runtime_storage_repo` (chatSessionsRepo) | 调用 (`call`) | 读取记录 |
| `runtime_storage_repo` (chatSessionsRepo) | `runtime_storage_idb` (storage local mount) | 调用 (`call`) | 持久化 |
| `runtime_storage_repo` (chatSessionsRepo) | `runtime_storage_retain` (retainAssets) | 调用 (`call`) | 保存前认领媒体 |
| `runtime_storage_retain` (retainAssets) | `runtime_storage_assets` (chatAssetsRepo) | 调用 (`call`) | retain(sessionId) |
| `runtime_storage_fork` (forkSession) | `runtime_storage_repo` (chatSessionsRepo) | 调用 (`call`) | 复制并保存 |
| `runtime_storage_meta` (ChatSessionMeta) | `runtime_storage_local` (staysLocal) | 数据流 (`data`) | session eligibility |
| `runtime_storage_messages` (sessionMessages) | `runtime_storage_eligible` (isCloudSyncableMessage) | 数据流 (`data`) | message eligibility |
| `runtime_storage_eligible` (isCloudSyncableMessage) | `runtime_storage_outbox` (ChatSendOutboxEntry) | 数据流 (`data`) | 可同步文本入队 |
| `runtime_storage_local` (staysLocal) | `runtime_storage_reconcile` (reconcileCloudSessions) | 调用 (`call`) | 过滤local-only |
| `runtime_storage_reconcile` (reconcileCloudSessions) | `runtime_storage_mapper` (createCloudChatMapper) | 调用 (`call`) | 远端元数据操作 |
| `runtime_storage_outbox` (ChatSendOutboxEntry) | `runtime_storage_drain` (drainOutbox) | 数据流 (`data`) | 批次处理 |
| `runtime_storage_tombstone` (getTombstones / 删除墓碑) | `runtime_storage_reconcile` (reconcileCloudSessions) | 调用 (`call`) | 阻止重新导入 |
| `runtime_storage_ws` (createChatWsClient) | `runtime_storage_merge` (mergeCloudMessagesIntoLocal) | 数据流 (`data`) | 云新文本 |
| `runtime_storage_merge` (mergeCloudMessagesIntoLocal) | `runtime_storage_messages` (sessionMessages) | 调用 (`call`) | 合并 |

---

### 专题 11：主动反应旁路：Spark、Notebook与Character Orchestrator (`spark`)

> **专题定位**：外部通知、到期任务、头像触摸进入通知队列；独立Spark Agent可以说话、不回应或给外部模块命令。

- **配套矢量图纸**：[职责组织图 (`11-spark-organization.svg`)](./diagrams/11-spark-organization.svg) ｜ [调用与数据流图 (`11-spark-relations.svg`)](./diagrams/11-spark-relations.svg) ｜ [Graphviz 组织图源](./diagrams/11-spark-organization.dot) ｜ [Graphviz 关系图源](./diagrams/11-spark-relations.dot)
- **直接涉及源码范围**：`packages/stage-ui/src/stores/character/orchestrator/store.ts`, `packages/stage-ui/src/stores/character/avatar-touch.ts`, `packages/stage-ui/src/stores/character/notebook.ts`, `crates/alia-core-agent/src/agents/spark-notify/agent.rs`, `crates/alia-core-agent/src/agents/spark-notify/types.rs`, `crates/alia-core-agent/src/agents/spark-notify/tools.rs`, `crates/alia-core-agent/src/agents/spark-notify/plugins/reaction.rs`, `packages/stage-ui/src/stores/character/index.ts`

#### 读图与工程边界守则
- 这是独立通知Agent路径，不是主会话自动把所有消息循环一遍。
- 后台通知消费者和tick只由同步leader运行；失去leader/卸载即停止订阅与ticker。
- 此路径的provider/model来自全局Consciousness，提示词来自当前Character；不能画成主chat同样的按session配置解析。
- Notebook在此文件中用ref持有entries/tasks；未看到此store的持久化适配，不能称为长期记忆数据库。
- spark:emit的handler当前是no-op。
- processSparkNotify非立即处理或处理中的通知进入队列；失败按attempts/maxAttempts重排，达到上限丢弃并记录。

#### 节点与状态归属登记表（共 20 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `spark_external` | 通知来源 | **外部 spark:notify** | `modsServerChannelStore.onEvent` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:276` | 来自server channel的外部事件 | urgency / id / payload | WebUI 面板前端 |
| `spark_touch` | 通知来源 | **AvatarTouch Store** | `useAvatarTouchStore` | `packages/stage-ui/src/stores/character/avatar-touch.ts:19` | leader路由触摸通知并统一冷却 | region cooldown | WebUI 面板前端 |
| `spark_notebook` | 通知来源 | **Character Notebook** | `useCharacterNotebookStore` | `packages/stage-ui/src/stores/character/notebook.ts:33` | 管理note/diary/focus与任务 | entries / tasks 为ref | WebUI 面板前端 |
| `spark_tasks` | 通知来源 | **getDueTasks** | `getDueTasks` | `packages/stage-ui/src/stores/character/notebook.ts:123` | 查询时间窗内未结束任务 | dueAt / nextNotifyAt / priority | WebUI 面板前端 |
| `spark_orchestrator` | 应用调度 | **Character Orchestrator** | `useCharacterOrchestratorStore` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:17` | 通知调度、Agent调用、输出命令 | processing / pendingNotifies / scheduledNotifies | WebUI 面板前端 |
| `spark_leader` | 应用调度 | **同步Leader生命周期** | `initialize / onLeadershipChange` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:315` | leader启动消费者，follower停止 | leadership / stopLeadershipListener | WebUI 面板前端 |
| `spark_scheduler` | 应用调度 | **tick / enqueueDueTasks** | `enqueueDueTasks / tick` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:186` | 生成提醒并取出到期队列项 | tickInterval / taskWindow | WebUI 面板前端 |
| `spark_queue` | 应用调度 | **scheduledNotifies** | `scheduledNotifies / computeNextRunAt` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:30` | 紧急程度决定时间；失败重试 | nextRunAt / attempts / maxAttempts | WebUI 面板前端 |
| `spark_selection` | 通知Agent | **Consciousness + Character** | `processSparkNotify` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:122` | 全局provider/model + 当前角色prompt | 不是主chat会话选择 | WebUI 面板前端 |
| `spark_agent` | 通知Agent | **createSparkNotifyAgent** | `createSparkNotifyAgent` | `crates/alia-core-agent/src/agents/spark-notify/agent.rs:180` | 为一个通知建立conversation和插件session | handle(request) | Rust 核心实现 |
| `spark_prepare` | 通知Agent | **Spark plugin sessions** | `plugins.prepare` | `crates/alia-core-agent/src/agents/spark-notify/agent.rs:187` | 收集systemInstructions、userSections、tools | 每通知独立session | Rust 核心实现 |
| `spark_runner` | 通知Agent | **SparkNotifyRunner** | `SparkNotifyRunner` | `crates/alia-core-agent/src/agents/spark-notify/types.rs:151` | 外部注入执行端；Stage用useLLM.stream | run(request) | Rust 核心实现 |
| `spark_noresponse` | 通知决策 | **builtIn_sparkNoResponse** | `builtIn_sparkNoResponse` | `crates/alia-core-agent/src/agents/spark-notify/tools.rs:134` | 允许模型选择本次不回应 | noResponse | Rust 核心实现 |
| `spark_command` | 通知决策 | **builtIn_sparkCommand** | `builtIn_sparkCommand` | `crates/alia-core-agent/src/agents/spark-notify/tools.rs:151` | 校验并收集给子模块的命令草案 | commands[] | Rust 核心实现 |
| `spark_reaction` | 反应呈现 | **ReactionPlugin** | `createSparkNotifyReactionPlugin` | `crates/alia-core-agent/src/agents/spark-notify/plugins/reaction.rs:12` | 把可见文本delta/end交宿主 | onDelta / onEnd | Rust 核心实现 |
| `spark_character` | 反应呈现 | **Character Store** | `useCharacterStore` | `packages/stage-ui/src/stores/character/index.ts:38` | 管理文本反应历史及流状态 | reactions上限200 / streamingReactions | WebUI 面板前端 |
| `spark_speech` | 反应呈现 | **SpeechClient 通知producer** | `new Speech(..., notification)` | `packages/stage-ui/src/stores/character/index.ts:78` | 把通知反应literal/special交音频宿主 | turnId=spark:eventId | WebUI 面板前端 |
| `spark_out` | 通知决策 | **spark:command出站** | `modsServerChannelStore.send` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:148` | 扩展并投递Agent给出的命令 | destinations / event lineage | WebUI 面板前端 |
| `spark_noop` | 边界与清理 | **spark:emit 当前空实现** | `handleSparkEmit` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:265` | 消费入口存在，但现在直接return undefined | no-op | WebUI 面板前端 |
| `spark_dispose` | 边界与清理 | **dispose / stopConsumers** | `dispose` | `packages/stage-ui/src/stores/character/orchestrator/store.ts:328` | 取消leader监听、ticker与事件订阅 | 重置initialized | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 23 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `spark_external` (外部 spark:notify) | `spark_orchestrator` (Character Orchestrator) | 事件 (`event`) | handleIncomingSparkNotify |
| `spark_touch` (AvatarTouch Store) | `spark_orchestrator` (Character Orchestrator) | 调用 (`call`) | handleSparkNotify |
| `spark_notebook` (Character Notebook) | `spark_tasks` (getDueTasks) | 调用 (`call`) | 提供到期任务 |
| `spark_tasks` (getDueTasks) | `spark_scheduler` (tick / enqueueDueTasks) | 数据流 (`data`) | 任务列表 |
| `spark_leader` (同步Leader生命周期) | `spark_orchestrator` (Character Orchestrator) | 生命周期 (`lifecycle`) | 控制消费者归属 |
| `spark_scheduler` (tick / enqueueDueTasks) | `spark_queue` (scheduledNotifies) | 调用 (`call`) | 提醒入队/取到期项 |
| `spark_orchestrator` (Character Orchestrator) | `spark_queue` (scheduledNotifies) | 调用 (`call`) | 忙碌/非立即通知入队 |
| `spark_queue` (scheduledNotifies) | `spark_selection` (Consciousness + Character) | 调用 (`call`) | processSparkNotify |
| `spark_orchestrator` (Character Orchestrator) | `spark_selection` (Consciousness + Character) | 调用 (`call`) | 立即且空闲时直达 |
| `spark_selection` (Consciousness + Character) | `spark_agent` (createSparkNotifyAgent) | 数据流 (`data`) | event + provider + prompt |
| `spark_agent` (createSparkNotifyAgent) | `spark_prepare` (Spark plugin sessions) | 调用 (`call`) | 并行prepare |
| `spark_prepare` (Spark plugin sessions) | `spark_noresponse` (builtIn_sparkNoResponse) | 持有 / 归属 (`owns`) | 加入工具 |
| `spark_prepare` (Spark plugin sessions) | `spark_command` (builtIn_sparkCommand) | 持有 / 归属 (`owns`) | 加入工具 |
| `spark_agent` (createSparkNotifyAgent) | `spark_runner` (SparkNotifyRunner) | 调用 (`call`) | run conversation/tools |
| `spark_runner` (SparkNotifyRunner) | `spark_noresponse` (builtIn_sparkNoResponse) | 调用 (`call`) | 可选择不回应 |
| `spark_runner` (SparkNotifyRunner) | `spark_command` (builtIn_sparkCommand) | 调用 (`call`) | 可发出命令 |
| `spark_runner` (SparkNotifyRunner) | `spark_reaction` (ReactionPlugin) | 事件 (`event`) | 流式text-delta/result |
| `spark_reaction` (ReactionPlugin) | `spark_character` (Character Store) | 事件 (`event`) | onDelta/onEnd |
| `spark_character` (Character Store) | `spark_speech` (SpeechClient 通知producer) | 调用 (`call`) | literal/special输出 |
| `spark_command` (builtIn_sparkCommand) | `spark_out` (spark:command出站) | 数据流 (`data`) | 经schema验证与展开 |
| `spark_orchestrator` (Character Orchestrator) | `spark_noop` (spark:emit 当前空实现) | 调用 (`call`) | spark:emit入口 |
| `spark_leader` (同步Leader生命周期) | `spark_dispose` (dispose / stopConsumers) | 生命周期 (`lifecycle`) | 失去领导时停止消费者 |
| `spark_dispose` (dispose / stopConsumers) | `spark_scheduler` (tick / enqueueDueTasks) | 生命周期 (`lifecycle`) | 清除ticker；停止消费者 |

---

### 专题 12：视觉感知的两条入口与并发边界 (`vision`)

> **专题定位**：聊天附件/工具图片按会话选择模型；周期性屏幕观察经过独立编排，再可选写入模块上下文。

- **配套矢量图纸**：[职责组织图 (`12-vision-organization.svg`)](./diagrams/12-vision-organization.svg) ｜ [调用与数据流图 (`12-vision-relations.svg`)](./diagrams/12-vision-relations.svg) ｜ [Graphviz 组织图源](./diagrams/12-vision-organization.dot) ｜ [Graphviz 关系图源](./diagrams/12-vision-relations.dot)
- **直接涉及源码范围**：`crates/alia-vision-pipeline/src/use-chat-vision.rs`, `crates/alia-vision-pipeline/src/store.rs`, `crates/alia-vision-pipeline/src/use-vision-inference.rs`, `crates/alia-vision-pipeline/src/vision-read-queue.rs`, `crates/alia-vision-pipeline/src/use-vision-workloads.rs`, `crates/alia-vision-pipeline/src/orchestrator.rs`, `crates/alia-vision-pipeline/src/processing-store.rs`, `crates/alia-vision-pipeline/src/activity.rs`

#### 读图与工程边界守则
- 图片直接进入支持图像的聊天模型，或由选定 Vision 模型描述；两者是配置与能力条件下的分支。
- VisionReadQueue 是每个 renderer 窗口内的队列，不是全应用统一并发闸门。
- vision:workload:sourceId 提供稳定 contextId；replace-self 更新该来源的上下文，而非追加永久记忆。

#### 节点与状态归属登记表（共 16 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `vision_chatvision` | 聊天图片 | **useChatVision** | `useChatVision` | `crates/alia-vision-pipeline/src/use-chat-vision.rs:26` | 统一聊天图片路由判断 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `vision_selection` | 聊天图片 | **selectionOf(sessionId)** | `selectionOf` | `crates/alia-vision-pipeline/src/use-chat-vision.rs:42` | 从会话角色 getModules 得到 chat/vision 选择 | 无角色时读全局默认 | Rust 核心实现 |
| `vision_capability` | 聊天图片 | **declaresImageInput** | `declaresImageInput` | `crates/alia-vision-pipeline/src/use-chat-vision.rs:59` | 查模型目录 abilities.vision | 目录缺省不等于证实不能看图 | Rust 核心实现 |
| `vision_attachments` | 聊天图片 | **readsAttachedImages** | `readsAttachedImages` | `crates/alia-vision-pipeline/src/use-chat-vision.rs:68` | 是否让 Vision 先读用户附件 | useForChat 开关 | Rust 核心实现 |
| `vision_toolimages` | 聊天图片 | **toolImageReader** | `toolImageReader` | `crates/alia-vision-pipeline/src/use-chat-vision.rs:76` | 给工具截图投影提供可选描述器 | useForToolImages 开关 | Rust 核心实现 |
| `vision_settings` | 推理服务 | **Vision Store** | `useVisionStore` | `crates/alia-vision-pipeline/src/store.rs:10` | 保存全局视觉服务选择和启用配置 | activeProvider / activeModel | Rust 核心实现 |
| `vision_inference` | 推理服务 | **runVisionInference** | `runVisionInference` | `crates/alia-vision-pipeline/src/use-vision-inference.rs:115` | 统一视觉调用入口及结果状态 | lastText | Rust 核心实现 |
| `vision_read` | 推理服务 | **readImage / describeImage** | `describeImage / readImage` | `crates/alia-vision-pipeline/src/use-vision-inference.rs:57` | 解析图像、选择 provider、发起推理 | 调用超时与 AbortSignal | Rust 核心实现 |
| `vision_queue` | 推理服务 | **VisionReadQueue** | `VisionReadQueue` | `crates/alia-vision-pipeline/src/vision-read-queue.rs:48` | 按 provider id 分配本窗口 semaphore | slotsByProvider | Rust 核心实现 |
| `vision_workload` | 屏幕观察 | **Vision workload** | `getVisionWorkload` | `crates/alia-vision-pipeline/src/use-vision-workloads.rs:1` | 选择 screen:interpret / tool:image 等任务说明 | 任务提示 | Rust 核心实现 |
| `vision_capture` | 屏幕观察 | **VisionCapturePayload** | `VisionCapturePayload` | `crates/alia-vision-pipeline/src/orchestrator.rs:18` | 一次屏幕帧的输入契约 | imageDataUrl / sourceId / workloadId | Rust 核心实现 |
| `vision_ticker` | 屏幕观察 | **VisionProcessing Store** | `useVisionProcessingStore` | `crates/alia-vision-pipeline/src/processing-store.rs:40` | 管理采集 tick、处理占用和指标 | processing / captureIntervalMs | Rust 核心实现 |
| `vision_orchestrator` | 屏幕观察 | **Vision Orchestrator** | `useVisionOrchestratorStore` | `crates/alia-vision-pipeline/src/orchestrator.rs:50` | 编排截图推理和可选上下文发布 | lastResultText / lastResultAt / lastError | Rust 核心实现 |
| `vision_contextid` | 上下文与指标 | **getVisionContextId** | `getVisionContextId` | `crates/alia-vision-pipeline/src/orchestrator.rs:31` | 按 workload 与 source 构造稳定来源标识 | vision:{workload}:{source} | Rust 核心实现 |
| `vision_channel` | 上下文与指标 | **sendContextUpdate** | `modsServerChannelStore.sendContextUpdate` | `crates/alia-vision-pipeline/src/orchestrator.rs:92` | 发布文本+图片+元数据到模块上下文通道 | ReplaceSelf | Rust 核心实现 |
| `vision_activity` | 上下文与指标 | **Vision Activity Store** | `useVisionActivityStore` | `crates/alia-vision-pipeline/src/activity.rs:53` | 同步记录采集/推理活动指标 | 记录是运行活动，不自动为角色记忆 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 17 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `vision_chatvision` (useChatVision) | `vision_selection` (selectionOf(sessionId)) | 调用 (`call`) | 解析 |
| `vision_selection` (selectionOf(sessionId)) | `vision_capability` (declaresImageInput) | 契约 / 配置 (`contract`) | 聊天模型能力 |
| `vision_capability` (declaresImageInput) | `vision_attachments` (readsAttachedImages) | 数据流 (`data`) | 决定附件分支 |
| `vision_capability` (declaresImageInput) | `vision_toolimages` (toolImageReader) | 数据流 (`data`) | 决定工具图片分支 |
| `vision_attachments` (readsAttachedImages) | `vision_inference` (runVisionInference) | 调用 (`call`) | 需要描述时调用 |
| `vision_toolimages` (toolImageReader) | `vision_inference` (runVisionInference) | 调用 (`call`) | tool:image |
| `vision_settings` (Vision Store) | `vision_inference` (runVisionInference) | 契约 / 配置 (`contract`) | 服务选择 |
| `vision_inference` (runVisionInference) | `vision_queue` (VisionReadQueue) | 调用 (`call`) | 申请并发槽位 |
| `vision_inference` (runVisionInference) | `vision_read` (readImage / describeImage) | 调用 (`call`) | 获得槽位后调用 |
| `vision_workload` (Vision workload) | `vision_read` (readImage / describeImage) | 契约 / 配置 (`contract`) | 提示任务 |
| `vision_ticker` (VisionProcessing Store) | `vision_capture` (VisionCapturePayload) | 数据流 (`data`) | 宿主 handler 采集 |
| `vision_capture` (VisionCapturePayload) | `vision_orchestrator` (Vision Orchestrator) | 数据流 (`data`) | processCapture |
| `vision_orchestrator` (Vision Orchestrator) | `vision_inference` (runVisionInference) | 调用 (`call`) | 发起图片推理 |
| `vision_orchestrator` (Vision Orchestrator) | `vision_contextid` (getVisionContextId) | 调用 (`call`) | 确定来源 |
| `vision_contextid` (getVisionContextId) | `vision_channel` (sendContextUpdate) | 数据流 (`data`) | contextId |
| `vision_orchestrator` (Vision Orchestrator) | `vision_channel` (sendContextUpdate) | 事件 (`event`) | publishContext 为真时 |
| `vision_inference` (runVisionInference) | `vision_activity` (Vision Activity Store) | 事件 (`event`) | 记录推理活动 |

---

### 专题 13：语音宿主、控制总线与设备归属 (`voice_host`)

> **专题定位**：Stage.vue 装配播放设备与 Voice Store；其他界面通过命令/快照交互。PCM 留在音频宿主。

- **配套矢量图纸**：[职责组织图 (`13-voice-host-organization.svg`)](./diagrams/13-voice-host-organization.svg) ｜ [调用与数据流图 (`13-voice-host-relations.svg`)](./diagrams/13-voice-host-relations.svg) ｜ [Graphviz 组织图源](./diagrams/13-voice-host-organization.dot) ｜ [Graphviz 关系图源](./diagrams/13-voice-host-relations.dot)
- **直接涉及源码范围**：`apps/webui/src/components/sandbox/SandboxHost.vue`, `packages/stage-ui/src/stores/voice.ts`, `packages/stage-ui/src/composables/audio/voice-controller.ts`, `packages/stage-ui/src/stores/settings/audio-device.ts`, `packages/stage-ui/src/composables/audio/audio-device.ts`, `packages/audio-web/src/browser/sources.ts`, `crates/alia-audio-pipeline/src/audio-input.rs`, `packages/stage-ui/src/stores/modules/hearing.ts`, `packages/stage-ui/src/libs/audio/hearing-transcriber.ts`, `packages/stage-ui/src/composables/voice-drafts.ts`, `packages/stage-ui/src/stores/chat.ts`, `packages/stage-ui/src/services/speech/bus.ts`, `packages/stage-ui/src/services/speech/speech-client.ts`, `packages/stage-ui/src/stores/voice-messages.ts`, `packages/stage-ui/src/stores/voice-capture-lease.ts`, `packages/stage-ui/src/stores/audio.ts`

#### 读图与工程边界守则
- AudioInput 使用订阅计数共享一条音源；首个订阅打开，最后一个退出关闭。VoiceController 不拥有关闭它的权力。
- SpeechClient 每次创建独立 producerId/originId，同一 TurnRef 的生产者由宿主有序管理。
- manual/composer/call/voice-message 是不同的产品工作流，不能等同于一个自动提交的语音输入。
- capture lease 用于其他窗口录音/声纹登记期间抑制主机自动输入；这与 AudioInput 的历史保留 lease 不是同一个概念。

#### 节点与状态归属登记表（共 20 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `voice_host_stage` | 宿主装配 | **WebUI 语音宿主** | `Playback / connectOutput` | `apps/webui/src/components/sandbox/SandboxHost.vue:397` | 创建浏览器播放及回调；连接一次语音输出 | disconnectVoiceOutput | WebUI 面板前端 |
| `voice_host_voice` | 宿主装配 | **Voice Store** | `useVoiceStore` | `packages/stage-ui/src/stores/voice.ts:57` | 连接语音核心、UI快照、草稿、模型与命令 | activeTurns / responses / inputRequests | WebUI 面板前端 |
| `voice_host_controller` | 宿主装配 | **useVoiceController** | `useVoiceController` | `packages/stage-ui/src/composables/audio/voice-controller.ts:16` | 创建核心控制器并投影其状态到 Vue | state / snapshot / error | WebUI 面板前端 |
| `voice_host_settings` | 设备与PCM | **麦克风设置** | `useSettingsAudioDevice` | `packages/stage-ui/src/stores/settings/audio-device.ts:29` | 保存设备偏好并控制保持打开 | selectedAudioInput / enabled / off | WebUI 面板前端 |
| `voice_host_device` | 设备与PCM | **useAudioDevice** | `useAudioDevice` | `packages/stage-ui/src/composables/audio/audio-device.ts:77` | 创建受开关保护的设备源与共享输入 | input / unprocessedInput / permission | WebUI 面板前端 |
| `voice_host_source` | 设备与PCM | **microphoneSource** | `microphoneSource` | `packages/audio-web/src/browser/sources.ts:146` | 浏览器 getUserMedia/AudioWorklet 适配 | MediaStream / echoCancellation | WebUI 面板前端 |
| `voice_host_input` | 设备与PCM | **AudioInput** | `AudioInput` | `crates/alia-audio-pipeline/src/audio-input.rs:131` | 共享PCM与有限历史；订阅各自取消 | connection / subscribers / history | Rust 核心实现 |
| `voice_host_hearing` | 服务与提交 | **Hearing Store** | `useHearingStore` | `packages/stage-ui/src/stores/modules/hearing.ts:313` | 转写配置、输入模式及 provider 适配 | inputMode / autoSend / rephrase / endJudge | WebUI 面板前端 |
| `voice_host_transcriber` | 服务与提交 | **createHearingTranscriber** | `createHearingTranscriber` | `packages/stage-ui/src/libs/audio/hearing-transcriber.ts:28` | 把 provider 流转为统一完整转写修订 | 每次请求的分句/分词器 | WebUI 面板前端 |
| `voice_host_drafts` | 服务与提交 | **useVoiceDrafts** | `useVoiceDrafts` | `packages/stage-ui/src/composables/voice-drafts.ts:18` | 接收最终SpeechSubmission；草稿或提交 | drafts / frontDraftId | WebUI 面板前端 |
| `voice_host_chat` | 服务与提交 | **Chat Store** | `useChatStore` | `packages/stage-ui/src/stores/chat.ts:1` | 接收语音文本与打断回执 | 持久会话的宿主端口 | WebUI 面板前端 |
| `voice_host_bus` | 命令与快照 | **Speech / Voice Eventa Bus** | `getSpeechBusContext` | `packages/stage-ui/src/services/speech/bus.ts:46` | 跨界面命令与快照通道 | 广播通道 context | WebUI 面板前端 |
| `voice_host_client` | 命令与快照 | **SpeechClient** | `SpeechClient` | `packages/stage-ui/src/services/speech/speech-client.ts:11` | 跨renderer有序发送生产者命令 | originId / producerId / turn | WebUI 面板前端 |
| `voice_host_inputcommands` | 命令与快照 | **voiceInputCommand** | `voiceInputCommand` | `packages/stage-ui/src/services/speech/bus.ts:141` | begin/end/cancel/草稿/通话等请求 | requestId 绑定输入 | WebUI 面板前端 |
| `voice_host_snapshot` | 命令与快照 | **VoiceHostSnapshot** | `VoiceHostSnapshot` | `packages/stage-ui/src/services/speech/bus.ts:95` | 向界面同步连接、输入、草稿和通话状态 | 快照不含活PCM | WebUI 面板前端 |
| `voice_host_messages` | 旁路工作流 | **VoiceMessages Store** | `useVoiceMessagesStore` | `packages/stage-ui/src/stores/voice-messages.ts:33` | 语音附件录制/草稿工作流 | 附件草稿由宿主拥有 | WebUI 面板前端 |
| `voice_host_lease` | 旁路工作流 | **VoiceCaptureLease Store** | `useVoiceCaptureLeaseStore` | `packages/stage-ui/src/stores/voice-capture-lease.ts:12` | 外部录制时抑制主语音输入 | held | WebUI 面板前端 |
| `voice_host_listening` | 旁路工作流 | **startListening** | `startListening` | `packages/stage-ui/src/stores/voice.ts:576` | 安装VAD、唤醒词、说话人证据插件 | listening handles | WebUI 面板前端 |
| `voice_host_speaking` | 输出状态 | **Speaking Store** | `useSpeakingStore` | `packages/stage-ui/src/stores/audio.ts:76` | 向动作和UI投影播放状态 | nowSpeaking / mouthOpenSize | WebUI 面板前端 |
| `voice_host_playback` | 输出状态 | **Playback + BrowserPlayback** | `new Playback(new BrowserPlayback(...))` | `apps/webui/src/components/sandbox/SandboxHost.vue:397` | 平台播放时钟、声道与输出图 | speechDestination gain | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 23 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `voice_host_stage` (WebUI 语音宿主) | `voice_host_voice` (Voice Store) | 调用 (`call`) | connectOutput |
| `voice_host_stage` (WebUI 语音宿主) | `voice_host_playback` (Playback + BrowserPlayback) | 持有 / 归属 (`owns`) | 创建 |
| `voice_host_voice` (Voice Store) | `voice_host_controller` (useVoiceController) | 调用 (`call`) | 装配 transcribe/submit/speech ports |
| `voice_host_settings` (麦克风设置) | `voice_host_device` (useAudioDevice) | 调用 (`call`) | 创建设备 |
| `voice_host_device` (useAudioDevice) | `voice_host_source` (microphoneSource) | 调用 (`call`) | 构造源 |
| `voice_host_device` (useAudioDevice) | `voice_host_input` (AudioInput) | 持有 / 归属 (`owns`) | 构造共享输入 |
| `voice_host_source` (microphoneSource) | `voice_host_input` (AudioInput) | 数据流 (`data`) | PCM blocks |
| `voice_host_input` (AudioInput) | `voice_host_controller` (useVoiceController) | 数据流 (`data`) | 注入 audio |
| `voice_host_hearing` (Hearing Store) | `voice_host_transcriber` (createHearingTranscriber) | 调用 (`call`) | createTranscriber |
| `voice_host_transcriber` (createHearingTranscriber) | `voice_host_controller` (useVoiceController) | 数据流 (`data`) | 统一转写事件 |
| `voice_host_controller` (useVoiceController) | `voice_host_drafts` (useVoiceDrafts) | 调用 (`call`) | submit |
| `voice_host_drafts` (useVoiceDrafts) | `voice_host_chat` (Chat Store) | 调用 (`call`) | 提交会话 |
| `voice_host_voice` (Voice Store) | `voice_host_bus` (Speech / Voice Eventa Bus) | 事件 (`event`) | 注册invoke handlers |
| `voice_host_client` (SpeechClient) | `voice_host_bus` (Speech / Voice Eventa Bus) | 事件 (`event`) | open/text/special/end/finish/cancel |
| `voice_host_inputcommands` (voiceInputCommand) | `voice_host_bus` (Speech / Voice Eventa Bus) | 事件 (`event`) | 输入命令 |
| `voice_host_bus` (Speech / Voice Eventa Bus) | `voice_host_voice` (Voice Store) | 调用 (`call`) | 执行宿主操作 |
| `voice_host_voice` (Voice Store) | `voice_host_snapshot` (VoiceHostSnapshot) | 事件 (`event`) | publishSnapshot |
| `voice_host_voice` (Voice Store) | `voice_host_messages` (VoiceMessages Store) | 调用 (`call`) | connect |
| `voice_host_voice` (Voice Store) | `voice_host_lease` (VoiceCaptureLease Store) | 调用 (`call`) | connect |
| `voice_host_lease` (VoiceCaptureLease Store) | `voice_host_controller` (useVoiceController) | 生命周期 (`lifecycle`) | held时取消活动输入 |
| `voice_host_voice` (Voice Store) | `voice_host_listening` (startListening) | 调用 (`call`) | 启动连续听取 |
| `voice_host_listening` (startListening) | `voice_host_input` (AudioInput) | 数据流 (`data`) | 窗口观察 |
| `voice_host_playback` (Playback + BrowserPlayback) | `voice_host_speaking` (Speaking Store) | 事件 (`event`) | 播放开始/结束 |

---

### 专题 14：VoiceController内部：输入文档、插件与决策规则 (`voice_input`)

> **专题定位**：检测器产生证据，规则作决定，控制器执行命令；一个输入经过pending、capturing、finalizing、settled。

- **配套矢量图纸**：[职责组织图 (`14-voice-input-organization.svg`)](./diagrams/14-voice-input-organization.svg) ｜ [调用与数据流图 (`14-voice-input-relations.svg`)](./diagrams/14-voice-input-relations.svg) ｜ [Graphviz 组织图源](./diagrams/14-voice-input-organization.dot) ｜ [Graphviz 关系图源](./diagrams/14-voice-input-relations.dot)
- **直接涉及源码范围**：`crates/alia-core-agent/src/voice/controller.rs`, `crates/alia-core-agent/src/voice/input/types.rs`, `crates/alia-core-agent/src/voice/input/attempt.rs`, `crates/alia-audio-pipeline/src/capture.rs`, `crates/alia-core-agent/src/voice/input/speech-input.rs`, `crates/alia-core-agent/src/voice/input/ports.rs`, `crates/alia-core-agent/src/voice/plugins/registry.rs`, `crates/alia-core-agent/src/voice/plugins/types.rs`, `packages/stage-ui/src/libs/voice/voice-rephrase-plugin.ts`, `packages/stage-ui/src/libs/voice/voice-activity-plugin.ts`, `packages/stage-ui/src/libs/voice/speaker-evidence-plugin.ts`, `packages/stage-ui/src/libs/voice/voice-end-judge-plugin.ts`, `crates/alia-core-agent/src/voice/policy/events.rs`, `crates/alia-core-agent/src/voice/policy/engine.rs`, `crates/alia-core-agent/src/voice/policy/rules.rs`, `crates/alia-core-agent/src/voice/policy/types.rs`

#### 读图与工程边界守则
- end()停止PCM捕获，转写输出还须结束；finalizing已释放输入准入，下一句可以开始。
- 插件watch跟随最新文档并可取消；finalize依插件安装和注册顺序串行。
- 慢速结束判断携带readAt，VoicePolicy对照转写/活动/上下文revision拒绝过时证据。
- VoicePlugin grants是受约束调用接口；插件仍是可信进程内代码，不是沙箱。
- manual after-silence先等待目标播放静音；speech-onset先保留pre-roll再等待淡出。
- SpeechInputPorts有conversationContext接口，但当前Stage useVoiceStore构造控制器时未注入；默认上下文是revision=0/messages=[]。图中revision能力不应被误读为Stage已接入完整近期会话。
- 说话人证据进入TranscriptDocument/trace，但voice-drafts.ts明确暂不把该证据写入prompt，等待可靠的speaker gate。

#### 节点与状态归属登记表（共 23 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `voice_input_controller` | 核心协调 | **VoiceController** | `VoiceController` | `crates/alia-core-agent/src/voice/controller.rs:63` | 拥有准入、事件、插件与语音响应的生命周期 | current / attempts / responses | Rust 核心实现 |
| `voice_input_begin` | 核心协调 | **BeginSpeechInput** | `BeginSpeechInput` | `crates/alia-core-agent/src/voice/input/types.rs:7` | 声明session、原因、打断目标和起始策略 | sessionId / interruptTurns / start | Rust 核心实现 |
| `voice_input_attempt` | 输入实例 | **SpeechInputAttempt** | `SpeechInputAttempt` | `crates/alia-core-agent/src/voice/input/attempt.rs:35` | 管理一次输入从准入至提交 | pending→capturing→finalizing→settled | Rust 核心实现 |
| `voice_input_capture` | 输入实例 | **capture(AudioInput)** | `capture` | `crates/alia-audio-pipeline/src/capture.rs:1` | 截取一个连续PCM区间 | started / finish / cancel / done | Rust 核心实现 |
| `voice_input_input` | 输入实例 | **SpeechInput** | `SpeechInput` | `crates/alia-core-agent/src/voice/input/speech-input.rs:1` | 接收转写、修正和说话人证据 | document / transcript | Rust 核心实现 |
| `voice_input_transcript` | 输入实例 | **TranscriptDocument** | `TranscriptDocument` | `crates/alia-core-agent/src/voice/input/types.rs:43` | 版本化原文与修正文档 | raw / corrected / version / speakers | Rust 核心实现 |
| `voice_input_transcriber` | 宿主插口 | **StreamingTranscriber port** | `StreamingTranscriber` | `crates/alia-core-agent/src/voice/input/ports.rs:11` | 消费PCM流，产生转写事件 | AbortSignal | Rust 核心实现 |
| `voice_input_submit` | 宿主插口 | **submit port** | `SpeechInputPorts.submit` | `crates/alia-core-agent/src/voice/input/ports.rs:25` | 最终稿交宿主落为消息或草稿 | committed 或 drafted | Rust 核心实现 |
| `voice_input_submission` | 输入实例 | **SpeechSubmission** | `SpeechSubmission` | `crates/alia-core-agent/src/voice/input/types.rs:65` | 本次最终文本及完整修订来源 | submissionId / transcript / context | Rust 核心实现 |
| `voice_input_plugins` | 插件机制 | **VoicePlugins registry** | `VoicePlugins` | `crates/alia-core-agent/src/voice/plugins/registry.rs:1` | 安装插件并按输入/输出派生scope | grants / handles / cleanup | Rust 核心实现 |
| `voice_input_scope` | 插件机制 | **SpeechInputScope** | `SpeechInputScope` | `crates/alia-core-agent/src/voice/plugins/types.rs:63` | 每个插件对一个输入的限定操作 | inputId / sessionId / signal | Rust 核心实现 |
| `voice_input_watch` | 插件机制 | **watch 最新文档** | `SpeechInputScope.watch` | `crates/alia-core-agent/src/voice/plugins/types.rs:75` | key变化即取消旧任务，可debounce | 推测工作非最终写入 | Rust 核心实现 |
| `voice_input_finalize` | 插件机制 | **finalize 有序处理** | `FinalizeContext` | `crates/alia-core-agent/src/voice/plugins/types.rs:44` | 在完整转写上依次修正/附加上下文 | patch / attach / timeout | Rust 核心实现 |
| `voice_input_rephrase` | 具体插件 | **VoiceRephrase plugin** | `createVoiceRephrasePlugin` | `packages/stage-ui/src/libs/voice/voice-rephrase-plugin.ts:45` | 可选文本修正；由Stage注入模型操作 | watch准备；finalize提交 | WebUI 面板前端 |
| `voice_input_activity` | 具体插件 | **VoiceActivity plugin** | `createVoiceActivityPlugin` | `packages/stage-ui/src/libs/voice/voice-activity-plugin.ts:47` | 运行VAD/唤醒/回声准入并发出信号 | 音频窗口证据 | WebUI 面板前端 |
| `voice_input_speaker` | 具体插件 | **SpeakerEvidence plugin** | `createSpeakerEvidencePlugin` | `packages/stage-ui/src/libs/voice/speaker-evidence-plugin.ts:1` | 测量说话人候选并附入文档 | score是证据，不是身份概率 | WebUI 面板前端 |
| `voice_input_judge` | 具体插件 | **VoiceEndJudge plugin** | `createVoiceEndJudgePlugin` | `packages/stage-ui/src/libs/voice/voice-end-judge-plugin.ts:49` | 停顿后可选调用模型判断是否说完 | complete / incomplete + readAt | WebUI 面板前端 |
| `voice_input_events` | 规则引擎 | **生命周期与信号** | `inputBegunEvent / speechActivitySignal` | `crates/alia-core-agent/src/voice/policy/events.rs:11` | 同一总线供规则和UI/trace投影订阅 | begun/capturing/transcript/finalizing/settled | Rust 核心实现 |
| `voice_input_policy` | 规则引擎 | **VoicePolicy** | `VoicePolicy` | `crates/alia-core-agent/src/voice/policy/engine.rs:67` | 有序消费事件、更新revision、执行规则 | rules / inputs / decided / queue | Rust 核心实现 |
| `voice_input_rules` | 规则引擎 | **三类内置规则** | `createSpeechActivityRule / createPauseRule / createEndJudgementRule` | `crates/alia-core-agent/src/voice/policy/rules.rs:1` | 由语音活动、停顿、结束判断决定下一步 | 纯 reducer | Rust 核心实现 |
| `voice_input_revision` | 规则引擎 | **InputRevisions** | `InputRevisions` | `crates/alia-core-agent/src/voice/policy/types.rs:14` | 按input与证据种类检查新旧 | transcriptText / context / signal revisions | Rust 核心实现 |
| `voice_input_commands` | 规则引擎 | **VoiceCommand** | `VoiceCommand` | `crates/alia-core-agent/src/voice/policy/types.rs:25` | 核心定义的可执行操作 | begin / seal / cancel / interrupt | Rust 核心实现 |
| `voice_input_outcome` | 输入实例 | **AttemptOutcome** | `SpeechInputAttemptOutcome` | `crates/alia-core-agent/src/voice/input/types.rs:16` | 成功、草稿、取消、失败有明确区分 | 失败 stage: source/admission/capture/transcription/submission | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 28 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `voice_input_begin` (BeginSpeechInput) | `voice_input_controller` (VoiceController) | 数据流 (`data`) | beginInput |
| `voice_input_controller` (VoiceController) | `voice_input_attempt` (SpeechInputAttempt) | 持有 / 归属 (`owns`) | 创建与记录 |
| `voice_input_attempt` (SpeechInputAttempt) | `voice_input_capture` (capture(AudioInput)) | 调用 (`call`) | 申请捕获区间 |
| `voice_input_capture` (capture(AudioInput)) | `voice_input_transcriber` (StreamingTranscriber port) | 数据流 (`data`) | PCM流 |
| `voice_input_transcriber` (StreamingTranscriber port) | `voice_input_input` (SpeechInput) | 数据流 (`data`) | TranscriptionEvent |
| `voice_input_attempt` (SpeechInputAttempt) | `voice_input_input` (SpeechInput) | 持有 / 归属 (`owns`) | 创建 |
| `voice_input_input` (SpeechInput) | `voice_input_transcript` (TranscriptDocument) | 持有 / 归属 (`owns`) | 生成修订 |
| `voice_input_controller` (VoiceController) | `voice_input_plugins` (VoicePlugins registry) | 持有 / 归属 (`owns`) | 拥有registry |
| `voice_input_plugins` (VoicePlugins registry) | `voice_input_scope` (SpeechInputScope) | 持有 / 归属 (`owns`) | 每输入创建 |
| `voice_input_scope` (SpeechInputScope) | `voice_input_watch` (watch 最新文档) | 调用 (`call`) | 注册 |
| `voice_input_scope` (SpeechInputScope) | `voice_input_finalize` (finalize 有序处理) | 调用 (`call`) | 注册 |
| `voice_input_transcript` (TranscriptDocument) | `voice_input_watch` (watch 最新文档) | 事件 (`event`) | 新版本取消旧工作 |
| `voice_input_transcript` (TranscriptDocument) | `voice_input_finalize` (finalize 有序处理) | 数据流 (`data`) | 完整文档 |
| `voice_input_rephrase` (VoiceRephrase plugin) | `voice_input_scope` (SpeechInputScope) | 调用 (`call`) | onInput |
| `voice_input_activity` (VoiceActivity plugin) | `voice_input_events` (生命周期与信号) | 事件 (`event`) | speechActivitySignal |
| `voice_input_speaker` (SpeakerEvidence plugin) | `voice_input_input` (SpeechInput) | 数据流 (`data`) | updateSpeakerEvidence |
| `voice_input_judge` (VoiceEndJudge plugin) | `voice_input_events` (生命周期与信号) | 事件 (`event`) | endJudgedSignal |
| `voice_input_attempt` (SpeechInputAttempt) | `voice_input_events` (生命周期与信号) | 事件 (`event`) | 生命周期变化 |
| `voice_input_events` (生命周期与信号) | `voice_input_policy` (VoicePolicy) | 事件 (`event`) | 有序消费 |
| `voice_input_policy` (VoicePolicy) | `voice_input_rules` (三类内置规则) | 调用 (`call`) | reduce |
| `voice_input_policy` (VoicePolicy) | `voice_input_revision` (InputRevisions) | 持有 / 归属 (`owns`) | 记录与比较 |
| `voice_input_rules` (三类内置规则) | `voice_input_commands` (VoiceCommand) | 数据流 (`data`) | 产生命令 |
| `voice_input_commands` (VoiceCommand) | `voice_input_controller` (VoiceController) | 调用 (`call`) | execute |
| `voice_input_controller` (VoiceController) | `voice_input_attempt` (SpeechInputAttempt) | 调用 (`call`) | end/cancel |
| `voice_input_finalize` (finalize 有序处理) | `voice_input_submission` (SpeechSubmission) | 数据流 (`data`) | 修正文本/插件上下文 |
| `voice_input_submission` (SpeechSubmission) | `voice_input_submit` (submit port) | 调用 (`call`) | 宿主提交 |
| `voice_input_submit` (submit port) | `voice_input_outcome` (AttemptOutcome) | 数据流 (`data`) | 最终结果 |
| `voice_input_attempt` (SpeechInputAttempt) | `voice_input_outcome` (AttemptOutcome) | 生命周期 (`lifecycle`) | settle |

---

### 专题 15：语音输出：生产者、合成、播放与打断 (`voice_output`)

> **专题定位**：一个 TurnRef 对应一个 VoiceResponse；响应内可开多个有序 SpeechStream，每个响应拥有独立 PlaybackGroup。

- **配套矢量图纸**：[职责组织图 (`15-voice-output-organization.svg`)](./diagrams/15-voice-output-organization.svg) ｜ [调用与数据流图 (`15-voice-output-relations.svg`)](./diagrams/15-voice-output-relations.svg) ｜ [Graphviz 组织图源](./diagrams/15-voice-output-organization.dot) ｜ [Graphviz 关系图源](./diagrams/15-voice-output-relations.dot)
- **直接涉及源码范围**：`crates/alia-core-agent/src/voice/controller.rs`, `crates/alia-core-agent/src/voice/turn.rs`, `crates/alia-core-agent/src/voice/output/response.rs`, `packages/stage-ui/src/stores/modules/speech.ts`, `crates/alia-core-agent/src/voice/output/speech-transformation/segmenter.rs`, `crates/alia-core-agent/src/voice/output/speech-transformation/index.rs`, `crates/alia-audio-pipeline/src/speech-pipeline.rs`, `crates/alia-audio-pipeline/src/processors/tts-chunker.rs`, `crates/alia-audio-pipeline/src/playback.rs`, `packages/audio-web/src/browser/playback.ts`, `crates/alia-core-agent/src/voice/plugins/types.rs`, `apps/webui/src/components/sandbox/SandboxHost.vue`

#### 读图与工程边界守则
- 普通TTS走分段合成；双向流式TTS可并行消费文本并输出音频，不应画成同一条固定分句流水线。
- finish()等待全部producer和实际播放排空；end()只封闭一个producer。
- 打断先关闭目标response的接纳、淡出该playback group，再取消其所有producer；不等于停止所有音频。
- Interruption.silenced是硬件播放静音完成；Interruption.done是控制事件持久化回执，两者不同。
- 分段TTS的特殊标记按前序音频顺序到达；流式TTS无分段，对应标记在文本写入时到达。

#### 节点与状态归属登记表（共 22 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `voice_output_controller` | 响应生命周期 | **VoiceController.openResponse** | `openResponse` | `crates/alia-core-agent/src/voice/controller.rs:252` | 按TurnRef创建/复用响应；禁止重开已结束turn | responses / usedTurns | Rust 核心实现 |
| `voice_output_turn` | 响应生命周期 | **TurnRef** | `TurnRef` | `crates/alia-core-agent/src/voice/turn.rs:1` | 会话与回合身份 | sessionId / turnId | Rust 核心实现 |
| `voice_output_response` | 响应生命周期 | **VoiceResponse** | `VoiceResponse` | `crates/alia-core-agent/src/voice/output/response.rs:468` | 拥有生产者顺序、播放组与关闭信号 | streams / ordered / sealed / outcome | Rust 核心实现 |
| `voice_output_stream` | 响应生命周期 | **SpeechStream** | `SpeechStream` | `crates/alia-core-agent/src/voice/output/response.rs:112` | 单个文本生产者；write/special/flush/end/cancel | purpose / abort / done | Rust 核心实现 |
| `voice_output_output` | 宿主输出端口 | **SpeechOutput port** | `SpeechOutput` | `crates/alia-core-agent/src/voice/output/response.rs:39` | 注入TTS及播放实现 | synthesize 或 stream；playback | Rust 核心实现 |
| `voice_output_factory` | 宿主输出端口 | **SpeechStore.createOutput** | `createOutput` | `packages/stage-ui/src/stores/modules/speech.ts:686` | 快照角色 provider/model/voice 后选TTS路径 | 固定本响应服务配置 | WebUI 面板前端 |
| `voice_output_segmenter` | 文本与合成 | **MarkdownSegmenter** | `MarkdownSegmenter` | `crates/alia-core-agent/src/voice/output/speech-transformation/segmenter.rs:53` | 在文本流中识别稳定Markdown片段 | 保留尚未完成的片段 | Rust 核心实现 |
| `voice_output_speechtext` | 文本与合成 | **toSpeechText** | `toSpeechText` | `crates/alia-core-agent/src/voice/output/speech-transformation/index.rs:83` | 将Markdown等文本转为可朗读文字 | 公式/金额/路径等预处理 | Rust 核心实现 |
| `voice_output_pipeline` | 文本与合成 | **createSpeechPipeline** | `createSpeechPipeline` | `crates/alia-audio-pipeline/src/speech-pipeline.rs:65` | 普通TTS的intent、分块、合成、播放编排 | IntentHandle / TtsRequest | Rust 核心实现 |
| `voice_output_chunker` | 文本与合成 | **chunkTtsInput** | `chunkTtsInput` | `crates/alia-audio-pipeline/src/processors/tts-chunker.rs:71` | 按边界产生TTS请求及特殊控制标记 | chunk reason / flush | Rust 核心实现 |
| `voice_output_streamtts` | 文本与合成 | **流式TTS适配** | `streamSpeech branch` | `packages/stage-ui/src/stores/modules/speech.ts:710` | 文本流和音频分片并行 | SpeechAudio stream | WebUI 面板前端 |
| `voice_output_playback` | 播放资源 | **Playback** | `Playback` | `crates/alia-audio-pipeline/src/playback.rs:237` | 创建并行独立播放组，暴露音频时钟 | driver / output tap | Rust 核心实现 |
| `voice_output_group` | 播放资源 | **PlaybackGroup** | `PlaybackGroup` | `crates/alia-audio-pipeline/src/playback.rs:67` | 组内顺序播片段；finish排空、stop淡出 | queue / active clip / played receipts | Rust 核心实现 |
| `voice_output_driver` | 播放资源 | **BrowserPlayback** | `BrowserPlayback` | `packages/audio-web/src/browser/playback.ts:13` | 浏览器音频图与播放时钟适配 | AudioContext | WebUI 面板前端 |
| `voice_output_receipt` | 播放资源 | **PlaybackReceipt** | `PlaybackReceipt` | `crates/alia-audio-pipeline/src/playback.rs:20` | 记录实际播放进度与区间 | groupId / clipId / throughMs / interval | Rust 核心实现 |
| `voice_output_events` | 观察与表达 | **SpeechOutputEvent** | `SpeechOutputEvent` | `crates/alia-core-agent/src/voice/output/response.rs:67` | 关联合成与播放进程 | tts-request/result；playback-start/end；token-special | Rust 核心实现 |
| `voice_output_outputplugin` | 观察与表达 | **SpeechOutputScope** | `SpeechOutputScope` | `crates/alia-core-agent/src/voice/plugins/types.rs:81` | 按响应订阅输出事件并有限控制 | turn / signal / on | Rust 核心实现 |
| `voice_output_special` | 观察与表达 | **onSpecial → 表达信号分发** | `onSpecial / playSpecialToken` | `apps/webui/src/components/sandbox/SandboxHost.vue:430` | 把ACT/DELAY/CALL交给流控制系统 | turnId | WebUI 面板前端 |
| `voice_output_interrupt` | 打断闭环 | **VoiceController.interrupt** | `interrupt` | `crates/alia-core-agent/src/voice/controller.rs:271` | 对明确turn目标生成打断记录 | eventId / cause / unique turn keys | Rust 核心实现 |
| `voice_output_silenced` | 打断闭环 | **Interruption.silenced** | `silenced` | `crates/alia-core-agent/src/voice/controller.rs:298` | 等待目标播放组静音 | Promise<per-turn receipts> | Rust 核心实现 |
| `voice_output_record` | 打断闭环 | **recordInterruption port** | `notifyInterruption` | `crates/alia-core-agent/src/voice/controller.rs:308` | 静音后向会话宿主持久化打断事件 | acknowledged / queued / failed | Rust 核心实现 |
| `voice_output_done` | 打断闭环 | **Interruption.done** | `done` | `crates/alia-core-agent/src/voice/controller.rs:300` | 独立确认控制事件记录结果 | 失败重试复用eventId | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 25 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `voice_output_turn` (TurnRef) | `voice_output_controller` (VoiceController.openResponse) | 数据流 (`data`) | 以turnKey寻址 |
| `voice_output_controller` (VoiceController.openResponse) | `voice_output_response` (VoiceResponse) | 持有 / 归属 (`owns`) | 拥有 |
| `voice_output_response` (VoiceResponse) | `voice_output_group` (PlaybackGroup) | 持有 / 归属 (`owns`) | openGroup |
| `voice_output_response` (VoiceResponse) | `voice_output_stream` (SpeechStream) | 持有 / 归属 (`owns`) | openSpeech；按创建顺序 |
| `voice_output_factory` (SpeechStore.createOutput) | `voice_output_output` (SpeechOutput port) | 调用 (`call`) | 构造 |
| `voice_output_output` (SpeechOutput port) | `voice_output_response` (VoiceResponse) | 契约 / 配置 (`contract`) | 注入 |
| `voice_output_stream` (SpeechStream) | `voice_output_segmenter` (MarkdownSegmenter) | 数据流 (`data`) | 文本输入 |
| `voice_output_segmenter` (MarkdownSegmenter) | `voice_output_speechtext` (toSpeechText) | 数据流 (`data`) | 稳定片段转朗读 |
| `voice_output_speechtext` (toSpeechText) | `voice_output_pipeline` (createSpeechPipeline) | 数据流 (`data`) | 普通TTS分支 |
| `voice_output_pipeline` (createSpeechPipeline) | `voice_output_chunker` (chunkTtsInput) | 调用 (`call`) | 分段 |
| `voice_output_output` (SpeechOutput port) | `voice_output_pipeline` (createSpeechPipeline) | 契约 / 配置 (`contract`) | synthesize适配 |
| `voice_output_speechtext` (toSpeechText) | `voice_output_streamtts` (流式TTS适配) | 数据流 (`data`) | 流式TTS分支 |
| `voice_output_pipeline` (createSpeechPipeline) | `voice_output_group` (PlaybackGroup) | 数据流 (`data`) | 有序音频片段 |
| `voice_output_streamtts` (流式TTS适配) | `voice_output_group` (PlaybackGroup) | 数据流 (`data`) | 音频流分片 |
| `voice_output_playback` (Playback) | `voice_output_group` (PlaybackGroup) | 持有 / 归属 (`owns`) | 创建独立组 |
| `voice_output_playback` (Playback) | `voice_output_driver` (BrowserPlayback) | 调用 (`call`) | 驱动播放 |
| `voice_output_group` (PlaybackGroup) | `voice_output_receipt` (PlaybackReceipt) | 数据流 (`data`) | 报告实际播放 |
| `voice_output_response` (VoiceResponse) | `voice_output_events` (SpeechOutputEvent) | 事件 (`event`) | emit |
| `voice_output_events` (SpeechOutputEvent) | `voice_output_outputplugin` (SpeechOutputScope) | 事件 (`event`) | 订阅 |
| `voice_output_events` (SpeechOutputEvent) | `voice_output_special` (onSpecial → 表达信号分发) | 事件 (`event`) | token-special |
| `voice_output_interrupt` (VoiceController.interrupt) | `voice_output_response` (VoiceResponse) | 调用 (`call`) | interrupt(cause,fadeMs) |
| `voice_output_response` (VoiceResponse) | `voice_output_group` (PlaybackGroup) | 生命周期 (`lifecycle`) | stop + cancel producers |
| `voice_output_receipt` (PlaybackReceipt) | `voice_output_silenced` (Interruption.silenced) | 数据流 (`data`) | 静音回执 |
| `voice_output_silenced` (Interruption.silenced) | `voice_output_record` (recordInterruption port) | 调用 (`call`) | 等待后记录 |
| `voice_output_record` (recordInterruption port) | `voice_output_done` (Interruption.done) | 数据流 (`data`) | 持久化确认 |

---

### 专题 16：流控制、内联动作标记解析与情绪语义信号分发 (`expression`)

> **专题定位**：保留原版流控制与语义情绪标记解析核心能力：从语音/文本流中识别特殊标记，经 StreamingControl 与 createStreamingControlParser 解析、normalizeActPayload 归一化为标准 EmotionPayload 信号并推入分发队列；移除原版下半部分全部 17 个 Live2D/VRM/MMD 内部渲染器节点。

- **配套矢量图纸**：[职责组织图 (`16-expression-organization.svg`)](./diagrams/16-expression-organization.svg) ｜ [调用与数据流图 (`16-expression-relations.svg`)](./diagrams/16-expression-relations.svg) ｜ [Graphviz 组织图源](./diagrams/16-expression-organization.dot) ｜ [Graphviz 关系图源](./diagrams/16-expression-relations.dot)
- **直接涉及源码范围**：`apps/webui/src/components/sandbox/SandboxHost.vue`, `packages/stage-ui/src/stores/ai/chat-llm/streaming-control.ts`, `crates/alia-audio-pipeline/src/llm-streaming-control/controller.rs`, `crates/alia-audio-pipeline/src/llm-streaming-control/payloads.rs`, `packages/stage-ui/src/constants/emotions.ts`

#### 读图与工程边界守则
- 核心运行时只负责将模型输出的 <|ACT:...|> 标记解析为标准语义信号（Emotion / EmotionPayload），绝不内置绑定 Live2D 或 VRM 渲染器。
- 归一化后的情绪与动作信号通过 ExpressionBus 向 WebUI 测试沙盒展示，并可通过模块通道转发给任何未来按需挂载的外部表现插件。

#### 节点与状态归属登记表（共 7 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `expression_special` | 流控制入口 | **语音特殊标记** | `playSpecialToken` | `apps/webui/src/components/sandbox/SandboxHost.vue:357` | 带turnId向流控制系统分派 | ACT / DELAY / CALL | WebUI 面板前端 |
| `expression_control` | 流控制入口 | **StreamingControl Store** | `useLlmStreamingControlStore` | `packages/stage-ui/src/stores/ai/chat-llm/streaming-control.ts:23` | 注册/分派控制协议与trace | turn级CALL可跨窗口转发 | WebUI 面板前端 |
| `expression_parser` | 流控制入口 | **createStreamingControlParser** | `createStreamingControlParser` | `crates/alia-audio-pipeline/src/llm-streaming-control/controller.rs:1` | 匹配ACT/DELAY/CALL解析器并发布signal | 每turn处理状态 | Rust 核心实现 |
| `expression_normalize` | 流控制入口 | **normalizeActPayload** | `normalizeActPayload` | `crates/alia-audio-pipeline/src/llm-streaming-control/payloads.rs:1` | 把ACT字段归一为合法表达请求 | emotion / expression / motion | Rust 核心实现 |
| `expression_stage` | 语义表达信号路由 | **ExpressionBus.onSignal** | `ExpressionBus.onSignal` | `packages/stage-ui/src/services/expression-bus.ts:319` | 接收归一化后的 ACT 语义负载，提取情绪/动作信号并分发至 WebUI 面板与外部扩展通道。 | 显式motion优先 | WebUI 面板前端 |
| `expression_emotion` | 语义表达信号路由 | **Emotion / EmotionPayload** | `Emotion / EmotionPayload` | `packages/stage-ui/src/constants/emotions.ts:1` | 九类标签加强度 | happy/sad/angry/think/surprised/awkward/question/curious/neutral | WebUI 面板前端 |
| `expression_queue` | 语义表达信号路由 | **emotionsQueue · 信号分发队列** | `emotionsQueue` | `packages/stage-ui/src/services/expression-bus.ts:265` | 缓冲语义情绪与动作事件，供 WebUI 沙盒状态展示或外部已挂载的扩展插件订阅消费。 | 不是心理记忆库 | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 6 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `expression_special` (语音特殊标记) | `expression_control` (StreamingControl Store) | 调用 (`call`) | dispatchWith |
| `expression_control` (StreamingControl Store) | `expression_parser` (createStreamingControlParser) | 调用 (`call`) | dispatchWith |
| `expression_parser` (createStreamingControlParser) | `expression_normalize` (normalizeActPayload) | 数据流 (`data`) | ACT payload |
| `expression_normalize` (normalizeActPayload) | `expression_stage` (ExpressionBus.onSignal) | 事件 (`event`) | act signal |
| `expression_stage` (ExpressionBus.onSignal) | `expression_emotion` (Emotion / EmotionPayload) | 数据流 (`data`) | 取规范标签 |
| `expression_emotion` (Emotion / EmotionPayload) | `expression_queue` (emotionsQueue · 信号分发队列) | 数据流 (`data`) | enqueue |

---

### 专题 17：Extension Host 内部组织与状态所有权 (`platform-extension-host`)

> **专题定位**：ExtensionHost 管理运行 session、Provider 注册事务、Kit 绑定、权限、资源及销毁。Rust 宿主外层服务额外持有安装目录和启用配置。

- **配套矢量图纸**：[职责组织图 (`17-platform-extension-host-organization.svg`)](./diagrams/17-platform-extension-host-organization.svg) ｜ [调用与数据流图 (`17-platform-extension-host-relations.svg`)](./diagrams/17-platform-extension-host-relations.svg) ｜ [Graphviz 组织图源](./diagrams/17-platform-extension-host-organization.dot) ｜ [Graphviz 关系图源](./diagrams/17-platform-extension-host-relations.dot)
- **直接涉及源码范围**：`crates/alia-extension-host/src/features/auto-reload/index.rs`, `crates/alia-extension-host/src/host/managed-sessions.rs`, `crates/alia-extension-host/src/host/registry.rs`, `crates/alia-extension-host/src/extension/disposable.rs`, `crates/alia-extension-host/src/plugin-host/core.rs`, `crates/alia-extension-host/src/plugin-host/hosted-kits.rs`, `crates/alia-extension-host/src/plugin-host/runtimes/transport/loaders/fs.rs`, `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/dependencies.rs`, `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/extension-sessions.rs`, `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/permissions.rs`, `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/resources.rs`, `crates/alia-extension-host/src/plugin-host/shared/manifest.rs`

#### 读图与工程边界守则
- session phase 是 pending / setting-up / ready / failed / stopped；这是扩展加载状态，不是角色认知状态。
- SDK 默认 permissionResolver 缺省时以 manifest.permissions 作为 resolvedGrant；权限字段存在不等于宿主已有交互式审批或系统级沙箱。
- 插件由文件入口加载；这里的生命周期隔离不能等同于操作系统进程隔离。
- stop 撤销注册并清理资源，不会强制中止正在挂起的任意扩展 JavaScript。

#### 节点与状态归属登记表（共 26 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `X01` | 清单与加载 | **Manifest v2 校验** | `parseExtensionManifest` | `crates/alia-extension-host/src/plugin-host/shared/manifest.rs:26` | 严格解析清单；身份、版本、runtime、入口与 Kits 声明是加载前提。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X02` | 清单与加载 | **文件入口加载器** | `FileSystemLoader` | `crates/alia-extension-host/src/plugin-host/runtimes/transport/loaders/fs.rs:44` | 按 runtime 选择入口并加载定义。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X03` | 宿主主干 | **start(manifest)** | `ExtensionHost.start` | `crates/alia-extension-host/src/plugin-host/core.rs:974` | 校验 runtime/ALIA 兼容性，调用 loader，再启动定义。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X04` | 宿主主干 | **startExtension** | `startExtension` | `crates/alia-extension-host/src/plugin-host/core.rs:302` | 创建 session 身份、权限快照、子容器和 Provider 事务。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X05` | Session 所有权 | **ExtensionSession** | `ExtensionSession` | `crates/alia-extension-host/src/plugin-host/core.rs:106` | 属于一次加载；持有 manifest、phase、missingKitIds、modules、permissions 和 root subscriptions。 | phase / modules Map / grants | Rust 核心实现 |
| `X06` | 宿主状态服务 | **Session 注册表** | `ExtensionSessionService` | `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/extension-sessions.rs:16` | 按运行期 session id 查询、登记、删除加载实例。 | session registry | Rust 核心实现 |
| `X07` | Session 所有权 | **Session 子容器** | `createContainer(parent: host)` | `crates/alia-extension-host/src/plugin-host/core.rs:327` | 每个扩展 session 为宿主 injeca 容器的子容器；dispose 驱动清理。 | owned lifecycle / stop outcome | Rust 核心实现 |
| `X08` | 宿主状态服务 | **PermissionService** | `PermissionService` | `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/permissions.rs:269` | 组合声明和 grant；模块授权取扩展授权交集，不能扩大权限。 | permission snapshots / revisions | Rust 核心实现 |
| `X09` | 宿主状态服务 | **DependencyService** | `DependencyService` | `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/dependencies.rs:16` | 维护能力 announced/ready/degraded/withdrawn 状态与等待。 | capability records / waiters | Rust 核心实现 |
| `X10` | 宿主状态服务 | **KitProviderRegistry** | `KitProviderRegistry` | `crates/alia-extension-host/src/plugin-host/hosted-kits.rs:476` | 维护 Host 来源、扩展提供者、预留槽、注册事务及 generation。 | records / pending / transactions / generations | Rust 核心实现 |
| `X11` | 启动状态机 | **所需 Kit 等待** | `pending + watch` | `crates/alia-extension-host/src/plugin-host/core.rs:423` | 缺少必须 Kit 时返回 pending，等所有兼容 Provider ready 后才运行 setup。 | missingKitIds | Rust 核心实现 |
| `X12` | Session 所有权 | **ExtensionSetupContext** | `ctx` | `crates/alia-extension-host/src/plugin-host/core.rs:372` | 给扩展 root 提供 identity、kits、subscriptions、modules.register。 | root scope | Rust 核心实现 |
| `X13` | 启动状态机 | **扩展 setup(ctx)** | `setupExtensionSession` | `crates/alia-extension-host/src/plugin-host/core.rs:445` | 执行用户扩展初始化；成功后提交整个 Provider 事务。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X14` | Provider 生命周期 | **注册提交** | `commitTransaction` | `crates/alia-extension-host/src/plugin-host/hosted-kits.rs:747` | 所有声明 Kit 都填充且 setup 成功才发布；原子发布该次 session 提供者。 | ready provider snapshots | Rust 核心实现 |
| `X15` | Provider 生命周期 | **回滚与代际租约** | `rollbackTransaction / registration lease` | `crates/alia-extension-host/src/plugin-host/hosted-kits.rs:784` | 失败、停止释放预留；旧 handle 不能撤销新 generation 提供者。 | lease generation | Rust 核心实现 |
| `X16` | Module 所有权 | **Module 子作用域** | `ctx.modules.register` | `crates/alia-extension-host/src/plugin-host/core.rs:377` | 独立 identity、permissions、subscriptions、kits；同 session 中 module id 不重复。 | session.modules Map | Rust 核心实现 |
| `X17` | 宿主状态服务 | **Kit API 绑定表** | `KitApiBindingRegistryService` | `crates/alia-extension-host/src/plugin-host/core.rs:245` | 记录 module 与 Kit 能力的绑定；按所属 session 清理。 | bindings by module/session | Rust 核心实现 |
| `X18` | Module 所有权 | **模块资源追踪** | `getOrCreateExtensionModuleResourceTracker` | `crates/alia-extension-host/src/plugin-host/core.rs:522` | 记录模块拥有的绑定/资源，模块销毁时撤回。 | extensionModuleResources Map | Rust 核心实现 |
| `X19` | 宿主状态服务 | **ResourceService** | `ResourceService` | `crates/alia-extension-host/src/plugin-host/runtimes/shared/services/resources.rs:34` | 提供 keyed resolver/value 资源，不是聊天记忆数据库。 | resolvers / values | Rust 核心实现 |
| `X20` | 资源清理 | **DisposableStore** | `DisposableStore` | `crates/alia-extension-host/src/extension/disposable.rs:17` | 逆注册序逐一销毁；失败仍尝试剩余项，聚合错误。 | disposables[] / disposed | Rust 核心实现 |
| `X21` | 资源清理 | **stop(sessionId)** | `stop` | `crates/alia-extension-host/src/plugin-host/core.rs:1035` | 将 phase 置 stopped、撤 Provider、dispose session 容器。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X22` | 资源清理 | **分组清理** | `disposeSessionResources` | `crates/alia-extension-host/src/plugin-host/core.rs:736` | 顺序为绑定撤回、module 清理、root subscriptions；继续尝试所有组。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X23` | 资源清理 | **reload(sessionId)** | `reload` | `crates/alia-extension-host/src/plugin-host/core.rs:1046` | 旧 session 清理成功后重新 start；产生新 session identity。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `X24` | Rust 宿主外层 | **服务端插件清单注册表** | `createExtensionHostRegistry` | `crates/alia-extension-host/src/host/registry.rs:305` | 扫描 extensions 根目录，保存 validated manifest、路径与 id 索引。 | entries / manifests / lookup Map | Rust 核心实现 |
| `X25` | Rust 宿主外层 | **托管 session 与清理债务** | `ManagedExtensionSessions` | `crates/alia-extension-host/src/host/managed-sessions.rs:59` | 外层跟踪扩展加载状态与未完成清理；决定能否再次加载。 | managed sessions / cleanup report | Rust 核心实现 |
| `X26` | Rust 宿主外层 | **自动重载监听** | `createExtensionAutoReloadFeature` | `crates/alia-extension-host/src/features/auto-reload/index.rs:50` | 文件变化触发同一启停流程；重载不等于保留原对象。 | watchers / reload queue | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 31 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `X24` (服务端插件清单注册表) | `X01` (Manifest v2 校验) | 调用 (`call`) | 读取并校验 |
| `X24` (服务端插件清单注册表) | `X03` (start(manifest)) | 调用 (`call`) | 按清单加载 |
| `X03` (start(manifest)) | `X02` (文件入口加载器) | 调用 (`call`) | loadExtensionFor |
| `X03` (start(manifest)) | `X04` (startExtension) | 调用 (`call`) | loaded definition |
| `X04` (startExtension) | `X05` (ExtensionSession) | 持有 / 归属 (`owns`) | 创建 / 初始化 |
| `X04` (startExtension) | `X06` (Session 注册表) | 持有 / 归属 (`owns`) | 创建 / 初始化 |
| `X04` (startExtension) | `X07` (Session 子容器) | 持有 / 归属 (`owns`) | 创建 / 初始化 |
| `X04` (startExtension) | `X08` (PermissionService) | 持有 / 归属 (`owns`) | 创建 / 初始化 |
| `X04` (startExtension) | `X10` (KitProviderRegistry) | 持有 / 归属 (`owns`) | 创建 / 初始化 |
| `X04` (startExtension) | `X12` (ExtensionSetupContext) | 持有 / 归属 (`owns`) | 创建 / 初始化 |
| `X04` (startExtension) | `X11` (所需 Kit 等待) | 调用 (`call`) | 所需 Kit 缺失 |
| `X11` (所需 Kit 等待) | `X13` (扩展 setup(ctx)) | 调用 (`call`) | 兼容 Provider ready |
| `X04` (startExtension) | `X13` (扩展 setup(ctx)) | 调用 (`call`) | 依赖已满足 |
| `X12` (ExtensionSetupContext) | `X16` (Module 子作用域) | 调用 (`call`) | modules.register |
| `X12` (ExtensionSetupContext) | `X20` (DisposableStore) | 持有 / 归属 (`owns`) | root subscriptions |
| `X16` (Module 子作用域) | `X20` (DisposableStore) | 持有 / 归属 (`owns`) | module subscriptions |
| `X08` (PermissionService) | `X16` (Module 子作用域) | 调用 (`call`) | grant 交集 |
| `X13` (扩展 setup(ctx)) | `X14` (注册提交) | 调用 (`call`) | setup 成功 |
| `X13` (扩展 setup(ctx)) | `X15` (回滚与代际租约) | 生命周期 (`lifecycle`) | setup 失败 |
| `X14` (注册提交) | `X10` (KitProviderRegistry) | 调用 (`call`) | 发布提供者 |
| `X16` (Module 子作用域) | `X17` (Kit API 绑定表) | 调用 (`call`) | 能力绑定 |
| `X16` (Module 子作用域) | `X18` (模块资源追踪) | 持有 / 归属 (`owns`) | 资源所属 |
| `X21` (stop(sessionId)) | `X15` (回滚与代际租约) | 生命周期 (`lifecycle`) | 撤销提供者 |
| `X21` (stop(sessionId)) | `X07` (Session 子容器) | 生命周期 (`lifecycle`) | dispose |
| `X07` (Session 子容器) | `X22` (分组清理) | 生命周期 (`lifecycle`) | 清理 effect |
| `X22` (分组清理) | `X17` (Kit API 绑定表) | 生命周期 (`lifecycle`) | withdraw / unbind |
| `X22` (分组清理) | `X20` (DisposableStore) | 生命周期 (`lifecycle`) | 逆序销毁 |
| `X23` (reload(sessionId)) | `X21` (stop(sessionId)) | 生命周期 (`lifecycle`) | 先 stop |
| `X23` (reload(sessionId)) | `X03` (start(manifest)) | 生命周期 (`lifecycle`) | 后 fresh start |
| `X25` (托管 session 与清理债务) | `X21` (stop(sessionId)) | 调用 (`call`) | 管理清理 |
| `X26` (自动重载监听) | `X25` (托管 session 与清理债务) | 调用 (`call`) | 重载状态协调 |

---

### 专题 18：Kit 契约、身份、通道与实现边界 (`platform-kits-protocol`)

> **专题定位**：身份与事件契约、Host-provided Kit API、Extension-hosted Provider 注册、远程通道封装是不同层。不能把已有类型当作已经接通的执行能力。

- **配套矢量图纸**：[职责组织图 (`18-platform-kits-protocol-organization.svg`)](./diagrams/18-platform-kits-protocol-organization.svg) ｜ [调用与数据流图 (`18-platform-kits-protocol-relations.svg`)](./diagrams/18-platform-kits-protocol-relations.svg) ｜ [Graphviz 组织图源](./diagrams/18-platform-kits-protocol-organization.dot) ｜ [Graphviz 关系图源](./diagrams/18-platform-kits-protocol-relations.dot)
- **直接涉及源码范围**：`crates/alia-extension-host/src/kits/index.rs`, `crates/alia-extension-host/src/kits/tool/registry.rs`, `crates/alia-extension-host/src/kits/widget/index.rs`, `crates/alia-plugin-protocol/src/index.rs`, `crates/alia-plugin-protocol/src/types/events.rs`, `crates/alia-plugin-protocol/src/kits/widget/index.rs`, `crates/alia-plugin-protocol/src/kits/tool/index.rs`, `crates/alia-extension-host/Cargo.toml`, `crates/alia-extension-host/src/channels/index.rs`, `crates/alia-extension-host/src/channels/remote/websocket/index.rs`, `crates/alia-extension-host/src/kit/contract.rs`, `crates/alia-extension-host/src/kit/exposure-policy.rs`, `crates/alia-extension-host/src/kit/index.rs`, `crates/alia-extension-host/src/plugin-host/core.rs`, `crates/alia-extension-host/src/plugin-host/hosted-kits.rs`, `crates/alia-extension-host/src/plugin-host/runtimes/transport/index.rs`, `crates/alia-extension-host/src/plugin-host/runtimes/web/index.rs`, `crates/alia-extension-host/src/plugin/remote/index.rs`

#### 读图与工程边界守则
- server-sdk 的 WebSocketExtensionPeer 另有可运行的协议连接实现；它不能证明 plugin-host 的全部 transport 分支实现。
- 事件表中很多 configuration / contribution 名称只是共享契约；需要寻找具体 handler 才能说已接线。
- Extension-hosted Provider 可以被预留、登记、发布和撤回；这与 Consumer 能调用其 handlers 是两个完成度。

#### 节点与状态归属登记表（共 25 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `K01` | extension-protocol | **Extension 身份** | `ExtensionIdentity` | `crates/alia-plugin-protocol/src/index.rs:25` | 扩展 id、版本、session 身份的通用契约。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K02` | extension-protocol | **Module 身份** | `ExtensionModuleIdentity` | `crates/alia-plugin-protocol/src/index.rs:47` | 模块 id 指向其所属扩展身份。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K03` | extension-protocol | **权限领域** | `ModulePermissionArea` | `crates/alia-plugin-protocol/src/index.rs:62` | apis / resources / capabilities / processors / pipelines 的声明与授权形状。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K04` | plugin-protocol | **事件与负载契约** | `ProtocolEvents` | `crates/alia-plugin-protocol/src/types/events.rs:1246` | 共享 input/output、registry、config、spark、context、asset 事件名与负载。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K05` | plugin-protocol | **路由与投递契约** | `DeliveryConfig` | `crates/alia-plugin-protocol/src/types/events.rs:370` | broadcast / consumer / consumer-group，以及 first / round-robin / priority / sticky。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K06` | Kit 公共契约 | **KitContract** | `KitContract` | `crates/alia-extension-host/src/kit/contract.rs:99` | id、精确 version、methods、events、允许的暴露策略。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K07` | Kit 公共契约 | **方法 / 事件声明** | `defineKitMethod / defineKitEvent` | `crates/alia-extension-host/src/kit/contract.rs:78` | 建立输入、输出和事件负载的类型契约。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K08` | Kit 公共契约 | **暴露策略** | `ExposePolicy` | `crates/alia-extension-host/src/kit/exposure-policy.rs:6` | local-only / remote-observable / remote-callable；枚举本身不是远程执行器。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K09` | Host Kit API | **KitRef / Client 工厂** | `KitRef` | `crates/alia-extension-host/src/kit/index.rs:55` | Host Kit 的身份、版本、可用策略和 createClient。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K10` | Host Kit API | **Host API 注册** | `registerKitApi` | `crates/alia-extension-host/src/plugin-host/core.rs:488` | 把 Host Kit 的客户端工厂登记到提供者注册表。 | HostSourceLease | Rust 核心实现 |
| `K11` | Host Kit API | **use / tryUse / watch** | `createKitRegistry` | `crates/alia-extension-host/src/plugin-host/core.rs:598` | 消费者获取客户端或观察可用性；use 失败抛错，tryUse 返回结构化结果。 | watchers / subscriptions | Rust 核心实现 |
| `K12` | Host Kit API | **授权检查 / createClient** | `resolveKitApi` | `crates/alia-extension-host/src/plugin-host/core.rs:568` | 目前取 getHostClientFactory，检查 apis:invoke 权限，再构造带归属的 Client。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K13` | Extension-hosted Kit | **扩展提供 Kit** | `createExtensionKitRegistry.provide` | `crates/alia-extension-host/src/plugin-host/core.rs:648` | 仅 root setup 可 provide；对照 manifest 声明填注册事务。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K14` | Extension-hosted Kit | **Provider handlers 储存** | `captureProviderEndpoint` | `crates/alia-extension-host/src/plugin-host/hosted-kits.rs:701` | 检查 methods 成员名称并捕获 handlers；Phase 3 不调用这些方法。 | accepted endpoint handlers | Rust 核心实现 |
| `K15` | 尚未完整接通 | **Phase 4 消费闭环** | `Consumer Clients / calls / events / transport` | `crates/alia-extension-host/Cargo.toml:77` | 文档明确属于后续阶段；本提交 resolveKitApi 没有消费扩展提供者的方法调用链。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K16` | 通道积木 | **扩展 / Module 通道** | `createExtensionChannelScope / createModuleChannelScope` | `crates/alia-extension-host/src/channels/index.rs:43` | 把身份和已有 Eventa context 组成 scope；module 与 extension 共享同一 context。本函数不注册订阅或提供清理。 | identity + context reference | Rust 核心实现 |
| `K17` | 通道积木 | **原生 WebSocket 封装** | `createWebSocketHostChannel / createWebSocketExtensionTransport` | `crates/alia-extension-host/src/channels/remote/websocket/index.rs:15` | 把调用方已连接的 WebSocket 包装成 Eventa context；不负责完整连接生命周期。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K18` | 尚未完整接通 | **插件传输通道工厂** | `createPluginContext` | `crates/alia-extension-host/src/plugin-host/runtimes/transport/index.rs:22` | 仅 in-memory 实现；WebSocket / Wasm / Subprocess 传输通道统一实现。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K19` | 尚未完整接通 | **Web transport 工厂** | `createPluginContext` | `crates/alia-extension-host/src/plugin-host/runtimes/web/index.rs:21` | 仅 in-memory 实现；websocket / web-worker 分支 throw 未实现。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K20` | 尚未完整接通 | **远程扩展启动占位** | `setupRemotePluginScope` | `crates/alia-extension-host/src/plugin/remote/index.rs:7` | 函数体为空，不能标成完整的远程扩展启动流程。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K21` | Extension Kits | **Widget 扩展面板作者 API** | `createWidget` | `crates/alia-plugin-protocol/src/kits/widget/index.rs:68` | 使用 Extension widgetKit，把独立交互组件注册/挂载到桌面宿主。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K22` | Extension Kits | **Tool 作者 API** | `registerTools` | `crates/alia-plugin-protocol/src/kits/tool/index.rs:59` | 通过 Tool Kit 注册工具和 prompt，属于宿主工具可扩展点。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K23` | Extension Kits | **Widget Kit 描述** | `widgetPluginKitDescriptor` | `crates/alia-extension-host/src/kits/widget/index.rs:18` | 定义桌面 widget 的能力与配置契约。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `K24` | Extension Kits | **ExtensionToolRegistry** | `ExtensionToolRegistry` | `crates/alia-extension-host/src/kits/tool/registry.rs:46` | 归属 session/module 的工具注册表，卸载时清理所有者 scope。 | tools / toolset prompts | Rust 核心实现 |
| `K25` | Extension Kits | **内置 Kit 组装** | `createBuiltInExtensionKits` | `crates/alia-extension-host/src/kits/index.rs:101` | 注册 Widget/Widget 描述与 Widget/Tool 客户端工厂；逆序撤注册。 | HostSourceLease[] | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 24 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `K01` (Extension 身份) | `K02` (Module 身份) | 契约 / 配置 (`contract`) | 所属身份 |
| `K01` (Extension 身份) | `K04` (事件与负载契约) | 契约 / 配置 (`contract`) | 协议复用 |
| `K02` (Module 身份) | `K04` (事件与负载契约) | 契约 / 配置 (`contract`) | 事件来源 |
| `K03` (权限领域) | `K12` (授权检查 / createClient) | 契约 / 配置 (`contract`) | 检查授权 |
| `K05` (路由与投递契约) | `K04` (事件与负载契约) | 契约 / 配置 (`contract`) | 投递语义 |
| `K07` (方法 / 事件声明) | `K06` (KitContract) | 契约 / 配置 (`contract`) | 成员契约 |
| `K08` (暴露策略) | `K06` (KitContract) | 契约 / 配置 (`contract`) | 暴露限制 |
| `K09` (KitRef / Client 工厂) | `K10` (Host API 注册) | 调用 (`call`) | 登记客户端工厂 |
| `K10` (Host API 注册) | `K12` (授权检查 / createClient) | 调用 (`call`) | 取得 Host 工厂 |
| `K11` (use / tryUse / watch) | `K12` (授权检查 / createClient) | 调用 (`call`) | 解析可用客户端 |
| `K06` (KitContract) | `K13` (扩展提供 Kit) | 契约 / 配置 (`contract`) | 共享契约 |
| `K13` (扩展提供 Kit) | `K14` (Provider handlers 储存) | 调用 (`call`) | 登记 handlers |
| `K01` (Extension 身份) | `K16` (扩展 / Module 通道) | 契约 / 配置 (`contract`) | scope 身份 |
| `K02` (Module 身份) | `K16` (扩展 / Module 通道) | 契约 / 配置 (`contract`) | module 身份 |
| `K17` (原生 WebSocket 封装) | `K16` (扩展 / Module 通道) | 调用 (`call`) | Eventa context |
| `K25` (内置 Kit 组装) | `K10` (Host API 注册) | 调用 (`call`) | registerKitApi |
| `K25` (内置 Kit 组装) | `K23` (Widget Kit 描述) | 调用 (`call`) | registerKit descriptor |
| `K25` (内置 Kit 组装) | `K24` (ExtensionToolRegistry) | 调用 (`call`) | Tool Registry |
| `K21` (Widget 扩展面板作者 API) | `K11` (use / tryUse / watch) | 调用 (`call`) | kits.use(widgetKit) |
| `K22` (Tool 作者 API) | `K11` (use / tryUse / watch) | 调用 (`call`) | kits.use(toolKit) |
| `K14` (Provider handlers 储存) | `K15` (Phase 4 消费闭环) | 尚未接通 (`unimplemented`) | 注册已实现，调用闭环后续 |
| `K17` (原生 WebSocket 封装) | `K18` (插件传输通道工厂) | 尚未接通 (`unimplemented`) | 尚未由统一工厂接通 |
| `K17` (原生 WebSocket 封装) | `K19` (Web transport 工厂) | 尚未接通 (`unimplemented`) | 尚未由统一工厂接通 |
| `K20` (远程扩展启动占位) | `K17` (原生 WebSocket 封装) | 尚未接通 (`unimplemented`) | 仅预留启动位置 |

---

### 专题 19：模块 WebSocket 通道：连接、身份、路由和资源 (`platform-channel`)

> **专题定位**：server-sdk 持有客户端连接；server-runtime 持有连接/模块注册表与路由策略。模型执行、会话历史和媒体资源所有者在别处。

- **配套矢量图纸**：[职责组织图 (`19-platform-channel-organization.svg`)](./diagrams/19-platform-channel-organization.svg) ｜ [调用与数据流图 (`19-platform-channel-relations.svg`)](./diagrams/19-platform-channel-relations.svg) ｜ [Graphviz 组织图源](./diagrams/19-platform-channel-organization.dot) ｜ [Graphviz 关系图源](./diagrams/19-platform-channel-relations.dot)
- **直接涉及源码范围**：`crates/alia-channel-gateway/src/client/index.rs`, `crates/alia-plugin-protocol/README.md`, `crates/alia-channel-gateway/src/index.rs`, `crates/alia-protocol/src/server-ws/codec.rs`, `crates/alia-channel-gateway/src/server-ws/alia/consumers.rs`, `crates/alia-channel-gateway/src/server/index.rs`, `crates/alia-channel-gateway/README.md`, `crates/alia-channel-gateway/src/client.rs`, `crates/alia-channel-gateway/src/extension-peer.rs`, `crates/alia-protocol/src/types/websocket/events.rs`

#### 读图与工程边界守则
- Client.connect() 的 ready 表示 transport 已连接且所需握手完成，不只是 WebSocket open。
- 通道不持有 Chat Session，不执行语言模型；它是模块事件路由服务。
- metadata.source 可由客户端声明；metadata.sender 是服务器根据实际连接填入的来源。
- server-runtime 中有协议中性的 peer/heartbeat 逻辑迁移到 better-ws/server 的 TODO，不把 TODO 画成已完成。
- sendOrThrow 只是对本地无法发送抛错，不代表对端应用已执行，也不是持久消息队列的投递保证。

#### 节点与状态归属登记表（共 23 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `C01` | 客户端进程 | **ALIA Client** | `Client` | `crates/alia-channel-gateway/src/client.rs:215` | 面向外部模块的连接对象，封装握手、发事件、监听与资源获取。 | status / connectTask / eventListeners | Rust 核心实现 |
| `C02` | 客户端底座 | **通用 WebSocket 底座** | `createClient` | `crates/alia-channel-gateway/src/client/index.rs:362` | 协议无关的 transport、重连、heartbeat 和 preparing 生命周期。 | transport state | Rust 核心实现 |
| `C03` | 客户端进程 | **连接状态机** | `ClientStatus` | `crates/alia-channel-gateway/src/client.rs:29` | idle → connecting → authenticating → announcing → ready，另外含 reconnecting/closing/closed/failed。 | connectionStatus | Rust 核心实现 |
| `C04` | 客户端握手 | **认证与宣告** | `prepareProtocolConnection` | `crates/alia-channel-gateway/src/client.rs:439` | 有 token 先 module:authenticate，再 extension:module:announce，等确认或 registry sync 中出现自身。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C05` | 客户端进程 | **事件发送 / 订阅** | `send / sendOrThrow / onEvent` | `crates/alia-channel-gateway/src/client.rs:344` | send 返回可用性；sendOrThrow 失败抛错；onEvent 分发给监听器并提供退订。 | event listeners | Rust 核心实现 |
| `C06` | 客户端进程 | **多 Module 扩展 peer** | `WebSocketExtensionPeer` | `crates/alia-channel-gateway/src/extension-peer.rs:69` | 显式 peer 认证、扩展 announce 与多个 module announce 的便捷封装。 | extension identity / Client | Rust 核心实现 |
| `C07` | 共享协议 | **事件封套** | `WebSocketBaseEvent` | `crates/alia-protocol/src/types/websocket/events.rs:24` | type、data、metadata.source、event.id/parentId、sender 和 route。source 声明与 server sender 分开。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C08` | server-runtime 进程 | **服务生命周期** | `createServer` | `crates/alia-channel-gateway/src/server/index.rs:108` | 独立/嵌入服务器的 start/stop/restart/updateConfig。 | server handle / options | Rust 核心实现 |
| `C09` | server-runtime 进程 | **通道 App** | `setupApp` | `crates/alia-channel-gateway/src/index.rs:215` | 组装 Axum Router、WebSocket handler、注册表、路由中间件和健康检查。 | dispose flag / instanceId | Rust 核心实现 |
| `C10` | 服务端接收 | **事件解码与校验** | `parseEvent` | `crates/alia-protocol/src/server-ws/codec.rs:51` | 解析 envelope，接受基本 type/data 形状并处理 heartbeat 帧；不是每个业务负载的完整校验器。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C11` | 服务端接收 | **认证处理** | `module / peer / extension authenticate` | `crates/alia-channel-gateway/src/index.rs:620` | 检查配置 token，设置连接 authenticated；未认证业务事件拒绝。 | peer authenticated | Rust 核心实现 |
| `C12` | 服务端状态 | **Peer 注册表** | `peers Map` | `crates/alia-channel-gateway/src/index.rs:233` | 按连接 id 保存连接、认证、身份、heartbeat 和健康信息。 | Map<peerId, AuthenticatedPeer> | Rust 核心实现 |
| `C13` | 服务端状态 | **Module 注册表** | `peersByModule / extensionModules` | `crates/alia-channel-gateway/src/index.rs:234` | 模块宣告关联到连接；同一个 extension peer 可宣告多个模块。 | module maps | Rust 核心实现 |
| `C14` | 服务端接收 | **宣告 / Registry 同步** | `extension:module:announce` | `crates/alia-channel-gateway/src/index.rs:724` | 验证身份，注册模块并给客户端确认；registry:modules:sync 广播已知模块。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C15` | 服务端路由 | **可信连接来源** | `metadata.sender` | `crates/alia-channel-gateway/src/index.rs:878` | 转发时重写 sender.peerId 与已宣告 modules，接收方据此识别真实发送连接。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C16` | 服务端路由 | **目的地 / Policy** | `collectDestinations / forEachEventMiddlewares` | `crates/alia-channel-gateway/src/index.rs:883` | 应用目标筛选与策略，可 drop、定向 targets 或 broadcast。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C17` | 服务端状态 | **Consumer 注册与选择** | `createConsumerOrchestrator` | `crates/alia-channel-gateway/src/server-ws/alia/consumers.rs:174` | 按 event/group 管理消费者，维护 round-robin/sticky 状态。 | registrations / cursors / sticky assignments | Rust 核心实现 |
| `C18` | 服务端路由 | **选择策略** | `selectConsumerPeerId` | `crates/alia-channel-gateway/src/server-ws/alia/consumers.rs:117` | 在认证且健康的候选中按 first、priority、round-robin 或 sticky 选择。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C19` | 服务端路由 | **转发 / 广播** | `consumer delivery / broadcast` | `crates/alia-channel-gateway/src/index.rs:901` | consumer 模式发一个被选连接；普通路由遍历符合条件的认证 peer，跳过发件人。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C20` | 服务端生命周期 | **Heartbeat 与移除** | `markPeerAlive / health check` | `crates/alia-channel-gateway/src/index.rs:247` | 更新活跃时间；检测失活、发布健康变化；断开时撤销模块与 consumer。 | lastHeartbeatAt / healthy | Rust 核心实现 |
| `C21` | 宿主会话边界 | **输入的外部场景绑定** | `overrides.binding / overrides.sessionId` | `crates/alia-plugin-protocol/README.md:22` | input:text 可带外部 scene binding；由宿主解析到持久会话。binding 优先于 sessionId。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C22` | 资源访问 | **资源请求** | `getAsset` | `crates/alia-channel-gateway/src/client.rs:367` | asset:get:request 与 parentId 关联响应；默认10秒超时，finally 移除监听。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `C23` | 资源访问 | **资源所有者是 Stage** | `alia-asset:<id>` | `crates/alia-channel-gateway/README.md:44` | 聊天事件只带附件引用。Stage 返回数据；缺失/超过50MB/无回答拒绝。通道本身不存媒体。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 30 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `C01` (ALIA Client) | `C02` (通用 WebSocket 底座) | 持有 / 归属 (`owns`) | 持有 transport |
| `C01` (ALIA Client) | `C03` (连接状态机) | 调用 (`call`) | 公开状态 |
| `C01` (ALIA Client) | `C04` (认证与宣告) | 调用 (`call`) | connect / prepare |
| `C01` (ALIA Client) | `C05` (事件发送 / 订阅) | 调用 (`call`) | 发送 / 监听 |
| `C06` (多 Module 扩展 peer) | `C01` (ALIA Client) | 调用 (`call`) | 使用 Client |
| `C04` (认证与宣告) | `C11` (认证处理) | 调用 (`call`) | authenticate |
| `C04` (认证与宣告) | `C14` (宣告 / Registry 同步) | 调用 (`call`) | announce |
| `C05` (事件发送 / 订阅) | `C07` (事件封套) | 调用 (`call`) | 封装事件 |
| `C08` (服务生命周期) | `C09` (通道 App) | 调用 (`call`) | setupApp |
| `C09` (通道 App) | `C12` (Peer 注册表) | 持有 / 归属 (`owns`) | 持有 / 管理 |
| `C09` (通道 App) | `C13` (Module 注册表) | 持有 / 归属 (`owns`) | 持有 / 管理 |
| `C09` (通道 App) | `C17` (Consumer 注册与选择) | 持有 / 归属 (`owns`) | 持有 / 管理 |
| `C09` (通道 App) | `C20` (Heartbeat 与移除) | 持有 / 归属 (`owns`) | 持有 / 管理 |
| `C07` (事件封套) | `C10` (事件解码与校验) | 调用 (`call`) | JSON 帧 |
| `C10` (事件解码与校验) | `C11` (认证处理) | 调用 (`call`) | 控制事件 |
| `C14` (宣告 / Registry 同步) | `C13` (Module 注册表) | 数据流 (`data`) | 更新模块身份 |
| `C13` (Module 注册表) | `C14` (宣告 / Registry 同步) | 数据流 (`data`) | registry sync |
| `C10` (事件解码与校验) | `C15` (可信连接来源) | 调用 (`call`) | 业务事件已认证 |
| `C12` (Peer 注册表) | `C15` (可信连接来源) | 调用 (`call`) | 可信发送连接 |
| `C15` (可信连接来源) | `C16` (目的地 / Policy) | 调用 (`call`) | source / sender / route |
| `C16` (目的地 / Policy) | `C17` (Consumer 注册与选择) | 调用 (`call`) | consumer delivery |
| `C17` (Consumer 注册与选择) | `C18` (选择策略) | 调用 (`call`) | 候选 |
| `C18` (选择策略) | `C19` (转发 / 广播) | 调用 (`call`) | 目标 peer |
| `C16` (目的地 / Policy) | `C19` (转发 / 广播) | 调用 (`call`) | targets / broadcast |
| `C19` (转发 / 广播) | `C05` (事件发送 / 订阅) | 调用 (`call`) | 路由事件 |
| `C20` (Heartbeat 与移除) | `C12` (Peer 注册表) | 生命周期 (`lifecycle`) | 健康 / 注销 |
| `C21` (输入的外部场景绑定) | `C05` (事件发送 / 订阅) | 调用 (`call`) | input:text |
| `C22` (资源请求) | `C19` (转发 / 广播) | 调用 (`call`) | asset:get:request |
| `C19` (转发 / 广播) | `C23` (资源所有者是 Stage) | 调用 (`call`) | Stage 读取媒体 |
| `C23` (资源所有者是 Stage) | `C22` (资源请求) | 数据流 (`data`) | 定向回应 parentId |

---

### 专题 20：Hosted Backend：产品资源、认证、同步与模型网关 (`platform-backend`)

> **专题定位**：这是独立的 Rust Axum API 与 Better Auth/OIDC 服务。聊天同步 WebSocket 与模块 server channel 是两套不同接口。资源领域服务负责持久数据，模型网关负责托管推理。

- **配套矢量图纸**：[职责组织图 (`20-platform-backend-organization.svg`)](./diagrams/20-platform-backend-organization.svg) ｜ [调用与数据流图 (`20-platform-backend-relations.svg`)](./diagrams/20-platform-backend-relations.svg) ｜ [Graphviz 组织图源](./diagrams/20-platform-backend-organization.dot) ｜ [Graphviz 关系图源](./diagrams/20-platform-backend-relations.dot)
- **直接涉及源码范围**：`crates/alia-server/src/app.rs`, `crates/alia-server/src/libs/request-auth.rs`, `crates/alia-server/src/routes/chat-ws/rpc.rs`, `crates/alia-server/src/routes/chat-ws/runtime.rs`, `crates/alia-server/src/schemas/characters.rs`, `crates/alia-server/src/schemas/chats.rs`, `crates/alia-server/src/server.rs`, `crates/alia-server/src/services/domain/chats.rs`, `crates/alia-auth-service/src/auth.rs`, `crates/alia-auth-service/src/server.rs`, `crates/alia-server/src/router.rs`, `crates/alia-auth-service/src/schema.rs`

#### 读图与工程边界守则
- 两个 WebSocket 网络应分图：Hosted Chat WS 用来存取/同步产品会话；server-channel 路由模块事件。
- PostgreSQL 同一数据库并不意味着服务间业务数据所有权混同；API 负责共享迁移，Auth 不重复运行迁移。
- 客户端可以直连用户配置的模型服务；托管网关只是其中一条推理路径。
- ORM 的 relations(...) 指数据库关联，不应翻译成已实现的人际羁绊状态系统。
- pushMessages 明确只接受 user / assistant 两种 role；云端消息序列同步不自动包括 core-agent 中全部原生 continuation、live task、音频资源或扩展 session 状态。

#### 节点与状态归属登记表（共 26 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `B01` | 部署进程 | **本地部署边缘** | `caddy` | `crates/alia-server/src/router.rs:128` | 本地共享公开入口把 Auth 与资源 API 路由到不同进程。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B02` | 部署进程 | **Resource API 进程** | `runApiServer` | `crates/alia-server/src/server.rs:22` | createApp → serve Rust Axum HTTP → injectWebSocket。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B03` | API 进程 | **API 依赖装配** | `createApp` | `crates/alia-server/src/app.rs:524` | injeca 装配数据库、Redis、对象存储、领域服务、网关与遥测。 | app dependency container | Rust 核心实现 |
| `B04` | API 进程 | **Rust Axum 路由与中间件** | `buildApp` | `crates/alia-server/src/app.rs:152` | CORS、会话认证、body limits、业务 routes、WebSocket 和错误处理。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B05` | API 认证边界 | **Token / JWKS 校验** | `resolveRequestAuth` | `crates/alia-server/src/libs/request-auth.rs:147` | 资源 API 校验由 Auth 签发的访问凭证，解析对应用户。 | JWKS cache | Rust 核心实现 |
| `B06` | Auth 进程 | **Auth 进程** | `createAuthServer` | `crates/alia-auth-service/src/server.rs:128` | 单独构建认证 HTTP 服务、DB/Redis、邮件和资源 API 客户端。 | 独立 container | Rust 核心实现 |
| `B07` | Auth 进程 | **Better Auth / OIDC** | `createAuth` | `crates/alia-auth-service/src/auth.rs:436` | 注册认证插件，签发 OIDC/JWT；预置 WebUI 面板 / 外部扩展客户端 客户端。 | auth sessions / issuer keys | Rust 核心实现 |
| `B08` | 持久数据契约 | **Auth 所有的数据库契约** | `auth-shared schema` | `crates/alia-auth-service/src/schema.rs:4` | 账户、会话、认证及 OAuth/OIDC 相关 schema 由 auth-shared 定义。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B09` | 基础设施 | **PostgreSQL / 迁移** | `datastore:db` | `crates/alia-server/src/app.rs:545` | API 建立 Rust SQLx (SQLite-vec / PgVector) 数据连接并负责共享数据库迁移。 | 关系持久数据 | Rust 核心实现 |
| `B10` | 基础设施 | **Redis** | `datastore:redis` | `crates/alia-server/src/app.rs:575` | 用于跨实例同步、cache、streams 和并发/计费协调所需状态。 | 共享运行状态 | Rust 核心实现 |
| `B11` | 基础设施 | **S3 对象存储** | `createS3ObjectStore` | `crates/alia-server/src/app.rs:603` | 模型、资源文件等的二进制对象；本地 Compose 提供 RustFS。 | binary objects | Rust 核心实现 |
| `B12` | 资源领域服务 | **角色资源服务** | `createCharacterService` | `crates/alia-server/src/app.rs:653` | 面向产品角色资源的业务层，和用户会话角色卡对象不同。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B13` | 资源领域服务 | **角色卡服务** | `createCharacterCardService` | `crates/alia-server/src/app.rs:658` | 持久角色卡及字段，接 /api/v1/character-cards。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B14` | 资源领域服务 | **形象 / 声音资源** | `createVoicePackService / createDisplayModelService` | `crates/alia-server/src/app.rs:727` | voice-pack 和 display-model 是资源领域服务；实际渲染和播放在客户端。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B15` | 资源领域服务 | **Provider 配置 / 目录** | `createProviderService / createProviderCatalogService` | `crates/alia-server/src/app.rs:674` | 管理用户配置与托管能力目录；配置加密依赖 envelopeCrypto。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B16` | 会话同步领域 | **ChatService** | `createChatService` | `crates/alia-server/src/services/domain/chats.rs:49` | 创建/读取会话成员，写入消息并分配顺序号，按用户授权读取。 | chat / member / message persistence | Rust 核心实现 |
| `B17` | 会话同步运行时 | **Hosted Chat WS Runtime** | `createChatWsRuntime` | `crates/alia-server/src/routes/chat-ws/runtime.rs:16` | 拥有进程内连接注册和 Redis 广播协调。与 server-runtime peers 表不同。 | ChatConnectionRegistry / BroadcastCoordinator | Rust 核心实现 |
| `B18` | 会话同步运行时 | **sendMessages / pullMessages RPC** | `registerChatRpcHandlers` | `crates/alia-server/src/routes/chat-ws/rpc.rs:37` | 解析请求 → pushMessages → pull newly stored messages → 按会话成员广播；pull 按 seq 增量获取。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B19` | 会话同步运行时 | **Chat WS v1 / v2** | `createChatWsV1Handlers / createChatWsV2Handlers` | `crates/alia-server/src/app.rs:215` | /ws/chat 与 /ws/v2/chat 共用核心 runtime；v2 在 open 后做认证。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B20` | 持久数据契约 | **核心资源表** | `character / avatarModel / characterCapabilities / prompts` | `crates/alia-server/src/schemas/characters.rs:47` | 角色资源表族；characterRelations 是 ORM 表关系定义，不是情感羁绊引擎。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B21` | 持久数据契约 | **会话与消息表** | `chats / chatMembers / messages` | `crates/alia-server/src/schemas/chats.rs:44` | chatId、成员、role、seq、content、mediaIds、stickerIds 等；不是运行时全部状态的镜像。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B22` | 托管模型网关 | **托管推理路由器** | `createLlmRouterService` | `crates/alia-server/src/app.rs:819` | 托管网关的 provider route、并发额度、key 轮换及尝试逻辑；不是 Stage core-agent。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B23` | 托管模型网关 | **OpenAI / Audio API** | `v1Routes.openaiRoutes / audioRoutes` | `crates/alia-server/src/app.rs:431` | 托管 chat-completions、responses 与 speech 服务的协议入口。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B24` | 托管模型网关 | **语音实时通路** | `audio speech WS / transcriptions stream` | `crates/alia-server/src/app.rs:264` | 独立实时语音合成 WS 与转写流入口。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |
| `B25` | 托管服务支撑 | **用量 / 计费 / 请求日志** | `billingService / llmBilling / flux / requestLog` | `crates/alia-server/src/app.rs:742` | 托管服务用量、余额交易、支付与请求记录；不能当作角色记忆。 | 账本 / 请求记录 | Rust 核心实现 |
| `B26` | 跨服务生命周期 | **账户删除协作** | `createUserDeletionService` | `crates/alia-server/src/app.rs:790` | Auth 删除前调用 API 删除业务资源；Auth 和 API 各自拥有相应数据处理。 | 本图未单独列出；按相连对象与源码确认。 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 31 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `B01` (本地部署边缘) | `B02` (Resource API 进程) | 调用 (`call`) | 资源 API 路由 |
| `B01` (本地部署边缘) | `B06` (Auth 进程) | 调用 (`call`) | 认证路由 |
| `B02` (Resource API 进程) | `B03` (API 依赖装配) | 调用 (`call`) | createApp |
| `B03` (API 依赖装配) | `B04` (Rust Axum 路由与中间件) | 调用 (`call`) | buildApp |
| `B04` (Rust Axum 路由与中间件) | `B05` (Token / JWKS 校验) | 调用 (`call`) | 认证中间件 |
| `B05` (Token / JWKS 校验) | `B07` (Better Auth / OIDC) | 调用 (`call`) | issuer / JWKS |
| `B06` (Auth 进程) | `B07` (Better Auth / OIDC) | 调用 (`call`) | createAuth |
| `B07` (Better Auth / OIDC) | `B08` (Auth 所有的数据库契约) | 调用 (`call`) | 认证 schema |
| `B03` (API 依赖装配) | `B09` (PostgreSQL / 迁移) | 数据流 (`data`) | DB / migrate |
| `B03` (API 依赖装配) | `B10` (Redis) | 数据流 (`data`) | Redis |
| `B03` (API 依赖装配) | `B11` (S3 对象存储) | 数据流 (`data`) | 对象存储 |
| `B04` (Rust Axum 路由与中间件) | `B12` (角色资源服务) | 调用 (`call`) | characters routes |
| `B04` (Rust Axum 路由与中间件) | `B13` (角色卡服务) | 调用 (`call`) | character-cards routes |
| `B04` (Rust Axum 路由与中间件) | `B14` (形象 / 声音资源) | 调用 (`call`) | display-model / voice-pack routes |
| `B04` (Rust Axum 路由与中间件) | `B15` (Provider 配置 / 目录) | 调用 (`call`) | providers routes |
| `B04` (Rust Axum 路由与中间件) | `B19` (Chat WS v1 / v2) | 调用 (`call`) | WebSocket upgrade |
| `B19` (Chat WS v1 / v2) | `B17` (Hosted Chat WS Runtime) | 调用 (`call`) | 共用 runtime |
| `B19` (Chat WS v1 / v2) | `B18` (sendMessages / pullMessages RPC) | 调用 (`call`) | 注册 RPC |
| `B18` (sendMessages / pullMessages RPC) | `B16` (ChatService) | 调用 (`call`) | push / pull |
| `B18` (sendMessages / pullMessages RPC) | `B17` (Hosted Chat WS Runtime) | 调用 (`call`) | 消息广播 |
| `B17` (Hosted Chat WS Runtime) | `B10` (Redis) | 数据流 (`data`) | 跨实例广播 |
| `B16` (ChatService) | `B21` (会话与消息表) | 数据流 (`data`) | 持久消息 |
| `B12` (角色资源服务) | `B20` (核心资源表) | 数据流 (`data`) | 角色表族 |
| `B20` (核心资源表) | `B09` (PostgreSQL / 迁移) | 数据流 (`data`) | 关系数据 |
| `B21` (会话与消息表) | `B09` (PostgreSQL / 迁移) | 数据流 (`data`) | 关系数据 |
| `B14` (形象 / 声音资源) | `B11` (S3 对象存储) | 数据流 (`data`) | 二进制资源 |
| `B04` (Rust Axum 路由与中间件) | `B23` (OpenAI / Audio API) | 调用 (`call`) | 托管协议入口 |
| `B23` (OpenAI / Audio API) | `B22` (托管推理路由器) | 调用 (`call`) | 模型路由 |
| `B22` (托管推理路由器) | `B25` (用量 / 计费 / 请求日志) | 调用 (`call`) | 记录 / 计费 |
| `B04` (Rust Axum 路由与中间件) | `B24` (语音实时通路) | 调用 (`call`) | 实时音频 |
| `B07` (Better Auth / OIDC) | `B26` (账户删除协作) | 调用 (`call`) | 删除业务资源 |

---

### 专题 21：异步任务、资源生存期与事件流底座 (`lifetime`)

> **专题定位**：把取消、清理、排队、去重、有界保留与订阅需求拆成独立原语。它们解决执行可靠性，不拥有角色人格或对话记忆。

- **配套矢量图纸**：[职责组织图 (`21-lifetime-organization.svg`)](./diagrams/21-lifetime-organization.svg) ｜ [调用与数据流图 (`21-lifetime-relations.svg`)](./diagrams/21-lifetime-relations.svg) ｜ [Graphviz 组织图源](./diagrams/21-lifetime-organization.dot) ｜ [Graphviz 关系图源](./diagrams/21-lifetime-relations.dot)
- **直接涉及源码范围**：`crates/alia-stream-kit/README.md`

#### 读图与工程边界守则
- Task.done表示工作终态；Task.closed表示清理完成；两者不能混用。
- Task.finish不会abort signal；已交接出去的播放可以继续。
- Queue.clear只清待执行项，Scope关闭不等待子Scope清理完成。
- 图中的buffer→feed、demand→feed是可组合用法，不声称包内部自动连接。

#### 节点与状态归属登记表（共 23 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `B-TASK` | 任务与生命周期 | **Task · 一次工作** | `pub trait Task<T` | `crates/alia-stream-kit/src/task.rs:23` | 以统一句柄控制完成、取消、失败、清理。 | signal / done / closed；拥有自己的AbortController | Rust 核心实现 |
| `B-OUTCOME` | 任务与生命周期 | **Outcome · 三种结果** | `pub enum Outcome` | `crates/alia-stream-kit/src/task.rs:2` | finished / cancelled / failed；一次任务只settle一次。 | 终态与value/reason/error | Rust 核心实现 |
| `B-CREATE` | 任务与生命周期 | **createTask** | `pub fn createTask` | `crates/alia-stream-kit/src/task.rs:54` | 创建任务，关联父signal和可选timeout。 | cleanup栈、settled标志、timer | Rust 核心实现 |
| `B-RUN` | 任务与生命周期 | **runTask** | `pub fn runTask` | `crates/alia-stream-kit/src/task.rs:134` | 将异步work结果映射成Task终态。 | 由Task拥有终态；work接收Task | Rust 核心实现 |
| `B-WORST` | 任务与生命周期 | **worstOutcome** | `pub fn worstOutcome` | `crates/alia-stream-kit/src/task.rs:157` | 合并多个结果：failed优先于cancelled，再是finished。 | 纯计算，无持久状态 | Rust 核心实现 |
| `B-SCOPE` | 资源所有权 | **Scope · 资源生存期** | `pub trait Scope` | `crates/alia-stream-kit/src/scope.rs:7` | 统一终止signal并倒序运行defer清理。 | signal / cleanup栈 / closed Promise | Rust 核心实现 |
| `B-CHILD` | 资源所有权 | **scope.child()** | `child()` | `crates/alia-stream-kit/src/scope.rs:68` | 创建随父关闭的子Scope，也可先独立关闭。 | 子Scope拥有自己的清理；父不等待子清理完成 | Rust 核心实现 |
| `B-DEFER` | 资源所有权 | **scope.defer / close** | `pub fn close` | `crates/alia-stream-kit/src/scope.rs:46` | 关闭时只运行一次清理；关闭后新注册清理立即异步运行。 | 清理失败上报，不改变为领域业务失败 | Rust 核心实现 |
| `B-LATEST` | 调度策略 | **createLatest · 最新输入** | `pub fn createLatest` | `crates/alia-stream-kit/src/latest.rs:38` | 新key替换待执行输入并取消旧run；可debounce。 | 待执行input、当前AbortController与running Promise、key与计时器 | Rust 核心实现 |
| `B-QUEUE` | 调度策略 | **createQueue · FIFO** | `pub fn createQueue` | `crates/alia-stream-kit/src/queue.rs:15` | 依次处理每个item及它的handlers；暴露生命周期事件。 | queue数组 / draining / listeners | Rust 核心实现 |
| `B-QEVENT` | 调度策略 | **Queue Events** | `pub trait Events` | `crates/alia-stream-kit/src/queue.rs:6` | enqueue/dequeue/process/error/result/drain通知。 | 监听器归属于Queue实例 | Rust 核心实现 |
| `B-QCLEAR` | 调度策略 | **queue.clear()** | `pub fn clear` | `crates/alia-stream-kit/src/queue.rs:96` | 仅清空等待队列；当前handler的取消由上层负责。 | 不隐式中止正在运行的业务任务 | Rust 核心实现 |
| `B-PUSH` | 流桥接 | **createPushStream** | `pub fn createPushStream` | `crates/alia-stream-kit/src/stream.rs:15` | 将push生产者包装成ReadableStream；支持close/error/cancel。 | stream controller 与closed标志 | Rust 核心实现 |
| `B-READ` | 流桥接 | **readStream** | `export pub async fn readStream` | `crates/alia-stream-kit/src/stream.rs:57` | 依次读取并await handler，结束后释放reader锁。 | reader生命周期 | Rust 核心实现 |
| `B-BATCH` | 有界事件流 | **createBatcher** | `pub fn createBatcher` | `crates/alia-stream-kit/src/batcher.rs:41` | 达到maxItems或maxWaitMs即发送一批。 | 当前批、首项时刻、唯一定时器 | Rust 核心实现 |
| `B-BUFFER` | 有界事件流 | **RetentionBuffer** | `pub struct RetentionBuffer` | `crates/alia-stream-kit/src/retention-buffer.rs:34` | 按maxBytes/maxAgeMs保留新记录，批量报告丢弃。 | entries / head / bytes | Rust 核心实现 |
| `B-SEQ` | 有界事件流 | **SequenceTracker** | `pub struct SequenceTracker` | `crates/alia-stream-kit/src/sequence-tracker.rs:13` | 允许乱序到达；同一个seq只接收一次。 | 连续前缀与ahead: Set<number> | Rust 核心实现 |
| `B-WATER` | 有界事件流 | **Watermark** | `pub struct Watermark` | `crates/alia-stream-kit/src/watermark.rs:16` | 根据最新时间与allowedLatenessMs标记迟到记录。 | 最新事件时间 | Rust 核心实现 |
| `B-DEMAND` | 订阅与需求 | **DemandUnion** | `pub struct DemandUnion` | `crates/alia-stream-kit/src/demand-union.rs:22` | 合并所有读者需要的track；全量需求用*表示。 | 成员→需求集合，当前并集 | Rust 核心实现 |
| `B-LEASE` | 订阅与需求 | **createLeaseTable** | `pub fn createLeaseTable` | `crates/alia-stream-kit/src/lease-table.rs:26` | touch续约、expired检出失联所有者。 | key→最近续约时间 | Rust 核心实现 |
| `B-COOL` | 订阅与需求 | **createCooldown** | `pub fn createCooldown` | `crates/alia-stream-kit/src/cooldown.rs:21` | 每个key在窗口内最多获得一次行动资格。 | key→下一次允许时刻 | Rust 核心实现 |
| `B-FEED` | 订阅与需求 | **createCatchUpFeed** | `pub fn createCatchUpFeed` | `crates/alia-stream-kit/src/catch-up-feed.rs:55` | 先发历史和caughtUp，再无缝切到live。 | subscribers / delivery queues | Rust 核心实现 |
| `B-DELIVERY` | 订阅与需求 | **DeliveryStrategy** | `pub enum DeliveryStrategy` | `crates/alia-stream-kit/src/catch-up-feed.rs:8` | unbounded同步发送；sliding丢最旧；dropping丢新项。 | 订阅级capacity与排队策略 | Rust 核心实现 |

#### 调用、数据、事件与契约关系表（共 13 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `B-CREATE` (createTask) | `B-TASK` (Task · 一次工作) | 持有 / 归属 (`owns`) | 创建 |
| `B-RUN` (runTask) | `B-CREATE` (createTask) | 调用 (`call`) | 建立任务 |
| `B-TASK` (Task · 一次工作) | `B-OUTCOME` (Outcome · 三种结果) | 持有 / 归属 (`owns`) | settle一次 |
| `B-WORST` (worstOutcome) | `B-OUTCOME` (Outcome · 三种结果) | 数据流 (`data`) | 读取优先级 |
| `B-SCOPE` (Scope · 资源生存期) | `B-CHILD` (scope.child()) | 持有 / 归属 (`owns`) | 派生子生存期 |
| `B-SCOPE` (Scope · 资源生存期) | `B-DEFER` (scope.defer / close) | 持有 / 归属 (`owns`) | 登记/执行清理 |
| `B-SCOPE` (Scope · 资源生存期) | `B-CHILD` (scope.child()) | 生命周期 (`lifecycle`) | 父signal传播 |
| `B-QUEUE` (createQueue · FIFO) | `B-QEVENT` (Queue Events) | 事件 (`event`) | 发送通知 |
| `B-QUEUE` (createQueue · FIFO) | `B-QCLEAR` (queue.clear()) | 调用 (`call`) | 清空等待项 |
| `B-PUSH` (createPushStream) | `B-READ` (readStream) | 数据流 (`data`) | ReadableStream |
| `B-FEED` (createCatchUpFeed) | `B-DELIVERY` (DeliveryStrategy) | 持有 / 归属 (`owns`) | 每个订阅选择策略 |
| `B-BUFFER` (RetentionBuffer) | `B-FEED` (createCatchUpFeed) | 数据流 (`data`) | 宿主可提供backlog |
| `B-DEMAND` (DemandUnion) | `B-FEED` (createCatchUpFeed) | 数据流 (`data`) | 宿主按读者合并需求 |

---

### 专题 22：执行观测：生产、运输、采集、投影、回放 (`observability`)

> **专题定位**：从Agent或语音产生记录，经Web本地采集器或Rust 服务端 TraceHub，到时间轴、指标与导出。记录用于观测，不等于角色可回忆的内容。

- **配套矢量图纸**：[职责组织图 (`22-observability-organization.svg`)](./diagrams/22-observability-organization.svg) ｜ [调用与数据流图 (`22-observability-relations.svg`)](./diagrams/22-observability-relations.svg) ｜ [Graphviz 组织图源](./diagrams/22-observability-organization.dot) ｜ [Graphviz 关系图源](./diagrams/22-observability-relations.dot)
- **直接涉及源码范围**：`crates/alia-trace-kit/README.md`, `packages/stage-ui/src/composables/use-trace.ts`

#### 读图与工程边界守则
- Web各tab各有collector并通过BroadcastChannel交换数据；桌面Hub集中拥有collector。
- Collector保留有界历史与活跃pinned状态；关闭buffer不等于停止所有记录。
- source/epoch/seq与clock一起处理重启、乱序、重复和不同时钟域。

#### 节点与状态归属登记表（共 23 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `T-SINK` | 记录产生 | **TraceSink · 生产者接口** | `pub trait TraceSink` | `crates/alia-trace-kit/src/core/sink.rs:13` | emit写记录；wants判断轨道需求；now提供源时钟。 | 由调用方注入；无sink时使用noop | Rust 核心实现 |
| `T-SOURCE` | 记录产生 | **TraceSource** | `pub struct TraceSource` | `crates/alia-trace-kit/src/core/sink.rs:78` | 补source/epoch/seq/clock，启动时公告profile，stop后忽略新记录。 | seq、wanted、stopped；每个TraceSource实例一个epoch | Rust 核心实现 |
| `T-SPAN` | 记录产生 | **TraceSpan / beginSpan** | `pub struct TraceSpan` | `crates/alia-trace-kit/src/core/sink.rs:177` | span.begin、instant、update、end描述一段执行。 | span ID、时间与最终状态 | Rust 核心实现 |
| `T-PROFILE` | 记录产生 | **Agent / Voice Profile** | `agentProfile` | `crates/alia-trace-kit/src/profiles/agent.rs:82` | 规定语义名称、轨道、属性；Voice独立profile。 | 共享定义，不拥有会话业务状态 | Rust 核心实现 |
| `T-RECORD` | 记录产生 | **TraceRecord + Schema** | `export let traceRecordSchema` | `crates/alia-trace-kit/src/core/schema.rs:71` | 校验source、clock、span、state、series等记录变体。 | 跨边界数据契约 | Rust 核心实现 |
| `T-APP` | 宿主运输 | **getTraceSink / configureTransport** | `export function getTraceSink` | `packages/stage-ui/src/composables/use-trace.ts:184` | 连接Stage生产者与当前宿主transport；配置切换会结束旧source。 | 模块级source、transport、local、feedSource | WebUI 面板前端 |
| `T-LOCAL` | 宿主运输 | **浏览器 LocalRuntime** | `interface LocalRuntime` | `packages/stage-ui/src/composables/use-trace.ts:45` | 每tab有local collector；同源BroadcastChannel转发记录。 | collector / channel / wanted | WebUI 面板前端 |
| `T-WANTED` | 宿主运输 | **SharedWantedTracks** | `export class SharedWantedTracks` | `packages/stage-ui/src/composables/use-trace.ts:73` | 合并跨tab需求；定期公告并淘汰失联读者。 | DemandUnion、LeaseTable、own需求 | WebUI 面板前端 |
| `T-HUB` | 宿主运输 | **Rust IOTraceHub (alia-trace-kit)** | `pub struct IOTraceHub` | `crates/alia-trace-kit/src/hub.rs:28` | main集中接收renderer记录，维护collector并供应读者。 | 一个collector、tick timer、wanted与dump listeners | Rust 核心实现 |
| `T-COLLECT` | 采集与保存 | **TraceCollector** | `pub struct TraceCollector` | `crates/alia-trace-kit/src/collector/collector.rs:137` | 接收/验证/归一化时间、保留记录、服务读者。 | epochs、liveEpoch、buffer、pinned集合、series与需求 | Rust 核心实现 |
| `T-CLOCK` | 采集与保存 | **ClockGraph** | `pub struct ClockGraph` | `crates/alia-trace-kit/src/core/clock.rs:55` | 根据时钟配对快照拟合偏移/漂移；按分段转换时间。 | clock-pair图、快照与discontinuity分段 | Rust 核心实现 |
| `T-PIN` | 采集与保存 | **Pinned 状态集合** | `private readonly pinnedStarts` | `crates/alia-trace-kit/src/collector/collector.rs:151` | 保留活跃sources/streams、未结束spans、状态和定义。 | pinnedStarts / pinnedSpans / pinnedStates / definitions | Rust 核心实现 |
| `T-RING` | 采集与保存 | **SeriesRing** | `pub struct SeriesRing` | `crates/alia-trace-kit/src/core/series-ring.rs:18` | 高频序列保留与降采样，防止长期会话无限增长。 | 采样环、保留窗口 | Rust 核心实现 |
| `T-STREAM` | 采集与保存 | **stream-kit 原语** | `this.buffer =` | `crates/alia-trace-kit/src/collector/collector.rs:187` | 提供RetentionBuffer、SequenceTracker、Watermark、Lease、Demand、Feed。 | 各原语由collector实例拥有 | Rust 核心实现 |
| `T-FLIGHT` | 采集与保存 | **Flight recorder** | `private readonly flightWindow` | `crates/alia-trace-kit/src/collector/collector.rs:177` | 错误/过期span触发窗口转储；关闭常规buffer仍可保留故障窗口。 | flightWindow / cooldown / dumpedSpans | Rust 核心实现 |
| `T-FEED` | 投影与阅读 | **TraceFeedSource** | `pub trait TraceFeedSource` | `crates/alia-trace-kit/src/core/feed.rs:44` | 读取traceClock与subscribe；抽象本地/远程/文件来源。 | 订阅由读者持有并退订 | Rust 核心实现 |
| `T-READER` | 投影与阅读 | **TraceReader** | `pub struct TraceReader` | `crates/alia-trace-kit/src/projections/reader.rs:77` | 消费feed/records，维护索引并推进多个projection。 | clock图、source索引、projection states | Rust 核心实现 |
| `T-TIMELINE` | 投影与阅读 | **Timeline Projection** | `pub fn createTimelineProjection` | `crates/alia-trace-kit/src/projections/timeline.rs:491` | 把span/state/series映射为轨道、泳道与时间轴。 | timeline state / series rings | Rust 核心实现 |
| `T-LOG` | 投影与阅读 | **Event Log Projection** | `export function` | `crates/alia-trace-kit/src/projections/event-log.rs:10` | 产生可查询的事件日志投影视图。 | event log条目 | Rust 核心实现 |
| `T-METRIC` | 投影与阅读 | **Metrics Projection** | `export function` | `crates/alia-trace-kit/src/projections/metrics.rs:31` | 从属性与定义汇总指标样本。 | 按指标组织的samples | Rust 核心实现 |
| `T-VIEW` | 界面与回放 | **createTraceViewReader** | `pub fn createTraceViewReader` | `crates/alia-trace-kit/src/views/reader.rs:15` | 组合reader、timeline、eventLog、metrics，返回纯数据视图。 | 读者与各projection的组合 | Rust 核心实现 |
| `T-STORE` | 界面与回放 | **useIOTracerStore** | `export const useIOTracerStore` | `packages/stage-ui/src/stores/devtools/io-tracer.ts:29` | 提供start/stop/clear/selection；节流更新界面版本。 | reader、version、selection、subscription、clockAnchor | WebUI 面板前端 |
| `T-EXPORT` | 界面与回放 | **Export / file replay** | `export function` | `packages/stage-ui/src/stores/devtools/io-tracer-export.ts:16` | 把series重建为可导出的chunks，文件可供reader回放。 | 导出快照，不自动写入角色记忆 | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 26 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `T-SPAN` (TraceSpan / beginSpan) | `T-SINK` (TraceSink · 生产者接口) | 调用 (`call`) | 写入span记录 |
| `T-PROFILE` (Agent / Voice Profile) | `T-SOURCE` (TraceSource) | 数据流 (`data`) | 公告定义 |
| `T-SOURCE` (TraceSource) | `T-RECORD` (TraceRecord + Schema) | 数据流 (`data`) | 补元数据后发送 |
| `T-APP` (getTraceSink / configureTransport) | `T-SOURCE` (TraceSource) | 持有 / 归属 (`owns`) | 延迟创建/切换 |
| `T-APP` (getTraceSink / configureTransport) | `T-SINK` (TraceSink · 生产者接口) | 数据流 (`data`) | 返回TraceSink接口 |
| `T-SOURCE` (TraceSource) | `T-SINK` (TraceSink · 生产者接口) | 契约 / 配置 (`contract`) | implements |
| `T-APP` (getTraceSink / configureTransport) | `T-LOCAL` (浏览器 LocalRuntime) | 数据流 (`data`) | Web默认路径 |
| `T-APP` (getTraceSink / configureTransport) | `T-HUB` (Rust IOTraceHub (alia-trace-kit)) | 事件 (`event`) | 桌面transport路径 |
| `T-LOCAL` (浏览器 LocalRuntime) | `T-WANTED` (SharedWantedTracks) | 持有 / 归属 (`owns`) | 跨tab合并需求 |
| `T-LOCAL` (浏览器 LocalRuntime) | `T-COLLECT` (TraceCollector) | 持有 / 归属 (`owns`) | tab内采集 |
| `T-HUB` (Rust IOTraceHub (alia-trace-kit)) | `T-COLLECT` (TraceCollector) | 持有 / 归属 (`owns`) | main集中采集 |
| `T-RECORD` (TraceRecord + Schema) | `T-COLLECT` (TraceCollector) | 数据流 (`data`) | ingestUntrusted验证 |
| `T-COLLECT` (TraceCollector) | `T-CLOCK` (ClockGraph) | 持有 / 归属 (`owns`) | 时钟归一化 |
| `T-COLLECT` (TraceCollector) | `T-PIN` (Pinned 状态集合) | 持有 / 归属 (`owns`) | 保存活跃状态 |
| `T-COLLECT` (TraceCollector) | `T-RING` (SeriesRing) | 持有 / 归属 (`owns`) | 保存序列 |
| `T-COLLECT` (TraceCollector) | `T-STREAM` (stream-kit 原语) | 持有 / 归属 (`owns`) | 生命周期/保留原语 |
| `T-COLLECT` (TraceCollector) | `T-FLIGHT` (Flight recorder) | 持有 / 归属 (`owns`) | 故障转储 |
| `T-COLLECT` (TraceCollector) | `T-FEED` (TraceFeedSource) | 数据流 (`data`) | 历史+caughtUp+live |
| `T-FEED` (TraceFeedSource) | `T-READER` (TraceReader) | 数据流 (`data`) | feed messages |
| `T-READER` (TraceReader) | `T-TIMELINE` (Timeline Projection) | 调用 (`call`) | 推进投影 |
| `T-READER` (TraceReader) | `T-LOG` (Event Log Projection) | 调用 (`call`) | 推进投影 |
| `T-READER` (TraceReader) | `T-METRIC` (Metrics Projection) | 调用 (`call`) | 推进投影 |
| `T-VIEW` (createTraceViewReader) | `T-READER` (TraceReader) | 持有 / 归属 (`owns`) | 组合 |
| `T-STORE` (useIOTracerStore) | `T-VIEW` (createTraceViewReader) | 调用 (`call`) | 创建与读取 |
| `T-STORE` (useIOTracerStore) | `T-EXPORT` (Export / file replay) | 数据流 (`data`) | JSONL导出 |
| `T-WANTED` (SharedWantedTracks) | `T-SOURCE` (TraceSource) | 调用 (`call`) | setWanted |

---

### 专题 23：项目唯一内置前端：WebUI 控制台面板（测试沙盒对话、配置中心与插件扩展管理） (`webui-console`)

> **专题定位**：项目本身只存在这一个内置 WebUI 面板（apps/webui），通过 REST + SSE 与 Rust 本机核心实例实时同步本地持久状态。面板按“工作台（运行概览、测试沙盒直接对话）、数字个体（身份与人格、记忆管理）、连接与能力（接入渠道、模型服务［含最多 6 路并发探活与日常对话/后台活动/内部推理/目标决策四维分工］、扩展管理、技能管理）、配置与运行（配置文件、自动化任务、数据与日志、权限与审批、系统设置）”组织，使一切核心配置、直接对话验证与插件扩展均可在 WebUI 内完成。

- **配套矢量图纸**：[职责组织图 (`23-webui-console-organization.svg`)](./diagrams/23-webui-console-organization.svg) ｜ [调用与数据流图 (`23-webui-console-relations.svg`)](./diagrams/23-webui-console-relations.svg) ｜ [Graphviz 组织图源](./diagrams/23-webui-console-organization.dot) ｜ [Graphviz 关系图源](./diagrams/23-webui-console-relations.dot)
- **直接涉及源码范围**：`apps/webui`, `packages/stage-ui`, `packages/ui`, `packages/ui-trace`, `packages/ui-timeline`, `packages/timeline-canvas`

#### 读图与工程边界守则
- 项目只内置 apps/webui 一个前端面板，其他一切外部渠道、2D/3D 形象渲染、直播、游戏均通过“接入渠道”与“扩展管理”以插件方式扩展。
- 工作台中的“测试沙盒”直接在 WebUI 面板内提供完整的对话交互体验，方便项目落地后开发者无需部署任何外部渠道即可直接验证对话、记忆与工具调用。
- 模型服务支持按任务分工（日常对话 / 后台活动 / 内部推理 / 目标决策）指派不同模型，且切换模型时数字个体的身份与记忆持续保留。

#### 节点与状态归属登记表（共 14 个节点）

| 节点 ID | 职责分组 | 名称 | 导出符号 (`symbol`) | 源码路径与行号 | 职责边界 | 状态与生命周期归属 | 架构分层 |
|---|---|---|---|---|---|---|---|
| `UI-ROOT` | 工作台 (Workbench) | **WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue** | `WebUIConsoleApp` | `apps/webui/src/App.vue:1` | 项目唯一的内置前端面板：顶部维护本机实例授权态与 REST + SSE 实时同步连接，左侧提供四大分组导航（工作台、数字个体、连接与能力、配置与运行）。 | REST + SSE 实时同步连接态 | WebUI 面板前端 |
| `UI-OVERVIEW` | 工作台 (Workbench) | **运行概览面板 · apps/webui/src/pages/workbench/overview.vue** | `WorkbenchOverviewPage` | `apps/webui/src/pages/workbench/overview.vue:1` | 汇总展示本机实例运行状态、当前激活的四维模型分工、已启用接入渠道/扩展插件数量与近期活动指标。 | 概览聚合视图态 | WebUI 面板前端 |
| `UI-SANDBOX` | 工作台 (Workbench) | **测试沙盒（内置直接对话体验台） · apps/webui/src/pages/workbench/sandbox.vue** | `WorkbenchSandboxPage` | `apps/webui/src/pages/workbench/sandbox.vue:1` | 在项目 WebUI 面板内直接实现与数字个体的多模态/语音流式对话功能，实时检视日常对话/内部推理/目标决策输出、记忆命中与工具调用，利于项目落地与开发者开箱直接体验。 | 沙盒会话与流式消息态 | WebUI 面板前端 |
| `UI-IDENTITY` | 数字个体 (Digital Individual) | **身份与人格配置页 · apps/webui/src/pages/individual/identity.vue** | `IdentityAndPersonaPage` | `apps/webui/src/pages/individual/identity.vue:1` | 配置数字个体的核心身份、人设提示词與模块继承覆盖（对接专题 03 alia-core-character），切换模型时身份持续保留。 | 身份与人设编辑草稿态 | WebUI 面板前端 |
| `UI-MEMORY` | 数字个体 (Digital Individual) | **记忆管理与笔记页 · apps/webui/src/pages/individual/memory.vue** | `MemoryManagementPage` | `apps/webui/src/pages/individual/memory.vue:1` | 检索、编辑、固定或清理长期记忆条目（MemoryEntry）与角色持久笔记（Notebook，对接专题 09/10/11），切换模型时记忆持续保留。 | 记忆筛选与分页视图态 | WebUI 面板前端 |
| `UI-CHANNELS` | 连接与能力 (Connections & Capabilities) | **接入渠道管理页 · apps/webui/src/pages/capabilities/channels.vue** | `ConnectedChannelsPage` | `apps/webui/src/pages/capabilities/channels.vue:1` | 配置与监控通过 alia-channel-gateway（专题 19）接入的所有外部可扩展渠道（如外部 IM、独立 WebChat、语音终端等）。 | 已接入渠道列表与连接状态 | WebUI 面板前端 |
| `UI-MODELS` | 连接与能力 (Connections & Capabilities) | **模型服务与四维使用分工页 · apps/webui/src/pages/capabilities/models.vue** | `ModelServicesPage` | `apps/webui/src/pages/capabilities/models.vue:1` | 管理已接入模型服务商地址与密钥、获取/手动添加模型、执行批量并发探活测试（每次最多 6 个并行调用并计入健康度），并配置 Alia 使用分工（日常对话 / 后台活动 / 内部推理 / 目标决策，对接专题 07）。 | 服务商列表、探活进度与四维分工映射 | WebUI 面板前端 |
| `UI-EXTENSIONS` | 连接与能力 (Connections & Capabilities) | **扩展管理页（插件中心） · apps/webui/src/pages/capabilities/extensions.vue** | `ExtensionsManagementPage` | `apps/webui/src/pages/capabilities/extensions.vue:1` | 管理通过 alia-extension-host 与 alia-plugin-protocol（专题 17/18）加载的所有外部插件扩展（含安装目录、启停开关、Kit 插槽状态与动态工具租约）。 | 插件注册表与启停配置态 | WebUI 面板前端 |
| `UI-SKILLS` | 连接与能力 (Connections & Capabilities) | **技能管理页 · apps/webui/src/pages/capabilities/skills.vue** | `SkillsManagementPage` | `apps/webui/src/pages/capabilities/skills.vue:1` | 管理数字个体可调用的内建与扩展工具定义、技能配方（Recipe，对接专题 08/09）及启用策略。 | 技能与工具目录视图态 | WebUI 面板前端 |
| `UI-PROFILES` | 配置与运行 (Configuration & Runtime) | **配置文件与自动化任务页 · apps/webui/src/pages/runtime/profiles-automation.vue** | `ProfilesAndAutomationPage` | `apps/webui/src/pages/runtime/profiles-automation.vue:1` | 管理多套运行时配置文件（Config Profiles）导入导出，以及后台自动化任务（Automation，对接专题 09/11）的定时与触发规则。 | 配置快照与定时任务列表 | WebUI 面板前端 |
| `UI-LOGS` | 配置与运行 (Configuration & Runtime) | **数据与日志（Trace 诊断）页 · apps/webui/src/pages/runtime/data-logs.vue** | `DataAndLogsTracePage` | `apps/webui/src/pages/runtime/data-logs.vue:1` | 内嵌 packages/ui-trace、packages/ui-timeline 与 packages/timeline-canvas（对接专题 22 observability），可视化检视全链路 Trace 时间轴、会话持久数据与 JSONL 录制回放。 | 时间轴视口与日志过滤态 | WebUI 面板前端 |
| `UI-APPROVALS` | 配置与运行 (Configuration & Runtime) | **权限与审批及系统设置页 · apps/webui/src/pages/runtime/approvals-settings.vue** | `ApprovalsAndSettingsPage` | `apps/webui/src/pages/runtime/approvals-settings.vue:1` | 管理扩展插件权限决议（对接专题 17 permissionResolver）、高风险工具审批、本机实例授权（对接专题 20 alia-auth-service）与本地持久存储设置。 | 待审批队列与系统设置态 | WebUI 面板前端 |
| `UI-TIMELINE-CANVAS` | 配置与运行 (Configuration & Runtime) | **Canvas 时间轴绘制原语 · packages/timeline-canvas/src/index.ts** | `@alia/timeline-canvas` | `packages/timeline-canvas/src/index.ts:1` | 为数据与日志页提供高帧率 Canvas 2D 时间标尺与 Span 块绘制能力。 | Canvas 视口变换缓存 | WebUI 面板前端 |
| `UI-TIMELINE-CTRL` | 配置与运行 (Configuration & Runtime) | **时间轴缩放与游标控制 · packages/ui-timeline/src/index.ts** | `@alia/ui-timeline` | `packages/ui-timeline/src/index.ts:1` | 管理全链路诊断时间轴的缩放（Zoom）、平移（Pan）与游标定位。 | viewportRange / cursorTime | WebUI 面板前端 |

#### 调用、数据、事件与契约关系表（共 18 条有向边）

| 起点 (`from`) | 终点 (`to`) | 关系类型 (`kind`) | 传递载荷 / 交互语义 (`label`) |
|---|---|---|---|
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-OVERVIEW` (运行概览面板 · apps/webui/src/pages/workbench/overview.vue) | 持有 / 归属 (`owns`) | 路由切换与SSE实时状态分发 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-SANDBOX` (测试沙盒（内置直接对话体验台） · apps/webui/src/pages/workbench/sandbox.vue) | 持有 / 归属 (`owns`) | 挂载内置直接对话测试沙盒 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-IDENTITY` (身份与人格配置页 · apps/webui/src/pages/individual/identity.vue) | 持有 / 归属 (`owns`) | 挂载身份与人格配置 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-MEMORY` (记忆管理与笔记页 · apps/webui/src/pages/individual/memory.vue) | 持有 / 归属 (`owns`) | 挂载长期记忆与笔记管理 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-CHANNELS` (接入渠道管理页 · apps/webui/src/pages/capabilities/channels.vue) | 持有 / 归属 (`owns`) | 挂载外部接入渠道配置 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-MODELS` (模型服务与四维使用分工页 · apps/webui/src/pages/capabilities/models.vue) | 持有 / 归属 (`owns`) | 挂载模型服务与四维使用分工配置 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-EXTENSIONS` (扩展管理页（插件中心） · apps/webui/src/pages/capabilities/extensions.vue) | 持有 / 归属 (`owns`) | 挂载外部插件扩展管理 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-SKILLS` (技能管理页 · apps/webui/src/pages/capabilities/skills.vue) | 持有 / 归属 (`owns`) | 挂载技能与Recipe管理 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-PROFILES` (配置文件与自动化任务页 · apps/webui/src/pages/runtime/profiles-automation.vue) | 持有 / 归属 (`owns`) | 挂载配置文件与自动化任务管理 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-LOGS` (数据与日志（Trace 诊断）页 · apps/webui/src/pages/runtime/data-logs.vue) | 持有 / 归属 (`owns`) | 挂载数据与日志Trace诊断面板 |
| `UI-ROOT` (WebUI 控制台根壳与实时同步栏 · apps/webui/src/App.vue) | `UI-APPROVALS` (权限与审批及系统设置页 · apps/webui/src/pages/runtime/approvals-settings.vue) | 持有 / 归属 (`owns`) | 挂载权限审批与系统设置 |
| `UI-MODELS` (模型服务与四维使用分工页 · apps/webui/src/pages/capabilities/models.vue) | `UI-SANDBOX` (测试沙盒（内置直接对话体验台） · apps/webui/src/pages/workbench/sandbox.vue) | 契约 / 配置 (`contract`) | 按日常对话/内部推理/目标决策分工驱动沙盒 |
| `UI-IDENTITY` (身份与人格配置页 · apps/webui/src/pages/individual/identity.vue) | `UI-SANDBOX` (测试沙盒（内置直接对话体验台） · apps/webui/src/pages/workbench/sandbox.vue) | 数据流 (`data`) | 注入持续保留的身份与人设提示词 |
| `UI-MEMORY` (记忆管理与笔记页 · apps/webui/src/pages/individual/memory.vue) | `UI-SANDBOX` (测试沙盒（内置直接对话体验台） · apps/webui/src/pages/workbench/sandbox.vue) | 数据流 (`data`) | 注入持续保留的长期记忆与笔记上下文 |
| `UI-EXTENSIONS` (扩展管理页（插件中心） · apps/webui/src/pages/capabilities/extensions.vue) | `UI-SKILLS` (技能管理页 · apps/webui/src/pages/capabilities/skills.vue) | 事件 (`event`) | 插件动态注册ToolKit与技能 |
| `UI-APPROVALS` (权限与审批及系统设置页 · apps/webui/src/pages/runtime/approvals-settings.vue) | `UI-EXTENSIONS` (扩展管理页（插件中心） · apps/webui/src/pages/capabilities/extensions.vue) | 契约 / 配置 (`contract`) | 提供扩展权限审批决议 |
| `UI-LOGS` (数据与日志（Trace 诊断）页 · apps/webui/src/pages/runtime/data-logs.vue) | `UI-TIMELINE-CTRL` (时间轴缩放与游标控制 · packages/ui-timeline/src/index.ts) | 持有 / 归属 (`owns`) | 组合时间轴缩放与游标控制器 |
| `UI-TIMELINE-CTRL` (时间轴缩放与游标控制 · packages/ui-timeline/src/index.ts) | `UI-TIMELINE-CANVAS` (Canvas 时间轴绘制原语 · packages/timeline-canvas/src/index.ts) | 调用 (`call`) | 驱动Canvas高帧率时间轴重绘 |

---

## 第四部分：24 条原版核心架构边界与核对发现（Findings & Architectural Invariants）

### 01. Automation文档与实现不一致

在该commit的packages与apps源码搜索checkAutomations/isAutoRunRecipe/builtIn_armRecipe/modelTimed/interface Automation无命中，仅README与ADR声明；Recipe类型没有automation/modelTimed字段。不能把它们作为已实现调度器。

- **关联源码定位**：`crates/alia-core-agent/src/runtime/recipe.rs:42` ｜ `crates/alia-core-agent/src/lib.rs:95` ｜ `docs/ai/adr/2026-10-10-skills-automations-and-memory.md:33`

### 02. Responses轮次上限文档过期

README声称最多10步；源码Chat和Responses均用stopWhen:not(hasToolCall())，这两个adapter没有设置固定轮次上限。

- **关联源码定位**：`crates/alia-core-agent/src/runtime/chat-completions.rs:113` ｜ `crates/alia-core-agent/src/runtime/responses.rs:264` ｜ `crates/alia-core-agent/src/lib.rs:69`

### 03. 后台任务状态以源码为准

实际状态仅running/done/failed/interrupted；ADR包含armed/stopped及自动定时能力，不能直接当当前状态机。

- **关联源码定位**：`packages/stage-ui/src/types/chat-session.ts:13` ｜ `docs/ai/adr/2026-10-10-skills-automations-and-memory.md:48`

### 04. 跨会话并行、同会话FIFO

sendQueues/activeSends是以sessionId为key的Map；不同session并行，activeSendSessionId只是显示镜像。

- **关联源码定位**：`crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:526` ｜ `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:1414`

### 05. 图片与音频的逐步适配程度不同

audio在prepareConversation随每步目标重新适配；图像决策在streamWithStageAdapters按首步模型选择，源码NOTICE标明尚待迁移。

- **关联源码定位**：`packages/stage-ui/src/stores/chat.ts:691` ｜ `packages/stage-ui/src/stores/chat.ts:886`

### 06. 运行上下文桶未统一自动过期

registry的400上限只裁剪contextHistory；activeContexts append-self桶没有同样上限。performSend还有TODO清理断开服务的stale runtime context。

- **关联源码定位**：`crates/alia-core-agent/src/runtime/context-registry.rs:137` ｜ `crates/alia-core-agent/src/runtime/chat-orchestrator-runtime.rs:726`

### 07. 内存工具写入受persona限制而非概念化向量记忆

memory实际为索引+UUIDv7事实条目，IDB持久化与同源BroadcastChannel同步；没有在此路径实现embedding/RAG向量召回。

- **关联源码定位**：`packages/stage-ui/src/stores/memory.ts:85` ｜ `crates/alia-storage/src/repos/memory.repo.rs:51`

### 08. core-character不是已完成的角色大脑

此提交core-character/src只有avatar-model.ts与index.ts，提供形象引用和类型契约。personality是Stage角色卡中参与提示词的文本；不能据包名把人格、情绪、记忆等实现都画在core-character之下。

- **关联源码定位**：`crates/alia-core-character/src/index.rs:1` ｜ `crates/alia-core-character/src/avatar-model.rs:1` ｜ `crates/alia-core-character/src/inheritance.rs:140`

### 09. 当前ACT主路径需替换旧队列路径

流式输出中的内联动作标记（ACT）通过 StreamingControl.onSignal 与 normalizeActPayload 归一化为标准 EmotionPayload 信号，分发至 WebUI 测试沙盒与外部已挂载的扩展通道，核心运行时不耦合任何具体 2D/3D 渲染器。

- **关联源码定位**：`apps/webui/src/components/sandbox/SandboxHost.vue:319` ｜ `packages/stage-ui/src/stores/ai/chat-llm/streaming-control.ts:44` ｜ `packages/stage-ui/src/composables/queues.ts:9`

### 10. 语音资源所有权与输入阶段必须分开

VoiceController的audio和playback由外部注入，close不关闭共享资源；每个VoiceResponse拥有独立PlaybackGroup。输入end进入finalizing时已释放准入，因此下一句capturing可与上一句最终处理重叠。

- **关联源码定位**：`crates/alia-core-agent/src/voice/controller.rs:30` ｜ `crates/alia-core-agent/src/voice/controller.rs:371` ｜ `crates/alia-core-agent/src/voice/output/response.rs:484` ｜ `crates/alia-core-agent/src/voice/input/attempt.rs:78`

### 11. Stage尚未接入结束判断的会话上下文

SpeechInputPorts支持conversationContext，但此版本useVoiceStore构造VoiceController时没有传入该port；核心回退为revision=0、messages=[]。不能把接口存在画成Stage已接入完整近期会话。

- **关联源码定位**：`packages/stage-ui/src/stores/voice.ts:116` ｜ `crates/alia-core-agent/src/voice/controller.rs:84` ｜ `crates/alia-core-agent/src/voice/input/ports.rs:22`

### 12. 说话人证据尚不进入语言模型prompt

说话人候选分数能进入TranscriptDocument与trace，但voice-drafts明确将speaker evidence排除在prompt之外，理由是尚无校准阈值并等待speaker gate。不能将其画成可靠身份识别已经影响推理。

- **关联源码定位**：`packages/stage-ui/src/composables/voice-drafts.ts:27` ｜ `crates/alia-core-agent/src/voice/input/types.rs:29`

### 13. 两种TTS输出的标记时序不同

普通分段TTS与双向流式TTS是不同路径。分段路径的token-special按前序音频排队到达；流式provider没有同样的段落，特殊标记在文本写入时到达。不能画成完全相同的表情同步保证。

- **关联源码定位**：`crates/alia-core-agent/src/voice/output/response.rs:39` ｜ `crates/alia-core-agent/src/voice/output/response.rs:97` ｜ `packages/stage-ui/src/stores/modules/speech.ts:710`

### 14. 角色speech类型字段不等于全部已按角色生效

AliaExtension含pitch/rate/ssml等字段；createOutput确实快照角色provider/model/voice，但其pitch和ssmlEnabled仍从全局Speech Store读取。应分别标注契约字段与当前消费路径。

- **关联源码定位**：`crates/alia-core-character/src/card.rs:39` ｜ `packages/stage-ui/src/stores/modules/speech.ts:686` ｜ `packages/stage-ui/src/stores/modules/speech.ts:701`

### 15. Spark与主chat的配置解析路径不同

Character Orchestrator的processSparkNotify读取全局Consciousness activeProvider/activeModel，同时读取当前Character的systemPrompt；这不是主chat与语音输出按session所属角色getModules的同一路径。

- **关联源码定位**：`packages/stage-ui/src/stores/character/orchestrator/store.ts:122` ｜ `apps/webui/src/components/sandbox/SandboxHost.vue:453` ｜ `crates/alia-core-character/src/inheritance.rs:1312`

### 16. Notebook持久化与spark:emit仍有实现边界

已读CharacterNotebook实现将entries/tasks保存为内存ref，未见该Store的持久化适配，不能称作已实现的长期记忆数据库。Character Orchestrator的spark:emit处理器当前直接返回undefined。

- **关联源码定位**：`packages/stage-ui/src/stores/character/notebook.ts:33` ｜ `packages/stage-ui/src/stores/character/orchestrator/store.ts:265`

### 17. 视觉并发队列仅在单个renderer窗口内

VisionReadQueue按provider管理semaphore，但每个窗口有自己的queue。源码明确提示多窗口可以同时读同一provider，尚不能把它标为全应用统一并发闸门。

- **关联源码定位**：`crates/alia-vision-pipeline/src/vision-read-queue.rs:29`

### 18. 统一远程扩展宿主未形成完整闭环

统一远程扩展宿主未形成完整闭环：底层 WebSocket Eventa 包装存在，但 Node/Web plugin-host transport 工厂多分支直接 throw，setupRemotePluginScope 为空。

- **关联源码定位**：`crates/alia-extension-host/src/channels/remote/websocket/index.rs:15` ｜ `crates/alia-extension-host/src/plugin-host/runtimes/transport/index.rs:22` ｜ `crates/alia-extension-host/src/plugin-host/runtimes/web/index.rs:21` ｜ `crates/alia-extension-host/src/plugin/remote/index.rs:7`

### 19. Extension-hosted Kit 当前登记/提交/撤回 Provider，但

Extension-hosted Kit 当前登记/提交/撤回 Provider，但消费 methods/events 属 README 的 Phase 4。resolveKitApi 只调用 getHostClientFactory。

- **关联源码定位**：`crates/alia-extension-host/Cargo.toml:77` ｜ `crates/alia-extension-host/src/plugin-host/core.rs:568` ｜ `crates/alia-extension-host/src/plugin-host/hosted-kits.rs:701`

### 20. ExtensionHost 默认没有 permissionResolver 时 re

ExtensionHost 默认没有 permissionResolver 时 resolvedGrant 回退到 manifest.permissions；权限契约存在不代表交互式审批或系统沙箱。

- **关联源码定位**：`crates/alia-extension-host/src/plugin-host/core.rs:319` ｜ `crates/alia-server/src/bootstrap.rs:385`

### 21. server-channel 路由模块事件，不拥有会话/模型；Hosted Chat

server-channel 路由模块事件，不拥有会话/模型；Hosted Chat WS 独立负责 push/pull 与 Redis 广播。

- **关联源码定位**：`crates/alia-channel-gateway/README.md:3` ｜ `crates/alia-server/src/routes/chat-ws/rpc.rs:37` ｜ `crates/alia-server/src/routes/chat-ws/runtime.rs:16`

### 22. ORM characterRelations 是表间关联；不能用 Relations

ORM characterRelations 是表间关联；不能用 Relations 命名证明心理关系/羁绊模型。

- **关联源码定位**：`crates/alia-server/src/schemas/characters.rs:184`

### 23. 共享协议另有 extension-protocol 与 plugin-protoco

共享协议另有 extension-protocol 与 plugin-protocol；桌面作者能力由 alia-plugin-protocol (Kits) 提供，旧图仅画 plugin-sdk 不足以解释实际层级。

- **关联源码定位**：`crates/alia-plugin-protocol/src/index.rs:25` ｜ `crates/alia-plugin-protocol/README.md:3` ｜ `crates/alia-plugin-protocol/src/kits/widget/index.rs:68`

### 24. 唯一内置 WebUI 面板（apps/webui）与“万物皆插件扩展”边界

项目本身只内置一个 WebUI 控制台面板（apps/webui）与一套 Rust 核心后端（crates/*）。WebUI 面板提供测试沙盒直接对话、身份与记忆配置、模型服务与四维使用分工（日常对话/后台活动/内部推理/目标决策）、接入渠道与扩展管理、自动化与审批；其他一切外部聊天客户端（如独立 WebChat/IM）、2D/3D 形象渲染（Live2D/VRM）、直播、游戏均属于通过 ChannelGateway 与 ExtensionHost 挂载的可扩展事物。

- **关联源码定位**：`apps/webui/src/App.vue:1` ｜ `crates/alia-extension-host/src/plugin-host/core.rs:1`

---

## 第五部分：9 条原版核心端到端运行次序（Execution Sequences）

### 01. 用户发送→模型多轮→最终保存

1. Stage startSend以(sessionId,messageId)去重，等待memory.ready/recipes.ready。
2. prepareSend加载session，从其character member取有效Provider/Model，并存附件引用。
3. runtime.submit捕获session generation，进入该session独立FIFO。
4. performSend调用commitUserMessage；存储回执完成accepted，重复id到此停止。
5. 构建身份+记忆索引+保留会话turn+writer context快照；生成Conversation。
6. Stage adapter展开媒体bytes，按当前能力做image/audio请求适配。
7. core streamFrom先resolveStep→generation协议选择→prepareConversation→投影。
8. Chat/Responses adapter调用模型，模型tool call执行本地工具；tool result回入下一轮。
9. prepareStep每次重新解析目标配置；scope改变则RequestSwitch携partialTurn开新流。
10. text delta经marker parser/categorizer到可见slices与语音hooks。
11. 全部SDK steps及消费者完成，onGeneratedTurn给出所有settled rounds。
12. commitAssistantMessage保存完整回复；done完成，清理activeSends，下一队列项执行。

### 02. 长期记忆写入与回忆

1. 本轮session确定persona，sessionSystemPrompt注入own+general索引，只有id和description。
2. 模型调用builtIn_readMemory(id)读取可见正文；读取不会自动把全部正文加入每次请求。
3. builtIn_writeMemory绑定运行persona；repo在navigator.locks内读取当前记录。
4. 只有本persona自己的id可原位更新；其他id新增UUIDv7记录，不覆盖general。
5. IDB写入后store reload并BroadcastChannel通知其他窗口重读。
6. builtIn_forgetMemory同样检查persona并写tombstone；只有Settings可makeGeneral或删general。

### 03. 后台Recipe任务与通知

1. owner关键词或builtIn_useRecipe选择enabled+approved的background Recipe。
2. startRecipe为同persona创建hidden新session，记录parentSessionId/recipeId/task=running。
3. startSend(background:true)进入独立队列并行执行，且speaks=false。
4. 任务完成将状态从running改done/failed；只有成功状态迁移才允许发送结果notice。
5. 父会话收到带source的notice，主Agent再决定回应或安静；外部scene不接该notice。
6. 用户stop改interrupted并abort；换leader后遗留任务也标interrupted，不自动恢复。
7. 收尾保留最近3个finished hidden sessions，dismiss或超过数量清理旧session。

### 04. 语音输入到回答

1. 界面控制或VAD规则：给出BeginSpeechInput：session、start策略与明确interruptTurns。
2. VoiceController / SpeechInputAttempt：创建attempt，等待音源或目标静音；按策略保留pre-roll并建立capture。
3. StreamingTranscriber / SpeechInput：转写适配消费PCM并输出revision，SpeechInput更新raw与corrected文档。
4. Voice插件与VoicePolicy：watch可推测修正文案；VAD、停顿与结束判断通过events和readAt revision产生seal命令。
5. SpeechInputAttempt / finalize：end停止捕获并释放准入；转写结束后串行完成finalize，构造SpeechSubmission。
6. VoiceStore / useVoiceDrafts：按输入工作流返回composer文本、保留voice draft或调用chat.submit。
7. VoiceResponse / SpeechStream：本turn的文本进入有序producer，保留literal与special的相对位置。
8. SpeechOutput / PlaybackGroup：普通分段TTS或流式TTS产生音频，在响应独立播放组内按顺序播放。
9. Stage / renderer / VoiceResponse：输出事件更新字幕、口型和ACT表达；finish等待producer及实际播放完成。

### 05. 用户打断的静音与记录闭环

1. 新输入或显式控制：提交明确的interruptTurns与cause，不能把所有音频当作默认目标。
2. VoiceController / VoiceResponse：按turn去重并关闭目标响应接纳，先给目标PlaybackGroup安装fade，再中止producer。
3. PlaybackGroup：丢弃待播放clip，停止活动clip，等待实际静音并返回PlaybackReceipt。
4. Interruption.silenced / SpeechInputAttempt：silenced完成后允许按准入策略继续新输入；这只证明声音已停。
5. notifyInterruption / 宿主Chat：记录eventId、turn、cause及播放回执；done独立报告记录成功、排队或失败，失败重试复用eventId。

### 06. 模型内联表达指令（ACT）解析与语义信号分发次序

1. 模型输出 / SpeechOutput：流中出现 ACT 特殊标记，onSpecial 携带 turnId 交给流式控制入口处理。
2. StreamingControl：dispatchWith 匹配 createStreamingControlParser 解析器并发布 act signal，调用 normalizeActPayload 归一化。
3. ExpressionBus.onSignal：提取规范化的 Emotion / EmotionPayload 语义标签并进入信号分发队列（emotionsQueue），同步通知 WebUI 测试沙盒展示并通过模块通道广播给外部订阅插件。

### 07. Extension Host 内部组织与状态所有权 · 执行次序

1. start → 兼容性检查 → loadExtensionFor → nextSessionIdentity → grants / child container / Provider reservations
2. 所需 Kit 缺失 → pending → 监听兼容 Provider → setting-up；依赖满足则直接 setup
3. setup 成功 → transaction.commit → ready；失败 → failed → cleanup → stopped
4. stop → 撤 Provider / rollback → dispose child container → bindings → modules → root subscriptions → remove session
5. reload → stop 成功 → fresh start / 新 session id

### 08. 模块 WebSocket 通道：连接、身份、路由和资源 · 执行次序

1. Client.connect → transport open → 可选 token auth → module announce → ACK / registry contains self → ready
2. 业务消息 → parseEvent → auth check → server sender stamp → route middleware → consumer selector 或 broadcast → peer
3. asset reference → getAsset request id → server 转发 → Stage 按 sender 校验并取数据 → peer:<id> 响应 → parentId 匹配

### 09. Hosted Backend：产品资源、认证、同步与模型网关 · 执行次序

1. 客户端认证 → Auth/OIDC 发 token → API resolveRequestAuth/JWKS → 用户 principal
2. sendMessages → request parse → ChatService.pushMessages → DB seq → pull inserted rows → local connection registry + Redis broadcast
3. pullMessages(afterSeq) → 授权 / DB 查询 → 增量消息 → 客户端重建会话投影
