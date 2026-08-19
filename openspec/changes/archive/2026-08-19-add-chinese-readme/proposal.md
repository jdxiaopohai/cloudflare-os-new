# Proposal: add-chinese-readme

## Why

项目根目录的 `README.md` 目前仅有英文版本。对于中文使用者（包括本项目维护者本人及潜在的中文读者）来说，阅读英文文档存在门槛，不利于快速理解 Cloudflare OS 的定位、功能与使用方法。提供一份中文翻译版本可以降低阅读成本，方便中文用户快速上手。

## What Changes

- 新增 `README.zh-CN.md`：对根目录 `README.md` 的完整简体中文翻译，与英文原版结构一一对应（标题层级、列表、表格、链接、代码块均保持一致）。
- 在英文 `README.md` 顶部添加一行指向中文版本的链接（如 `简体中文版本：README.zh-CN.md`），方便读者发现译文。
- 翻译约定：
  - 专有名词保留英文原文并酌情附中文注释，如 Gadget、Blueprint、Gatekeeper、Durable Object、Dynamic Worker、Cap'n Web、Workers 等产品/技术名称不译。
  - 所有相对链接（如 `packages/gatekeeper-github/README.md`、`docs/images/q3-planning-workspace.png`）保持原路径不变，确保在仓库内仍可正常跳转。
  - 代码块、命令（如 `pnpm run-local`）和 URL 原样保留，不做翻译。
- **非目标**：不修改英文 `README.md` 的正文内容；不翻译 `packages/` 下各子包的 README；不建立持续的翻译同步机制（译文以本次英文版快照为准）。

## Capabilities

### New Capabilities

（无 —— 本变更为纯文档变更，不引入新的系统行为能力。）

### Modified Capabilities

（无 —— 本变更不修改任何现有行为规范。）

本变更属于纯文档类变更，已在 `.openspec.yaml` 中声明 `skip_specs: true`。

## Impact

- **受影响文件**：新增 `README.zh-CN.md`；`README.md` 仅新增一行译文链接。
- **代码/API/依赖**：无任何影响，不涉及任何包、构建脚本或配置。
- **风险**：极低。唯一需注意点是后续英文 README 更新时译文可能滞后，已在“非目标”中明确本次不建立同步机制。
