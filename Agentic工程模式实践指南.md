# Agentic 工程模式实践指南

> 基于 Simon Willison 的 [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/) 整理
> 
> 生成日期：2026-07-10

---

## 目录

1. [什么是 Agentic 工程？](#1-什么是-agentic-工程)
2. [核心原则](#2-核心原则)
3. [与 Coding Agent 协作](#3-与-coding-agent-协作)
4. [测试与质量保证](#4-测试与质量保证)
5. [理解代码](#5-理解代码)
6. [实用 Prompt 模板](#6-实用-prompt-模板)
7. [速查清单](#7-速查清单)

---

## 1. 什么是 Agentic 工程？

**Agentic 工程** = 使用 Coding Agent 辅助开发软件的实践。

### 关键概念

| 概念 | 定义 |
|------|------|
| **Agent** | 在循环中调用工具以实现目标的软件 |
| **Coding Agent** | 能**编写并执行**代码的 Agent（如 Claude Code、OpenAI Codex、Gemini CLI） |
| **核心能力** | 代码执行 — 没有执行能力，LLM 输出的价值有限 |

### 与 Vibe Coding 的区别

- **Vibe Coding**（Andrej Karpathy 提出）：提示 LLM 写代码，同时"忘记代码的存在" → **原型级、未审查**
- **Agentic 工程**：系统性地使用 Agent，产出**生产级**代码 → 有审查、有测试、有质量保证

### 人类的角色

> 写代码从来不是软件工程师的唯一活动。真正的技艺在于**弄清楚该写什么代码**。

- 导航数十种可能的解决方案，找到最适合当前场景的
- 为 Agent 提供工具、描述问题、验证结果
- 持续迭代指令和工具体系

---

## 2. 核心原则

### 2.1 写代码现在很便宜了

**范式转变：** 代码曾经昂贵（几百行干净代码 = 一天工作量），现在 Agent 让"敲代码"几乎免费。

#### 宏观层面的改变
- 过去：大量时间用于设计、估算、规划 → 确保昂贵的编码时间高效使用
- 现在：功能不再需要"赚回开发成本"才值得做

#### 微观层面的改变
- 过去："重构那个函数多花一小时值不值？" → 权衡
- 现在：直接让 Agent 去做，10 分钟后检查结果

#### ⚠️ 好代码仍然有成本

"好代码"的定义：
- ✅ 能工作，没有 bug
- ✅ 我们**知道**它能工作（有验证手段）
- ✅ 解决了正确的问题
- ✅ 优雅处理错误，不只考虑 happy path
- ✅ 简单、最小化、人类和机器都能理解
- ✅ 有测试保护（回归测试）
- ✅ 有适当文档
- ✅ 设计支持未来变更（YAGNI + 可维护性）
- ✅ 满足非功能需求：可访问性、安全性、可扩展性等

#### 🎯 实践建议

> 每当直觉说"不值得花时间做那个"时，**发一个 prompt 试试**。最坏的结果不过是 10 分钟后发现不值得那些 token。

---

### 2.2 囤积你会做的事情

**核心思想：** 你掌握的每一种技术能力，都是未来 Agent 的燃料。

#### 为什么要囤积？

- 理论上可能 ≠ 实际见过能跑的代码
- 积累"这个问题能用什么技术解决"的答案库
- 每个解决方案都成为 Agent 的输入素材

#### 如何囤积？

| 方式 | 说明 |
|------|------|
| **博客/TIL** | 记录你解决过的问题 |
| **GitHub 仓库** | 收集 proof-of-concept 代码 |
| **HTML 工具集** | 单页 HTML + JS + CSS 解决特定问题 |
| **研究仓库** | Agent 研究问题后返回的代码 + 报告 |

#### 组合已有方案（关键 Prompt 模式）

```
这段代码展示了如何将 PDF 转为图片：[代码片段]
这段代码展示了如何 OCR 图片：[代码片段]
用这些例子组合一个单页 HTML 工具，支持拖拽 PDF、逐页转 JPEG、运行 OCR、显示结果。
```

**效果：** Agent 能把两个独立的代码片段完美组合成新工具。

#### Agent 时代更强大的用法

```bash
# 让 Agent 从你的工具库获取源码并组合
curl 获取 https://tools.simonwillison.net/ocr 和 https://tools.simonwillison.net/gemini-bbox 的源码，构建一个新工具...

# 让 Agent 参考已有项目的做法
给 ~/dev/ecosystem/datasette-oauth 添加 mock HTTP 测试，参考 ~/dev/ecosystem/llm-mistral 的方式

# 让 Agent 克隆你的仓库学习
克隆 simonw/research 到 /tmp，找 Rust 编译为 WebAssembly 的例子，用它构建一个 demo
```

**关键洞察：** Coding Agent 意味着我们只需要**搞清楚一次**某个技巧。只要用可运行的代码示例记录下来，Agent 就能在未来复用。

---

### 2.3 AI 应该帮我们产出更好的代码

**核心观点：** 如果用 Agent 导致代码质量下降，那是你的**选择问题**，不是必然结果。

#### 避免产生技术债

常见的技术债类型（都是"概念简单但耗时"的改动）：
- API 设计不覆盖后来出现的重要场景 → 修几十个地方太慢，就加了个重复 API
- 命名选错了（teams vs groups）→ 全改太累，只在 UI 层修
- 系统长出了重复但略有不同的功能 → 需要合并重构
- 文件膨胀到几千行 → 理想情况应该拆分模块

#### Agent 可以处理这些重构

```
启动一个 Agent，告诉它改什么，让它在分支/工作区后台运行。
```

推荐使用异步 Agent：
- Gemini Jules
- OpenAI Codex Web
- Claude Code Web

**评估流程：** PR 出来 → 好就合并 → 差不多就再提示 → 差就丢掉

> 这些改进的成本已经低到可以对"小代码异味"零容忍。

#### 让 Agent 帮你探索更多选项

```
Redis 适合作为预期数千并发用户的动态 feed 的选择吗？
```

Agent 可以从一个 prompt 构建模拟系统 + 负载测试 → 低成本验证技术选型。

#### 拥抱复合工程循环

Every 公司的做法（Compound Engineering）：
1. 每个编码项目结束 → **回顾**
2. 把有效的做法**文档化**
3. 文档成为未来 Agent 运行的输入
4. **小改进会复合累积**

---

### 2.4 反模式：要避免的事情

#### ❌ 把未审查的代码甩给协作者

**绝对不要提交你没审查过的 PR。**

一个好的 Agentic PR 应该：
- ✅ 代码能工作，你有信心它能工作
- ✅ 变更足够小，便于高效审查
- ✅ 包含额外上下文（高层目标、相关 issue 链接）
- ✅ PR 描述你自己审查过（Agent 写的描述也要审！）

**建议：** 提供你手动测试过的证据 —— 截图、视频、具体实现选择的说明。

---

## 3. 与 Coding Agent 协作

### 3.1 理解 Coding Agent 的工作原理

```
┌─────────────────────────────────────────────────┐
│                  Coding Agent                    │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │   LLM    │  │ System   │  │    Tools     │  │
│  │ (模型)   │←→│ Prompt   │  │ (Bash, etc.) │  │
│  └──────────┘  └──────────┘  └──────────────┘  │
│       ↕                            ↕            │
│  ┌──────────────────────────────────────────┐   │
│  │            工具循环 (Loop)               │   │
│  │  Prompt → LLM → 调用工具 → 结果 → LLM   │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

#### 关键组件

| 组件 | 说明 |
|------|------|
| **LLM** | 核心模型（GPT-5、Claude、Gemini 等） |
| **Token** | LLM 不直接处理文字，而是 token 整数序列 |
| **Chat 模板** | 模拟对话格式（user/assistant 交替） |
| **Token 缓存** | 常见前缀可复用计算，降低费用 |
| **工具调用** | Agent 的定义特征 — 函数调用 + 结果回传 |
| **System Prompt** | 隐藏指令，告诉模型如何行为（可达数百行） |
| **推理 (Reasoning)** | 模型在回复前"思考"，花更多 token 换更好结果 |

#### LLM 的无状态性

每次调用都是从零开始。要维持对话，软件需要**重放整个对话历史** → 对话越长，每次调用越贵。

---

### 3.2 使用 Git 与 Coding Agent 协作

#### 基础 Prompt

| Prompt | 作用 |
|--------|------|
| `Start a new Git repo here` | 初始化仓库 |
| `Commit these changes` | 创建提交 |
| `Add username/repo as a github remote` | 配置 GitHub |
| `Review changes made today` | 查看最近变更（好用！加载上下文） |
| `Integrate latest changes from main` | 合并主分支最新代码 |
| `Sort out this git mess for me` | 解决 Git 混乱（合并冲突等） |
| `Find and recover my code that does ...` | 找回丢失的代码 |
| `Use git bisect to find when this bug was introduced` | 二分查找 bug 引入点 |

#### 高级：重写历史

Git 历史不是固定记录，而是**刻意编写的项目进展故事**。

| Prompt | 作用 |
|--------|------|
| `Undo last commit` | 撤销上次提交 |
| `Remove uv.lock from that last commit` | 从提交中移除单个文件 |
| `Combine last three commits with a better commit message` | 合并提交 + 改写消息 |
| `Start a new repo at /tmp/xxx and build a library there with the lib/xxx.py module from here - build a similar commit history` | 从旧仓库提取代码到新仓库，保留历史 |

> Agent 在 commit message 上的品味通常很好，甚至经常比你自己写的更好。

---

### 3.3 子 Agent (Subagents)

**为什么需要子 Agent？** LLM 有上下文限制（通常 ~1M token，200K 以下质量更好）。子 Agent 用全新上下文窗口处理任务，保护主 Agent 的宝贵上下文。

#### 三种使用模式

**1. 探索型子 Agent**
```
Agent 自动派遣子 Agent 探索代码库，返回摘要。
不消耗主 Agent 的 token。
```

**2. 并行子 Agent**
```
同时运行多个子 Agent → 显著提速
可用更快更便宜的模型（如 Claude Haiku）
```

**Prompt 示例：**
```
用子 Agent 找到并更新所有受此变更影响的模板
```

**3. 专家型子 Agent**

| 角色 | 职责 |
|------|------|
| 代码审查 Agent | 审查代码，找出 bug、功能缺口、设计弱点 |
| 测试运行 Agent | 运行测试，隐藏详细输出，只报告失败 |
| 调试 Agent | 专门调试问题，推理代码路径找根因 |

#### 支持子 Agent 的工具

- OpenAI Codex subagents
- Claude subagents
- Gemini CLI subagents
- Cursor Subagents
- VS Code Copilot subagents

---

## 4. 测试与质量保证

### 4.1 红/绿 TDD

**"Use red/green TDD" — 四个词包含大量工程纪律。**

#### 流程

```
1. 🔴 红色阶段：先写测试 → 确认测试失败
2. 🟢 绿色阶段：写实现代码 → 确认测试通过
```

#### 为什么特别适合 Agent？

- Agent 可能写出不工作的代码 → 测试保护
- Agent 可能写出多余代码 → 测试定义了需求
- 测试通过 ≠ 功能正确 → 但至少排除了常见错误
- 随着项目增长，回归测试越来越重要

#### ⚠️ 关键：必须先确认测试失败

跳过这步 → 风险是测试本身已经通过了，没有真正验证新实现。

#### Prompt 示例

```
构建一个 Python 函数从 markdown 字符串提取标题。使用红/绿 TDD。
```

---

### 4.2 先运行测试

**"First run the tests" — 另一个四词魔法咒语。**

#### 作用

| 效果 | 说明 |
|------|------|
| 告诉 Agent 有测试套件 | 迫使它学会如何运行测试 |
| 了解项目规模 | 测试数量暗示复杂度 |
| 建立测试心态 | Agent 后续更倾向于扩展测试 |

#### Prompt 示例

```
First run the tests
```
或者更具体：
```
Run "uv run pytest"
```

---

### 4.3 Agent 手动测试

**代码通过测试 ≠ 功能正常。** 自动化测试无法替代手动测试。

#### Python 库测试

```
用 `python -c` 在一些边界情况上测试那个新函数
```

#### Web API 测试

```
启动开发服务器，用 `curl` 探索那个新的 JSON API
```

> "explore" 这个词会让 Agent 自动尝试 API 的多个方面。

#### 浏览器自动化测试

**Playwright** — 最强大的浏览器自动化工具（微软开源）

```
用 Playwright 测试那个
```

**Rodney** — Simon Willison 的 Chrome DevTools Protocol 工具

```
启动开发服务器，然后用 `uvx rodney --help` 测试新主页，看截图确认菜单位置正确
```

三个技巧：
1. `uvx rodney --help` → 自动安装 Rodney
2. `--help` 输出专为 Agent 设计
3. "look at screenshots" → 提示 Agent 用视觉能力评估页面

#### 用 Showboat 记录测试过程

```
运行 `uvx showboat --help`，然后创建 notes/api-demo.md showboat 文档，用它测试和记录那个新 API
```

Showboat 命令：
- `note` → 添加 Markdown 注释
- `exec` → 执行命令并记录输出（防止 Agent 作弊）
- `image` → 添加图片（配合 Rodney 截图）

---

## 5. 理解代码

### 5.1 线性代码走查

用 Agent 构建结构化的代码走查文档。

#### Prompt 示例

```
阅读源码，然后规划一个线性走查，详细解释代码如何工作

然后运行 "uvx showboat --help" 学习 showboat - 用 showboat 创建 walkthrough.md 文件，
用 showboat note 写注释，用 showboat exec 加 sed/grep/cat 包含你讨论的代码片段
```

**关键：** 让 Agent 用 shell 命令提取代码片段，而不是手动复制 → 避免幻觉。

#### 价值

- 快速理解新代码库
- 即使是 40 分钟 vibe coding 的玩具项目，也能成为学习新生态系统的机会
- 产出可复用的文档

---

## 6. 实用 Prompt 模板

### 6.1 原型开发（Artifacts/Canvas）

**自定义指令：**
```
不要在 artifact 中使用 React - 始终使用原生 HTML 和 vanilla JavaScript 和 CSS，最小依赖。

CSS 用两个空格缩进，以这段开头：
* { box-sizing: border-box; }

输入框和 textarea 字体大小 16px。字体优先 Helvetica。
JavaScript 用两个空格缩进。
标题用 Sentence case。
```

### 6.2 校对

```
你是即将发布的文章的校对员。
1. 找出拼写错误和打字错误
2. 找出语法错误
3. 注意重复用词
4. 发现逻辑错误或事实错误
5. 标记可以加强的薄弱论点
6. 确保没有空的或占位链接
```

### 6.3 Alt 文本

```
为用户粘贴的任何图片写 alt text。Alt text 始终放在 fenced code block 中。
始终在一行内呈现，方便 Markdown 图片使用。
图片上的所有文字必须完整包含。简短的图片性质描述放在前面。
```

### 6.4 播客精华

```
你会收到一个播客节目的文字稿。找出最有趣的引言 - 
最能体现整体主题的引言，引入惊人想法或表达特别清晰/有趣/犀利的引言。
只回答这些引言 - 长引言没问题。
```

---

## 7. 速查清单

### 🚀 开始新 Agent 会话时

- [ ] `First run the tests` — 让 Agent 了解项目
- [ ] `Review changes made today` — 加载最近上下文
- [ ] 确认 Agent 了解项目结构

### 📝 写代码时

- [ ] 使用 `Use red/green TDD` 确保质量
- [ ] 用子 Agent 处理探索和并行任务
- [ ] 让 Agent 手动测试（`python -c`、`curl`、Playwright）

### 🔍 代码审查时

- [ ] 自己先审查，不要把未审查的代码甩给别人
- [ ] PR 要小、有上下文、有测试证据
- [ ] Agent 写的 PR 描述也要审查

### 📚 理解代码时

- [ ] 用线性走查 + Showboat 生成文档
- [ ] 让 Agent 用 shell 命令提取代码（避免幻觉）

### 🔄 持续改进时

- [ ] 每个项目结束做回顾
- [ ] 把有效的做法文档化
- [ ] 文档成为未来 Agent 运行的输入（复合工程）

### 💡 心态转变

- [ ] "不值得花时间做" → **试试发个 prompt**
- [ ] 囤积你会的每个技术能力 → Agent 的燃料
- [ ] 重构太麻烦？→ Agent 可以后台做
- [ ] 技术选型不确定？→ Agent 可以构建原型验证

---

## 参考资源

- 原始指南：https://simonwillison.net/guides/agentic-engineering-patterns/
- Simon Willison 博客：https://simonwillison.net
- TIL 博客：https://til.simonwillison.net
- HTML 工具集：https://tools.simonwillison.net
- 研究仓库：https://github.com/simonw/research
- Showboat：https://github.com/simonw/showboat
- Rodney：https://github.com/simonw/rodney

---

*本指南基于 Simon Willison 的 Agentic Engineering Patterns 整理，旨在提供可直接落地的实践参考。*
