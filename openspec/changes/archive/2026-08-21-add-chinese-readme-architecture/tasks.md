# Tasks: add-chinese-readme-architecture

## 1. 收集架构事实

- [x] 1.1 阅读 `AGENTS.md`、`pnpm-workspace.yaml`、根 `wrangler.jsonc`，整理 monorepo 结构、共享工具链版本（catalog）与顶层路由模型
- [x] 1.2 阅读 `packages/router`、`packages/workshop-frontend`、`packages/workshop-backend`、`packages/workshop-shared` 的 `package.json` 与 `wrangler.jsonc`，确认各包职责、依赖与入口文件
- [x] 1.3 确认代码入口链路：`scripts/run-local.ts` / `scripts/run-dev-server.ts` 启动方式、`packages/router/src/index.ts` 路由逻辑、`packages/workshop-frontend/index.html` → `src/main.tsx` 前端挂载、`packages/workshop-backend/src/server.ts` 经 capnweb-validate 构建为 worker 入口
- [x] 1.4 浏览 `packages/` 下 gatekeeper 相关包（`gatekeeper-*`、`mcp-shared`、`configurator-ui`）及 `scripts/release/`，概括 gatekeeper 模型与发布管线要点

## 2. 撰写架构章节

- [x] 2.1 在 `README.zh-CN.md` 的「功能特性」之后、「开始使用」之前插入新章节「技术架构与技术栈」，与全文标题层级风格一致（`##` 主标题 + `###` 小节）
- [x] 2.2 撰写「整体架构」小节：monorepo 结构、router/workshop-backend/workshop-frontend/workshop-shared/gatekeeper-* 各包职责与分层，附代码块形式的目录结构示意
- [x] 2.3 撰写「运行时架构」小节：Cloudflare Workers / workerd、Durable Objects（每工作区一个 DO）、Dynamic Workers + Facets（Gadget 沙箱）、Service Bindings 服务间通信
- [x] 2.4 撰写「通信协议」小节：Cap'n Web RPC（浏览器 WebSocket + Worker 间绑定）、agent 的 Code Mode 调用方式
- [x] 2.5 撰写「技术栈」小节：前端 React 19 / Vite 7 / Kumo UI / Phosphor / TanStack Router / Tailwind CSS 4 / Yjs / Monaco；后端与工具链 TypeScript 7（tsgo）/ pnpm 11 workspace+catalog / wrangler 4 / workerd / vitest 双测试工程 / vite-plus（vp）/ capnweb-validate
- [x] 2.6 撰写「代码入口」小节：本地启动命令链路（`pnpm run-local`、`pnpm dev-server` + `pnpm dev-client`）、路由入口 `packages/router/src/index.ts`、前端入口 `index.html` → `src/main.tsx`、后端入口 `src/server.ts` → `.wrangler/validate/src/server.ts`，说明各自端口与角色
- [x] 2.7 简要撰写「发布管线」小节：`scripts/release/` 字节级一致构建、R2 内容寻址上传、manifest 晋升

## 3. 校对与事实核查

- [x] 3.1 对照第 1 组收集的事实逐条核查章节内容，确认无臆造、无与仓库文件矛盾的陈述（版本号、包名、入口路径、端口均以仓库为准）
- [x] 3.2 沿用中文 README 既有翻译约定：Gadget、Blueprint、Gatekeeper、Durable Object、Dynamic Worker、Cap'n Web、Workers 等专有名词保留英文；代码块、命令、路径原样保留
- [x] 3.3 检查 Markdown 渲染：标题层级、表格、代码块无破损语法；新增章节与上下文衔接自然，不改动中文 README 其他既有内容
