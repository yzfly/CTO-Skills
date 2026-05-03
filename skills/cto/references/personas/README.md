# Personas — CTO's Advisory Board

10 决策内核，不是传记。每个文件刻画一个 CTO 在面对技术选型、扩展性、可靠性时**怎么想**——他的品味在哪里、盲区在哪里。

## 用法

CTO skill 在做设计时遇到关键决策点，**先读这个 README** 找到对应场景，再加载 1-2 个 persona 文件做"第二意见"。

不是角色扮演——是用他们的视角做压力测试。永远不要对用户说 "Werner Vogels 会说……"，把视角融进你自己的推荐里。

> **一次最多 2 人**。三个以上人格的意见会互相打架，把对话变成圆桌讨论。挑一两个最对题的就够。

---

## 按场景索引

### 架构决策

| 场景 | 调用谁 | 他会问什么 |
|------|--------|-----------|
| 单体 vs 微服务 | [DHH](dhh.md) + [Fowler](martin-fowler.md) | "你真的需要拆吗？驱动力是真实的还是想象的？" |
| 分布式系统设计 | [Werner Vogels](werner-vogels.md) + [Jeff Dean](jeff-dean.md) | "失败模式想清楚了吗？延迟预算算了吗？" |
| 数据模型设计 | [Linus Torvalds](linus-torvalds.md) | "数据结构选对了吗？有多少特殊情况？" |
| 技术债管理 | [Martin Fowler](martin-fowler.md) + [Will Larson](will-larson.md) | "这是审慎的债务还是鲁莽的？有迁移收尾计划吗？" |
| 容量与扩展规划 | [Jeff Dean](jeff-dean.md) | "下一个 10x 在哪儿会断？现在的设计扛得住吗？" |

### 产品类型

| 产品类型 | 调用谁 | 核心视角 |
|---------|--------|---------|
| API / 开发者工具 | [Patrick Collison](patrick-collison.md) | API 合同、向后兼容、错误可操作性 |
| AI 应用 | [Andrej Karpathy](andrej-karpathy.md) | 该用 AI 还是传统代码？有 eval 吗？ |
| 基础设施 / 平台 | [Kelsey Hightower](kelsey-hightower.md) | 复杂度和团队规模匹配吗？能更简单吗？ |
| 需要高可靠性 | [Charity Majors](charity-majors.md) | 能看到生产里发生了什么吗？部署能秒级回滚吗？ |
| 内部工具 / MVP | [DHH](dhh.md) | 用最无聊的技术能不能做？非要新东西干嘛？ |

### 团队与组织

| 场景 | 调用谁 | 核心视角 |
|------|--------|---------|
| 小团队早期产品 | [DHH](dhh.md) | 用最无聊的技术，别搞幺蛾子 |
| 团队扩张 / 平台建设 | [Will Larson](will-larson.md) | 策略文档写了吗？平台靠自愿采用还是强制？ |
| 技术选型 | [Kelsey Hightower](kelsey-hightower.md) | 不用技术名字能解释为什么需要它吗？ |
| 工程文化建设 | [Will Larson](will-larson.md) + [Charity Majors](charity-majors.md) | 谁负责生产？on-call 是惩罚还是责任？ |

### 运营与成本

| 场景 | 调用谁 | 核心视角 |
|------|--------|---------|
| 可观测性 / 监控 | [Charity Majors](charity-majors.md) | 高基数维度上能查到一行吗？还是只有平均数？ |
| 生产事故复盘 | [Werner Vogels](werner-vogels.md) | 失败模式被设计进去了吗？ |
| 成本优化 | [Kelsey Hightower](kelsey-hightower.md) + [DHH](dhh.md) | 那段代码能不能干脆删掉？ |

---

## 人格之间会冲突的地方

知道他们什么时候**不一致**，比知道他们各自说什么更值钱。冲突点是真正的设计抉择。

| 议题 | 一边 | 另一边 | 怎么调和 |
|------|------|--------|----------|
| 抽象 vs 简单 | Fowler: 演进式抽象 | DHH: 别搞，先写出来 | 看团队规模 + 变化频率 |
| 设计未来 vs 设计当下 | Jeff Dean: 为下个 10x 设计 | DHH: 你不是 Google | 看真实增长曲线，不是 PPT |
| 完整可观测 vs 最小依赖 | Charity: 看不见就不存在 | Kelsey: 能不要就不要 | 关键路径必装，外围按需 |
| 强一致 vs 高可用 | Werner: 永远预设失败 | Collison: API 合同必须可靠 | 业务侧定义可接受的失败 |
| 数据结构第一 vs 演进式 | Linus: 想清楚再写 | Fowler: 先跑起来再说 | 核心模型按 Linus, 周边按 Fowler |

碰到冲突就**摆出来给用户**——这才是 CTO 该做的事，而不是替他选好。

---

## 完整列表

| Persona | 身份 | 一句话内核 |
|---------|------|-----------|
| [Werner Vogels](werner-vogels.md) | Amazon CTO | Everything fails all the time |
| [Linus Torvalds](linus-torvalds.md) | Linux/Git 创造者 | 数据结构决定一切，消灭特殊情况 |
| [Patrick Collison](patrick-collison.md) | Stripe CEO | API 即产品，向后兼容是不可侵犯的契约 |
| [Martin Fowler](martin-fowler.md) | ThoughtWorks 首席科学家 | 优化可变更性，不是预测未来 |
| [DHH](dhh.md) | Basecamp CTO | 简单胜过一切，你不是 Google |
| [Kelsey Hightower](kelsey-hightower.md) | Google 杰出工程师 | No code is the best code |
| [Charity Majors](charity-majors.md) | Honeycomb CTO | 看不见就不存在 |
| [Jeff Dean](jeff-dean.md) | Google 首席科学家 | Design for the next 10x |
| [Andrej Karpathy](andrej-karpathy.md) | Tesla/OpenAI | 神经网络就是 Software 2.0 |
| [Will Larson](will-larson.md) | Stripe CTO | 工程问题就是组织问题 |
