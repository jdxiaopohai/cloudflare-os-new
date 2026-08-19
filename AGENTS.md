# Cloudflare OS 项目说明

这是一个面向 AI 生产力的工作台与安全沙箱平台，目标是让 AI Agent 能安全地构建“gadget”（小型应用）、连接外部资源，并在强约束下执行任务。

本项目的核心文档：
- [README.md](README.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [docs](docs)

## 1. 项目定位

- 集成了聊天式 AI Agent、可运行的小型应用、Gatekeeper 安全边界。
- 使用 Cloudflare Workers / Durable Objects / Dynamic Workers 作为底层运行时。
- 前端是纯客户端 SPA，后端与前端通过 Cap'n Web RPC 通信。

## 2. 关键目录

- [packages/workshop-backend](packages/workshop-backend)：内核层，负责架构、安全、会话、工作区编排，代码审查要求最高。
- [packages/workshop-frontend](packages/workshop-frontend)：工作台前端，负责 UI 和交互。
- [packages/workshop-shared](packages/workshop-shared)：前后端共享 API 定义，尤其是 RPC 接口。
- [packages/gatekeeper-*](packages)：外部服务连接器，负责 OAuth、授权、观测和审批控制。
- [packages/router](packages/router)：部署后的入口路由 Worker。
- [scripts](scripts)：构建、开发、测试和发布脚本。

## 3. 代码约定

### 3.1 核心安全与架构约束

- `workshop-backend` 是内核，改动要保持小而精，优先复用现有机制，而不是新建并行设计。
- `workshop-shared` 里的对外导出类型、常量和函数需要补充注释。
- 不要手写与 RPC 接口重复的接口类型并通过 `as unknown as` 逃逸；应尽量复用真实类型。
- Gatekeeper 只应在用户或管理员配置后获得“ambient”能力；不能自行伪造环境信息。

### 3.2 RPC 约定

- 本项目大量使用 Cap'n Web RPC，前后端 API 接口定义在 [packages/workshop-shared/src/api.ts](packages/workshop-shared/src/api.ts)。
- 若 RPC 返回 stub，优先使用 promise pipelining，不必强制 `await`。
- React 中如果 `useState()` 要保存 RPC stub，应该包在对象中存储，避免被当成函数执行。
- RPC stub 用完后应及时 `dispose`，避免服务端资源泄露。

### 3.3 日志与错误处理

- 后端日志使用 `@gadgets/backend-utils/logger`，不要直接用浏览器 `console`。
- 日志事件应具备稳定的 `component` 与必要字段，且不要记录秘密、token、header、request/response body。
- 错误上报需走受控的 `reportIssue(...)` 路径，并保留有限的上下文。

## 4. 开发与验证

优先使用 pnpm，而不是 npm。

常用命令：
- `pnpm build`：全量构建 / 类型检查
- `pnpm test`：运行测试
- `pnpm lint`：运行 lint 与类型脚本检查
- `pnpm run-local`：本地启动完整开发环境
- `pnpm dev-server`：启动开发服务器

对于单个包，优先使用：
- `vp run -F <package> build`
- `vp run -F <package> test`

## 5. 经验与坑点

- `workshop-backend` 和 `workshop-shared` 一般需要更严格的审查，尽量保持 diff 小而清晰。
- 修改 Gatekeeper/资源绑定相关逻辑时，优先检查资源授权入口和 admin 配置。
- 运行时环境变量对缓存有影响；涉及 env 的任务需要显式声明，否则会出现缓存数据错误。
- 本工程依赖 Cloudflare Workers 运行时特性，某些测试需要在 workerd 环境下执行。

## 6. 推荐的修改顺序

1. 先定位相关包和文件。
2. 看清 API 边界和安全约束，再动手改代码。
3. 优先跑最小相关验证命令。
4. 在修复前后确认行为与架构一致，不要引入并行抽象。

## 7. 参考资料

- [README.md](README.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [docs](docs)
- [packages/workshop-shared/src/api.ts](packages/workshop-shared/src/api.ts)
- [packages/workshop-shared/node_modules/capnweb/README.md](packages/workshop-shared/node_modules/capnweb/README.md)

本文件用于指导 AI 编程助手在该仓库中保持一致的架构理解、开发习惯和安全边界。
