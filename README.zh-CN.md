# Cloudflare OS：AI 生产力环境

Cloudflare OS 是一个面向 AI 生产力的“操作系统”，最初为 Cloudflare 内部使用而开发。Cloudflare 的大量员工——从工程到销售以及各个岗位——每天都在使用 Cloudflare OS 来辅助完成工作。

![Cloudflare OS 中一个 Q3 规划工作区，包含 AI 生成的幻灯片](docs/images/q3-planning-workspace.png)

这并不是一个传统意义上的计算机操作系统。我们在两种意义上使用“操作系统”这个词：

* 一个让*公司*能够安全地借助 AI 提升生产力的操作系统，让安全团队可以高枕无忧。
* 一个面向 AI 工作负载的操作系统，类似于传统操作系统管理计算工作负载的方式。

Cloudflare OS 特别提供三样东西：

1. 一个 agent 聊天界面，你可以让 agent 完成任务，它已预加载了关于你公司运作方式的知识。
2. 沙箱化的应用开发环境，你可以让 agent 构建“gadget”（小型个人应用），并安全地与他人分享你的成果。
3. 一个名为 Gatekeeper 的安全框架，为 agent 和应用施加防护栏，让非技术用户也可以安全地“放手去玩”，而不会出任何乱子。

我们将 Cloudflare OS 开源，以便其他人可以复制它并为自己的公司定制。我们的理念不是让你的公司来使用 Cloudflare OS，而是让你把它打造成“*你自己的公司* OS”。

## 快速开始

