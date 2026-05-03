# CTO Skill — AI CTO for Claude Code

> 把你的产品想法变成完整的、可执行的软件设计方案 —— 通过一场自然对话。

一个 [Claude Code](https://claude.ai/code) Skill，让 AI 扮演一位经验丰富的 CTO，通过对话帮你完成从模糊想法到技术方案的全过程。

**不需要你懂技术。** 你只需要描述你的业务，CTO 会帮你产出三份文档：
- `brief.md` — 产品定义（业务语言，非技术人员可读）
- `arch.md` — 架构决策（技术方案，coding agent 可执行）
- `specs/features/*.md` — 功能规格（逐个功能的详细设计）

产出的设计文档可以直接交给 Claude Code、Cursor、Codex 等 coding agent 去实现。

## 特性

### CTO 对话引擎
- 像和一个真实的技术合伙人对话 —— 不是填表格
- 每轮最多问一个问题，大多数细节 CTO 自行决策
- 在关键的"命运岔路口"（单体 vs 微服务、一致性 vs 可用性等）才停下来让你选
- 全程说业务语言，不抛技术术语

### CTO Hub — 10 位传奇 CTO 的决策人格
设计过程中，CTO 会调用这些大佬的视角做**压力测试**：

| 人格 | 身份 | 核心视角 |
|------|------|---------|
| Werner Vogels | Amazon CTO | Everything fails all the time |
| Linus Torvalds | Linux/Git 创造者 | 数据结构决定一切 |
| Patrick Collison | Stripe CEO | API 即产品 |
| Martin Fowler | ThoughtWorks | 演进式架构 |
| DHH | Basecamp CTO | 简单胜过一切 |
| Kelsey Hightower | Google | No code is the best code |
| Charity Majors | Honeycomb CTO | 看不见就不存在 |
| Jeff Dean | Google 首席科学家 | Design for the next 10x |
| Andrej Karpathy | Tesla/OpenAI | AI = Software 2.0 |
| Will Larson | Stripe CTO | 工程问题 = 组织问题 |

### 全栈编码参考库
设计完成后，所有相关的编码规范、最佳实践会随设计文档一起打包交付给 coding agent：

| 分类 | 内容 | 来源 |
|------|------|------|
| **编码规范** | Python, Go, JavaScript, TypeScript, Rust, SQL | Google, Uber, Airbnb, 官方 |
| **前端** | React/Next.js, Vue/Nuxt | Vercel, Vue 官方, Nuxt |
| **后端** | Go (42 skills), Python/FastAPI | samber, zhanymkanov |
| **设计** | API 设计, 数据库 Schema, 迁移, 设计系统 | Microsoft, Google, PostgreSQL |
| **评审** | Code Review 指南 | Google |
| **安全** | 13 类 SAST 漏洞检测 | utkusen |
| **方法论** | TDD, CI/CD, 调试, 发布等 17 种 | Addy Osmani |

## 安装

### 方式一：全局安装（推荐）

```bash
# 克隆仓库
git clone https://github.com/yzfly/CTO-Skills.git

# 复制 skill 到 Claude Code skills 目录
cp -r CTO-Skills/skills/cto ~/.claude/skills/
```

安装后在 Claude Code 中直接使用 `/cto` 即可触发。

### 方式二：项目级安装

```bash
# 在你的项目根目录
git clone https://github.com/yzfly/CTO-Skills.git
cp -r CTO-Skills/skills/cto .claude/skills/
```

### 方式三：下载 .skill 安装包（无需 git）

1. 去 [Releases 页面](https://github.com/yzfly/CTO-Skills/releases/latest) 下载最新的 `cto.skill` 文件
2. 解压到 Claude Code skills 目录：

```bash
unzip cto.skill -d ~/.claude/skills/
```

`.skill` 文件本质是 zip 压缩包，解压后会得到 `cto/` 目录，正好放在 `~/.claude/skills/` 下。

### 验证安装

在 Claude Code 中输入：

```
我想做一个创作者提现系统
```

或

```
I have an idea for a team knowledge base
```

CTO 会自动接管对话，开始帮你设计。

## 使用方式

### 触发方式

以下表述都会触发 CTO skill：

| 语言 | 示例 |
|------|------|
| 中文 | "我想做一个..."、"我有个想法"、"帮我设计..." |
| English | "I want to build..."、"I have an idea for..."、"Help me design..." |

### 对话流程

```
你: 我想做一个给自由职业者用的发票工具

CTO: [理解你的想法，问一个关键问题]

你: [回答]

CTO: [默默做了20个技术决策，只在关键岔路口问你]
     ...
     [几轮对话后]
     我觉得方案差不多了，要不要看看我们整理出来的设计文档？

→ 产出 brief.md + arch.md + specs/features/*.md + 编码参考库
```

### 产出示例

```
invoice-tool/
├── brief.md                    # 产品定义
├── arch.md                     # 架构决策（含 Coding Practices 附录）
├── specs/features/
│   ├── invoice-create.md       # 创建发票
│   ├── client-manage.md        # 客户管理
│   └── payment-track.md        # 收款追踪
└── references/                 # 编码参考（给 coding agent）
    ├── standards/python.md
    ├── backend/python-fastapi/
    ├── frontend/vue-nuxt/
    ├── design/database/
    ├── review/
    ├── security/
    └── methods/
```

## 目录结构

CTO Skill 采用**三层 references 架构**，每一层职能不同、读者不同、加载规则不同：

```
skills/cto/
├── SKILL.md                            # CTO 人格 + 对话引擎
└── references/
    ├── README.md                       # 三层架构总索引
    │
    ├── runtime/                        # Layer 1: CTO 对话时载入 (~64KB)
    │   ├── industry-reflexes.md        #   10 行业反射弧
    │   ├── tradeoffs.md                #   15 架构矛盾 + ADR 模板
    │   ├── templates.md                #   brief/arch/specs schema
    │   └── stack-practices.md          #   桥接层 (决定打包什么)
    │
    ├── personas/                       # Layer 2: 第二意见, 按场景调用 (~48KB)
    │   ├── README.md                   #   场景→人格 索引 + 人格冲突表
    │   └── *.md                        #   10 位 CTO 决策人格
    │
    └── payload/                        # Layer 3: 下游交付物, CTO 不读 (~3.7MB)
        ├── README.md
        ├── standards/                  #   编码规范 (6 语言)
        ├── frontend/                   #   前端 (React/Next.js, Vue/Nuxt)
        ├── backend/                    #   后端 (Go 42 skills, Python/FastAPI)
        ├── design/                     #   设计 (API, 数据库, 系统)
        ├── review/                     #   代码评审
        ├── security/                   #   安全扫描 (13 SAST)
        └── methods/                    #   工程方法论 (17 种)
```

### 三层职能

| 层 | 谁读 | 何时载入 | 体积 |
|----|------|---------|------|
| **runtime/** | CTO 自己 | 触发条件命中时, 懒加载 | 64KB |
| **personas/** | CTO 自己 | 关键决策点查"第二意见" | 48KB |
| **payload/** | 下游 coding agent | CTO 设计完, 按 stack 打包给用户项目 | 3.7MB |

CTO 永远不会把 `payload/` 载入对话——只在最终交付时按 `runtime/stack-practices.md` 的清单打包对应子集到用户项目的 `<project>/references/`。

## 致谢

本项目站在巨人的肩膀上，整合了以下优秀的开源项目和权威资源：

### 编码规范

| 资源 | 来源 | Stars | 许可 |
|------|------|-------|------|
| [Google Python Style Guide](https://github.com/google/styleguide) | Google | 39K+ | Apache-2.0 |
| [Google TypeScript Style Guide](https://github.com/google/styleguide) | Google | 39K+ | Apache-2.0 |
| [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) | Airbnb | 148K+ | MIT |
| [Uber Go Style Guide](https://github.com/uber-go/guide) | Uber | 17K+ | Apache-2.0 |
| [Rust API Guidelines](https://github.com/rust-lang/api-guidelines) | Rust 官方 | 1.3K+ | MIT / Apache-2.0 |
| [SQL Style Guide](https://github.com/treffynnon/sqlstyle.guide) | Simon Holywell | 1.5K+ | CC-BY-SA-4.0 |

### 前端

| 资源 | 来源 | Stars | 许可 |
|------|------|-------|------|
| [React Best Practices](https://github.com/vercel) | Vercel Engineering | - | MIT |
| [Vue.js Style Guide](https://github.com/vuejs/docs) | Vue.js 官方 | - | MIT |
| [Nuxt Best Practices](https://github.com/onmax/nuxt-skill-hub) | onmax | 41+ | - |

### 后端

| 资源 | 来源 | Stars | 许可 |
|------|------|-------|------|
| [Go Skills (42 skills)](https://github.com/samber/cc-skills-golang) | samber | 1.5K+ | MIT |
| [FastAPI Best Practices](https://github.com/zhanymkanov/fastapi-best-practices) | zhanymkanov | 17K+ | MIT |

### 设计

| 资源 | 来源 | Stars | 许可 |
|------|------|-------|------|
| [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines) | Microsoft | 23K+ | CC-BY-4.0 |
| [DESIGN.md Specification](https://github.com/google-labs-code/design.md) | Google Labs | 11K+ | Apache-2.0 |
| [Database Schema Designer](https://github.com/borghei/Claude-Skills) | borghei | 104+ | - |
| [PostgreSQL Guide](https://github.com/simplyblock/vela-skill) | simplyblock | 2+ | MIT |
| [SQL Schema Generator](https://github.com/majiayu000/claude-skill-registry) | majiayu000 | 261+ | MIT |
| [PostgreSQL "Don't Do This"](https://wiki.postgresql.org/wiki/Don%27t_Do_This) | PostgreSQL Wiki | - | CC-BY |

### 评审

| 资源 | 来源 | Stars | 许可 |
|------|------|-------|------|
| [Google Engineering Practices](https://github.com/google/eng-practices) | Google | 20K+ | CC-BY-3.0 |

### 安全

| 资源 | 来源 | Stars | 许可 |
|------|------|-------|------|
| [SAST Skills](https://github.com/utkusen/sast-skills) | utkusen | 633+ | MIT |

### 工程方法论

| 资源 | 来源 | Stars | 许可 |
|------|------|-------|------|
| [Agent Skills](https://github.com/addyosmani/agent-skills) | Addy Osmani | 27K+ | MIT |
| [AGENT-ZERO](https://github.com/msitarzewski/AGENT-ZERO) | msitarzewski | 200+ | Unlicense |

## 许可

本项目原创内容（SKILL.md、CTO Hub 人格文件、桥接层文件）采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) 许可。

引用的第三方内容保持各自原始许可（见上方致谢表格）。使用时请遵守各项目的许可条款。

## 作者

**云中江树** | 微信公众号: 云中江树

---

> **CTO Skill 的设计哲学：** 用户带着一个想法来找你，不想操心技术。你唯一的目标是让他们感觉在和一个真正懂行的人对话——最后，手里拿着一份完整的、可构建的设计方案，而他们没为此流过一滴汗。
