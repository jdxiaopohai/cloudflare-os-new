# Proposal: add-chinese-readme-architecture

## Why

上一个变更（已归档：`2026-08-19-add-chinese-readme`）为项目提供了 `README.zh-CN.md` 中文译文，但该译文仅忠实翻译了英文原版，对项目的**技术架构、技术栈与代码入口**着墨有限。对于想深入了解或二次开发该项目的中文读者，目前需要自行翻阅 `AGENTS.md`、各包的 `package.json`、`wrangler.jsonc` 等文件才能拼凑出整体架构与启动方式，门槛较高。在中文 README 中补充一节详细的技术架构、技术栈与前后端代码入口说明，可以让中文读者快速建立对系统的整体认知并知道从哪里开始读代码。

## What Changes

- 在 `README.zh-CN.md` 中新增一个章节（建议标题「技术架构与技术栈」，插入在「功能特性」之后、「开始使用」之前），用中文详细描述：
  - **整体架构**：monorepo 结构（pnpm workspace，`packages/*`）、各核心包的职责与分层——
    - `packages/router`：部署实例的公开入口，按路径前缀路由（`/api/*` → workshop-backend，`/gatekeeper/<name>/*` → 各 gatekeeper），本地开发时兼作 dev router 代理前端请求；
    - `packages/workshop-backend`：「内核」，运行在 Cloudflare Workers 上，负责沙箱、访问控制、用户/工作区 Durable Object；
    - `packages/workshop-frontend`：纯客户端 SPA（React 19 + Vite + Kumo UI + Phosphor 图标 + TanStack Router），通过 WebSocket 上的 RPC 与后端通信；
    - `packages/workshop-shared`：前后端共享的 Cap'n Web RPC 接口定义；
    - `packages/gatekeeper-*`：外部服务集成的 gatekeeper workers（OAuth、沙箱化 API 访问）；
    - `packages/mcp-shared`、`packages/configurator-ui` 等支撑库；
  - **运行时架构**：Cloudflare Workers / workerd、Durable Objects（每个工作区一个 DO）、Dynamic Workers + Facets（每个 Gadget 的运行沙箱）、Workers Bindings / Service Bindings 的服务间通信模型；
  - **通信协议**：Cap'n Web RPC（浏览器经 WebSocket、Worker 间经 service binding），agent 通过 Code Mode 调用；
  - **技术栈**：前端 React 19 / Vite 7 / Kumo UI / Phosphor / TanStack Router / Tailwind CSS 4 / Yjs / Monaco；后端与工具链 TypeScript 7（tsgo）/ pnpm 11（workspace + catalog）/ wrangler 4 / workerd / vitest（Node 与 workerd 双测试工程）/ vite-plus（`vp` 任务编排与缓存）/ capnweb-validate（RPC 运行时校验）；
  - **代码入口**（前后端如何启动、入口文件在哪）：
    - 本地整体启动：`pnpm run-local` → `scripts/run-local.ts`（构建前端产物后，以根 `wrangler.jsonc` + 各包 `wrangler.dev.jsonc` 启动 wrangler，监听 http://localhost:8787）；开发模式 `pnpm dev-server`（`scripts/run-dev-server.ts`）+ `pnpm dev-client`（`packages/workshop-frontend` 的 Vite dev server，http://localhost:3000）；
    - 路由入口：`packages/router/src/index.ts`（根 `wrangler.jsonc` 的 `main`，即 dev-router，按路径前缀转发）；
    - 前端入口：`packages/workshop-frontend/index.html` → `src/main.tsx`（React SPA 挂载点）；
    - 后端入口：`packages/workshop-backend/src/server.ts`（经自定义构建 `pnpm run build:worker` 由 capnweb-validate 输出到 `.wrangler/validate/src/server.ts`，作为后端 `wrangler.jsonc` 的 `main`）；
  - **发布管线**（简要）：`scripts/release/` 的字节级一致构建、R2 内容寻址上传、manifest 晋升机制。
- 内容来源以仓库事实为准：`AGENTS.md`、`pnpm-workspace.yaml`、根 `wrangler.jsonc`、各包 `package.json`/`wrangler.jsonc`、入口源码文件、`scripts/release/`。
- **非目标**：
  - 不修改英文 `README.md`（本节为中文版专属补充）；
  - 不改变任何代码、配置或系统行为；
  - 不逐一枚举全部 20+ 个 gatekeeper，只概括其角色并举几个例子；
  - 不绘制架构图图片，仅用文字、表格和代码块形式的结构示意。

## Capabilities

### New Capabilities

（无 —— 本变更为纯文档变更，不引入新的系统行为能力。）

### Modified Capabilities

（无 —— 本变更不修改任何现有行为规范。）

本变更属于纯文档类变更，已在 `.openspec.yaml` 中声明 `skip_specs: true`。

## Impact

- **受影响文件**：仅 `README.zh-CN.md` 新增一个章节。
- **代码/API/依赖**：无任何影响。
- **风险**：低。主要风险是架构/入口描述与代码事实不符——缓解方式是所有陈述均以仓库内文件（`AGENTS.md`、`pnpm-workspace.yaml`、各 `package.json`/`wrangler.jsonc`、入口源码）为依据，并在任务中安排事实核查。
