# AGENTS.md — Principia Agent 工程开发施工手册

## 1. 权威事实源与查阅顺序

本仓库是通用 AI 数字个体项目的唯一架构设计规范库（Principia）。任何开发 Agent 在规划任务、创建模块、实现接口或编写测试前，必须严格按以下顺序核对架构基线：

1. **先查机器可读唯一事实源 [architecture-data.json](./architecture-data.json) 或全量符号索引 [source-index.csv](./source-index.csv)**：
   - 确认目标能力归属哪个工作空间（`inventory` 共 **22 个核心工作空间**：14 个 Rust 核心 Crates `crates/*` + 1 个唯一内置 WebUI 面板 `apps/webui` + 6 个前端基础/诊断包 `packages/*`）。
   - 确认目标符号位于哪个专题（`sections` 共 **23 个核心专题**）、哪个源码文件路径（`path`）与导出符号名（`symbol`）。
   - 核对该节点的 **状态与生命周期归属（`state`）**、**上游调用方（`edges` 入边）** 与 **下游被调方（`edges` 出边）**。
2. **再查全景规范 [Principia-Architecture-Specification.md](./Principia-Architecture-Specification.md) 与交互图纸 [Principia-Architecture-Atlas.html](./Principia-Architecture-Atlas.html)**：
   - 核对对应专题的 **读图与工程边界守则（`notes`）**、**24 条架构边界守则（`findings`）** 以及 **9 条端到端运行次序（`sequences`）**。

---

## 2. 核心架构铁律（严禁违反）

1. **原版核心能力保留 + 可扩展功能彻底剥离铁律**：
   - 核心仓库**只包含通用 AI 数字个体的核心能力（`crates/*`）与唯一内置的 WebUI 控制台面板（`apps/webui`）**。
   - **严禁在核心仓库内置任何 2D/3D 形象渲染引擎（如 Live2D、VRM、Three.js、MMD、Spine、口型/微动驱动）、独立桌面/移动壳、或任何具体第三方平台/直播/游戏集成（如 Discord、Telegram、Satori、Bilibili、Minecraft）**。
   - 除 `apps/webui` 面板外，其他一切外部渠道、形象表现与外部工具均必须通过 `crates/alia-extension-host`（专题 `17`）、`crates/alia-plugin-protocol`（专题 `18`）与 `crates/alia-channel-gateway`（专题 `19`）以插件方式扩展。
2. **语言与生态边界铁律（Rust 核心后端 + Vue 3/TS 唯一内置 WebUI 面板）**：
   - **后端与底层核心必须使用 Rust (`crates/*`)**：包含 `alia-server`、`alia-protocol`、`alia-core-character`、`alia-core-agent`、`alia-provider-inference`、`alia-storage`、`alia-audio-pipeline`、`alia-vision-pipeline`、`alia-plugin-protocol`、`alia-extension-host`、`alia-channel-gateway`、`alia-auth-service`、`alia-stream-kit`、`alia-trace-kit`。
   - **前端仅保留唯一内置的 `apps/webui` 控制台面板与 6 个非渲染基础包 (`packages/stage-shared`, `packages/stage-ui`, `packages/audio-web`, `packages/ui`, `packages/timeline-canvas`, `packages/ui-timeline`, `packages/ui-trace`)**：在 WebUI 面板内完成全部配置、测试沙盒直接对话验证、渠道与插件扩展管理及 Trace 诊断。
3. **契约即 Rust 符号（Contract-as-Rust-Symbol + Specta），禁止手写孤立 JSON Schemas**：
   - 所有前后端共享数据结构、协议帧与领域类型必须在 Rust Crates 中定义为 Rust `struct` / `enum` 并标注 `#[derive(Serialize, Deserialize, specta::Type)]`，编译期自动导出到 `packages/stage-shared`。