要在本地快速运行 Cloudflare OS，请先[安装 pnpm](https://pnpm.io/)，然后执行：

    pnpm run-local

然后访问：http://localhost:8787

这会在本地通过 wrangler 和 workerd 运行整个技术栈。它不适用于生产环境，但是一个快速了解产品功能的方式。

或者，你也可以[部署到你自己的 Cloudflare 账户](https://os.cloudflare.app/deploy)。

（更多选项见本 README 末尾。）

### 可以尝试什么

试试这样的提示词：

* “为我即将与客户的会议制作幻灯片。”（这将使用内置的幻灯片 blueprint。）
* “做一个协作白板应用。”（这将从零开始创建一个新应用。）
* “做一个井字棋游戏。”然后说：“我下 X，你下 O。我已经走了第一步，轮到你了。”
* “为这个 GitHub 仓库做一个 issue 仪表盘。”（附加一个仓库；需要配置好 GitHub 集成。）
* “修正这个 Google Doc 里的错别字。”（附加一个文档；需要配置好 Google 集成。）

### 警告：早期体验阶段

Cloudflare OS 正处于高强度开发中。本仓库实际上是第 2 版——一次完全重写，把我们从第 1 版中学到的经验放到了全新的基础上。

截至 2026 年 8 月发布的版本，Cloudflare OS v2 功能已经相当强大，但仍有许多粗糙之处。我们知道，并且正在改进。目前请将本版本视为“早期体验”版本。

## 概述：Cloudflare OS 到底是什么？

### Gadget：一种全新的软件思维方式

Cloudflare OS 不仅仅是又一个带连接器的聊天框。整个系统围绕一种新的软件方式构建：每个用户都运行自己所使用的生产力应用的专属副本。

当你在 Cloudflare OS 中创建一份幻灯片时，你并不是在调用某个运行在云端的 SaaS 软件。系统会*专为你*创建一个幻灯片软件的*私有实例*。我们称之为“gadget”。这个实例运行在一个与其他所有人的幻灯片相隔离的独立沙箱中。

这带来两个深远的影响：
1. 幻灯片应用不可能存在把你的幻灯片泄露给攻击者的安全漏洞。Cloudflare OS 沙箱控制着对你的应用私有实例的一切访问。
2. 如果你愿意，你可以自由地修改代码。如果幻灯片应用缺少你需要的功能，你只要让你的 agent 加上即可。而且由于第 1 点，这样做完全安全。

这与过去 25 年的云架构和“软件即服务”（SaaS）模式大相径庭，但我们认为 AI 已经改变了这个等式。当任何用户都能通过提示 agent 来添加自己需要的功能时，软件的集中式模式就不再合理了。

### Gatekeeper：基于能力（capability）的安全层

Gatekeeper 就像是超级增强版的 MCP 服务器。

当你把某个外部资源介绍给 agent 或 Gadget 时，就会创建一个 Gatekeeper 来管理这次访问。Gatekeeper 是针对每个外部服务的专用软件，它调解 Gadget 与该服务之间的连接。它会：
* 为该服务提供一套整洁的 Cap'n Web API（包装该服务原生提供的任何 API）。
* 处理授权（例如通过 OAuth）。
* 强制将访问范围收窄到用户本意指定的那个具体资源。
* 记录 Gadget（或 agent）执行的每一个操作，供你审查。
* 对于任何有副作用的操作，让人类用户有机会批准或拒绝该操作（“人在回路”）。

关于最后一点，Gatekeeper 实现了对现有技术的一项重要突破。传统上，“人在回路”的设置要求人类*同步*批准操作。当 agent 想做某件事时，它必须*停下来*等待批准才能继续。这很恼人：你给 agent 布置了一个任务，然后走开去喝杯咖啡，回来却发现 agent 卡在第一步的审批上，毫无进展。结果，人们往往会妥协，把 agent 设置为“自动批准”，或者使用 `--dangerously-skip-permissions`，这显然是不安全的。

Gatekeeper 提供了更好的方式：当 agent（或 Gadget）执行一个需要批准的操作时，Gatekeeper 会在本地*模拟*其结果，让 agent 可以继续推进并排入更多操作。Gatekeeper 会告诉 agent 操作已完成；如果 agent 试图读取结果，Gatekeeper 会给它模拟的结果。当 agent 完成后，用户可以批量或逐个批准或拒绝这些操作——无论哪种方式，都可以在之后方便的时候再做。

从部署结构上看，每个 Gatekeeper 都实现为一个独立的 Worker。未来，我们设想 Gatekeeper 服务可以独立于 OS 实例部署和维护，但细节尚待确定。目前，我们在本仓库中提供了一些有趣的 Gatekeeper，你可以将它们与你自己的 OS 实例一起部署。

### 把它想象成一套办公套件

Cloudflare OS 的基本用户体验有点类似在线办公套件，比如 Google Docs 或 MS Office。但想象一下：不再是固定的一组文件类型（文档、电子表格、幻灯片），而是每个文件——或者说每个“Gadget”——都可能是一个专属的定制应用，由 AI 编写，恰好满足你的需求。

就像办公文档一样，每个 gadget 默认是私有的，但可以——安全地——共享，以便与你的团队或朋友协作。

就像办公文档一样，你可以拥有成千上万个 gadget，可以凭一时兴起随手创建。

就像办公文档一样，你可以从“模板”开始——这里称为“Blueprint”。但办公模板只是一些内容，而一个 Blueprint 定义的是一整个应用。

也像办公文档一样，你可以从自己的文档（Gadget）创建新模板（Blueprint）并与他人分享。但当你这样做时，你分享的是一整个应用的代码。

### 它某种程度上确实是一个操作系统

“OS”这个说法并不*完全*是营销话术。在技术层面上，Cloudflare OS 确实与一个操作系统有着真实的类比关系：

| 传统 OS        | Cloudflare OS              |
|----------------|----------------------------|
| kernel         | packages/workshop-backend  |
| device drivers | packages/gatekeeper-*      |
| shell          | packages/workshop-frontend |
| processes      | gadgets                    |
| executables    | blueprints                 |
| users          | users                      |
| ACLs           | shared permissions         |
| ???            | agents                     |

我们的“内核”在 workshop-backend 包中。这个后端确实做了很多与真实 OS 内核类似的事情：它把用户与程序和设备（我们称之为 Gadget 和 Gatekeeper）连接起来，同时通过沙箱化应用和强制执行访问控制来实现安全。

在这个类比中，Gatekeeper——把用户和 agent 连接到外部服务——就像驱动程序——把用户和程序连接到外部设备。

有一样东西是传统 OS 今天并没有真正管理、而 Cloudflare OS 管理了的：AI agent。仔细想想，这其实是传统 OS 缺失的一项功能。我们认为，AI agent 不能简单地被当作用户对待。它们必须对某个人类用户负责，同时又拥有自己受限的权限。agent 通过编写代码片段并即时执行来完成工作。这一切的理想安全模型是基于能力的安全（capability-based security），而不是访问控制列表（ACL）。明白我的意思了吗？也许传统 OS 也应该给 AI agent 特殊对待。

### 构建于 Workers 之上，出自 Workers 团队之手

Cloudflare OS 构建于 [Cloudflare Workers](https://workers.cloudflare.com) 之上，并大量使用 [Durable Objects](https://developers.cloudflare.com/durable-objects/)、[Dynamic Workers](https://blog.cloudflare.com/dynamic-workers/)，尤其是 [Facets](https://blog.cloudflare.com/durable-object-facets-dynamic-workers/)。每个工作区都是它自己的 Durable Object，每个 Gadget 运行在一个 Dynamic Worker Facet 中，Gatekeeper 也会向每个工作区安装 facet 来管理对远程服务的访问。

事实上，Cloudflare OS 正是由构建 Workers 的那批人打造的。它使用了 Workers 运行时最前沿的特性——事实上，Dynamic Workers、Facets 以及其他一些特性就是为了支持 Cloudflare OS 而专门加入运行时的，未来还会有更多。研究 Cloudflare OS 的源代码，是理解 Workers 运行时团队认为 Workers 应该如何使用的一个绝佳途径。

构建于 Workers 之上并不意味着 Cloudflare OS 只能运行在 Cloudflare 上。事实上，[`workerd`——Cloudflare Workers 运行时——本身就是开源的](https://github.com/cloudflare/workerd)，Cloudflare OS 可以完全基于它运行在你自己的服务器上。

## 功能特性

### 通用的多用途 agent

Cloudflare OS 的编码 agent 实际上是一个完全多用途的 agent，可以执行任意任务；和其他流行的编码 agent 一样，你不一定非要用它来写代码。你可以用它构建 Gadget，也可以跳过 Gadget，直接让 agent 执行任务。Cloudflare OS 的 agent 是一个 [Code Mode](https://blog.cloudflare.com/code-mode/) agent——它通过编写代码片段并立即执行来完成任务。它可以通过 Gatekeeper 连接到外部资源（类似 MCP——见下文）。

### 用 AI 构建应用

如果你愿意，当然可以手工编写 Gadget 的代码，但我们的预期是由 AI 为你写代码。Cloudflare OS 内置了一个编码 agent，它会构建你要求的任何东西，为你测试，并调试错误。

你可以自由选择 LLM。Cloudflare OS 支持许多主流的 AI 模型提供商和自托管模型，并且不断有新的提供商加入。

由于平台高度集成且精简，即使使用相同的底层 AI 模型，Cloudflare OS 的编码 agent 往往比通用编码 agent 表现更好、更快，消耗的 token 也更少。

### 与 AI 协作

每个用 Cloudflare OS 构建的应用都自动拥有一个对 agent 友好的 API。这意味着，在你让 AI 构建完应用之后，你还可以让 AI 在应用*内部*与你协作。无需构建 MCP 服务器，也无需集成定制的 agent 循环——它默认就在那里。

这之所以可行，是因为 Gadget 的客户端和服务端部分必须通过 [Cap'n Web RPC](https://github.com/cloudflare/capnweb) 通信。这是双赢的：
1. Cap'n Web 的样板代码极少，这让 agent 很容易使用。你基本上只需在服务端定义一个方法，然后从客户端调用它，就像本地调用一样。
2. 与此同时，这意味着服务端必然暴露出一个易于理解的 API，agent 可以直接调用。AI agent 运行框架使用 [Code Mode](https://blog.cloudflare.com/code-mode/) 进行工具调用，因此把 Gadget 的 API 直接暴露给 agent 调用是轻而易举的。

### 实时多人协作

你可以像共享典型在线办公套件中的文档一样共享你的 Gadget。你可以授予特定用户访问权限，或者创建一个分享链接，让任何打开它的人都能访问。而且就像那些在线办公套件一样，你能实时看到协作者的操作。

这之所以可行，是因为每个 Gadget 背后都有一个 [Durable Object](https://developers.cloudflare.com/durable-objects/)——Cloudflare 的有状态无服务器原语，它让实时多人协作变得简单。简单到编码 agent 默认就会实现它，根本不需要你开口要求。

### Blueprint：分享你的代码

如果你创建了一个对别人可能有用的 Gadget，但又不想共享这个 Gadget 本身，你可以改为分享一个 Blueprint，让其他人创建他们自己的 Gadget 副本。Blueprint 本质上就是代码的一份副本。

这听起来简单，但 Blueprint 是对云软件传统的一次重大变革。传统上，如果你创建了一个想分享给其他用户的 Web 应用，你会把应用托管在自己的服务器上，用户连接到你的服务器。Blueprint 则更像移动应用和传统 PC 应用：每个用户运行自己的软件副本。

在 AI 时代，这一变革至关重要。一方面，AI 让个人开发者能构建比以往更多的东西，但个人开发者维护一个在线服务仍然很困难；Blueprint 消除了这种需求。另一方面——甚至更重要的是——让每个用户运行自己的软件副本，使用户能够借助 AI *修改*软件以满足自己的需求。无需提交功能请求，无需恳求开发者优先排期。最终用户可以自己解决自己的问题。

### 默认沙箱化，默认安全

每个 Gadget 都运行在一个安全的沙箱中，未经你明确同意，它完全无法与互联网通信。具体来说：
* 服务端运行在一个[已禁用互联网访问的 Dynamic Worker](https://blog.cloudflare.com/dynamic-workers/) 中。它只能通过 [Workers Bindings](https://blog.cloudflare.com/workers-environment-live-object-bindings/) 与你明确指定的特定外部资源通信。
* 客户端代码运行在一个沙箱化的 iframe 中。这个 iframe 只能通过由父框架经 `postMessage()` 提供的 Cap'n Web RPC 会话与其服务端通信。除此之外，iframe 被阻止访问互联网（在浏览器允许的最大范围内，通过 `Content-Security-Policy` 和 iframe 沙箱设置实现）。

### 基于能力的访问控制

每个 agent、每个 Gadget 默认都无权访问任何东西。即使你已经为 Gadget Workshop 配置了对外部账户的访问权限，agent 和 Gadget 也*不会*自动获得使用它们的能力。

相反，你必须把每个 agent（或 Gadget）*介绍*给你想让它访问的特定资源。例如，你可以通过粘贴一个 GitHub 仓库的链接来介绍它，或者点击“添加资源”并通过 UI 选择。agent 也可以主动请求介绍它认为自己需要的资源，由你来提供或拒绝。

这与大多数 agent 运行框架不同：在那些框架中，MCP 服务器是预先配置好的，你所有服务的广泛访问权限在每次对话中都天然地提供给 agent。基于能力的介绍机制让每个 agent 都被限制在它完成手头工作实际需要的访问范围内。

## 开始使用

### 部署到你的 Cloudflare 账户

我们构建了一个在线流程，帮助你部署到自己的 Cloudflare 账户：

https://os.cloudflare.app/deploy

或者，如果你想要更复杂的部署——带上你自己的 gatekeeper 以及可能的代码改动——请查看我们的部署起始仓库：

https://github.com/cloudflare/cloudflare-os-starter

### 本地运行

要在本地快速运行 Cloudflare OS，请先[安装 pnpm](https://pnpm.io/)，然后执行：

    pnpm run-local

然后访问：http://localhost:8787

这会使用 `wrangler`（Workers 开发者工具 CLI）运行 Cloudflare OS。这不是在生产服务器上运行该 OS 的正确方式，但对于在本地机器上试用来说完全没问题。

你的数据将存储在名为 `.wrangler` 的子目录中。

### 使用 `workerd` 部署到你自己的服务器

**即将推出**

Cloudflare OS 可以完全运行在 `workerd`（Cloudflare 开源的 Workers 运行时）之上。事实上，上面的“本地运行”说明在底层使用的就是 `workerd`。我们仍在完善文档和工具，以帮助你顺利地在自己服务器上基于 `workerd` 部署该 OS。如果你喜欢冒险，可以[阅读 workerd 配置的底层文档](https://github.com/cloudflare/workerd/blob/main/src/workerd/server/workerd.capnp)（或者让你的 agent 去读）并亲自一试。

#### 配置外部服务

许多 Gatekeeper 需要配置才能连接到第三方服务，包括为每个服务获取 OAuth 客户端凭据。不幸的是，许多服务提供商故意不让这件事变得容易，因为 OAuth 的目标受众是开发者。

每个 gatekeeper 包中都包含了设置说明：

* [GitHub API](packages/gatekeeper-github/README.md)
* [Google API](packages/gatekeeper-google/README.md)
* [Cloudflare API](packages/gatekeeper-cloudflare/README.md)
* [Supabase API](packages/gatekeeper-supabase/README.md)
* [Notion API](packages/gatekeeper-notion/README.md)
* [Confluence API](packages/gatekeeper-confluence/README.md)
* [Email Workers](packages/gatekeeper-email/README.md)
* [Home Assistant](packages/gatekeeper-homeassistant/README.md)
* [Slack API](packages/gatekeeper-slack/README.md)
* [Spotify](packages/gatekeeper-spotify/README.md)
* [ZoomInfo API](packages/gatekeeper-zoominfo/README.md)

## 开发

开发时，你需要在两个终端中分别运行前端和后端两条命令：

    pnpm dev-server
    pnpm dev-client

然后访问：http://localhost:3000

### 贡献

目前，我们不寻求外部贡献。

AI 让写代码变得容易。如今的难点不在于写代码，而在于审查代码、确保质量保持高水准、保持产品的一致性。从这个角度看，很遗憾，外部代码贡献是在“捐赠”工作中简单的部分，同时制造了更多困难的工作。

话虽如此，我们很乐意接受那些能轻松验证的、修复问题的小型 PR。但是，请不要提交低价值的 PR（例如错别字修复）或超过十几行的 PR。此类 PR 将附带本指南的引用而被关闭。

如果你有一个希望我们考虑的重大想法，欢迎[发起讨论](https://github.com/cloudflare/cloudflare-os/discussions)。

随着项目的成熟，这一政策未来可能会改变。在此之前，感谢你的理解。

## 致谢

Cloudflare OS 的开源依赖多到无法在此一一列举。但我们想特别感谢几个承担了重任的项目：

* [Pi](https://pi.dev/)（具体来说是 `pi-agent-core`），它让我们用一个 API 就能轻松支持所有 LLM 提供商。
* [Monaco](https://microsoft.github.io/monaco-editor/)，它让嵌入一个漂亮的文本编辑器变得太容易了——对我们这些仍然会看代码的人来说。
* [Yjs](https://yjs.dev/)，我们大量使用它在客户端和 agent 之间同步代码变更并重放历史。
* [Vite](https://vite.dev/)，它让开发循环如此愉悦。
