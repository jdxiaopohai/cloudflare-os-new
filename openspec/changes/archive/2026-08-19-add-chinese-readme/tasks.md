# Tasks: add-chinese-readme

## 1. 创建中文译文

- [x] 1.1 通读英文 `README.md` 全文，确认章节结构（标题层级、列表、表格、代码块、图片与链接），作为翻译对照基准
- [x] 1.2 创建 `README.zh-CN.md`，逐节翻译 `README.md` 全部内容为简体中文，保持与原文章节结构一一对应
- [x] 1.3 按翻译约定校对译文：Gadget、Blueprint、Gatekeeper、Durable Object、Dynamic Worker、Cap'n Web、Workers 等专有名词保留英文原文；命令（如 `pnpm run-local`）、代码块与 URL 原样保留不译

## 2. 校验链接与格式

- [x] 2.1 核对译文中的相对链接（`docs/images/q3-planning-workspace.png`、`packages/gatekeeper-*/README.md` 等）路径与原文一致，在仓库内可正常跳转
- [x] 2.2 核对译文中的外部链接（如 https://os.cloudflare.app/deploy、各博客与 GitHub 链接）与原文一致
- [x] 2.3 检查 Markdown 渲染：表格（OS 类比表）、图片、代码块、引用格式与英文版一致，无破损语法

## 3. 关联英文原版

- [x] 3.1 在英文 `README.md` 顶部（标题下方）添加一行指向译文的链接，如 `> 简体中文版本：[README.zh-CN.md](README.zh-CN.md)`，不改动英文正文其他内容

## 4. 最终检查

- [x] 4.1 对照英文原文逐节复查译文，确认无遗漏段落、无翻译错误、术语全文统一
