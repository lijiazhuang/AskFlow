<p align="center">
  <a href="https://github.com/nageoffer/ragent">
    <picture>
      <source srcset="assets/ragent-ai-banner.png">
      <img src="assets/ragent-ai-banner.png" alt="Ragent AI">
    </picture>
  </a>
</p>

<p align="center">
  <strong>后端程序员转型 AI 工程师的第一站</strong><br/>
</p>

<p align="center">
  <a href="https://github.com/nageoffer/ragent/stargazers"><img alt="GitHub stars" src="https://img.shields.io/github/stars/nageoffer/ragent?style=flat-square&logo=github&color=e8b227" /></a>&nbsp;
  <a href="https://github.com/nageoffer/ragent/network/members"><img alt="GitHub forks" src="https://img.shields.io/github/forks/nageoffer/ragent?style=flat-square&logo=github&color=2d6a8a" /></a>&nbsp;
  <a href="https://github.com/nageoffer/ragent/graphs/contributors"><img alt="Contributors" src="https://img.shields.io/github/contributors/nageoffer/ragent?style=flat-square&color=b56e7a" /></a>&nbsp;
  <a href="./LICENSE"><img alt="License" src="https://img.shields.io/badge/license-Apache--2.0-4a9b8f?style=flat-square" /></a>
  <img src="https://img.shields.io/badge/Spring%20Boot-4.1-6DB33F?style=flat-square&logo=springboot&logoColor=white" />
  <img src="https://img.shields.io/badge/AgentScope-2.0-4a6fa5?style=flat-square" />
</p>

## 🚀 什么是 AskFlow？

AskFlow 是一个面向 RAG 演进的生产级 Java AI 应用平台，覆盖从文档入库到智能问答的完整链路。

- **混合检索**：向量、关键词、知识图谱、联网搜索并行召回，支持去重、RRF 融合与 Rerank。
- **问题理解**：支持查询词映射、问题重写与拆分、树形意图识别和多知识库路由。
- **模型与工具**：支持模型档位、首包探测、熔断降级，以及 AgentScope 的 MCP 工具发现与调用。
- **会话记忆**：最近 N 轮消息结合持久化摘要，控制 Token 成本并保留关键上下文。
- **流量保护**：Redis 公平排队与分布式并发控制，避免突发请求压垮模型服务。
- **知识闭环**：提供可编排入库 Pipeline、远程刷新、回答溯源、用户反馈、Trace 和管理后台。



## 🧭 快速导航

> 觉得不错？先点个 Star 收藏，所有链接都会在新标签页打开，不会离开本页 👇

<p align="left">
  <a href="https://github.com/nageoffer/ragent/stargazers">
    <img src="assets/star-me.svg" alt="Star this repo" />
  </a>
</p>

| &nbsp; | 链接 | 说明 |
|:---:|:---|:---|
| 📖 | <a href="https://nageoffer.com/ragent" target="_blank">官网文档</a> | Ragent AI 完整文档 |
| 🚀 | <a href="https://nageoffer.com/ragent/demo/" target="_blank">在线体验</a> | 无需部署，直接体验 Ragent AI |
| ⚡ | <a href="https://nageoffer.com/ragent/local-dev/" target="_blank">快速启动</a> | 本地搭建 Ragent AI 前后端项目 |
| 📝 | <a href="https://nageoffer.com/ragent/interview/" target="_blank">简历怎么写</a> | 项目如何写到简历上 |
| 💡 | <a href="https://nageoffer.com/ragent/why-springai-langchain4j/" target="_blank">为什么不用 Spring AI / LangChain4j</a> | 技术选型的思考 |



## 🏗️ AskFlow 核心设计

采用前后端分离的模块化单体架构，后端按职责分为七个 Maven 模块：

| 模块 | 职责                                                                                 |
|:---|:-------------------------------------------------------------------------------------|
| `infra-ai` | Chat / Embedding / Rerank / VLM 模型客户端、模型档位、路由、首包探测、健康状态与降级 |
| `system` | 用户认证与审计日志，位于各业务引擎之下的系统支撑域                                   |
| `rag` | RAG 问答、知识库、入库 Pipeline、意图树、检索、会话及管理端 API                      |
| `agent` | AgentScope ReAct 执行引擎，接入知识检索与 MCP 工具，支持技能、记忆和人工确认                             |
| `bootstrap` | 启动装配层，仅含启动类与主配置                                                       |
| `framework` | 统一响应与异常、幂等、分布式 ID、MQ 适配、SSE 与跨节点流式取消等基础能力             |
| `mcp-server` | 基于 MCP Java SDK 的独立工具示例服务，内置通用工具（日期、天气、联网搜索）、企业助手和比特严选商城三组工具 |

这个分层不是为了炫技，而是把业务编排、AI 供应商差异和通用基础设施隔离开。切换模型、向量库或对象存储时，核心问答流程不需要跟着重写。

![](assets/ragent-module-layering-v3.png)

一次提问在服务里怎么走，v1 和 v2 是两套路子。

MCP 客户端只在 `agent` 模块中装配，使用 AgentScope 连接和发现工具，服务地址配置在 `agent.mcp.servers`。意图树中的 MCP 节点仍指定可用工具及确认规则；Workflow 只识别知识库和系统意图，不连接 MCP 服务。`mcp-server` 是独立示例服务，使用与 AgentScope 一致的 MCP Java SDK 版本实现服务端协议。

- v1 是 Workflow 编排，路径写死在代码里：问题重写、意图识别、多路召回、组装生成，一步接一步走完。
- v2 换成 Agentic 架构，路径由模型自己定：知识检索、MCP 工具和 Skills 技能都变成它手里的工具，查不查、查几次、查完接着干什么，在 ReAct 循环里边推理边决定。所以链路比 v1 长，也不再是一条直线。

> 下图是 v2 的核心流程。实际项目代码中的逻辑比图上更复杂，落地过程中还涉及很多细节和优化。

![](assets/ragent-chain-v4.png)

两套并存，不是谁替代谁。问答类知识助手需求走 v1 更快也更可控，要规划、要动手操作的场景才交给 v2。

## ✨ 项目质量怎么样？

这里的质量不靠一张架构图来证明，而是看代码边界、测试、故障处理和运维闭环是否真实存在。以下数据按当前仓库统计，代码行数包含注释和空行。

### 1. 规模与完整度

- **后端**：7 个 Maven 模块，分别承载基础能力、知识库问答、Agent、启动装配和示例 MCP 服务。
- **前端**：知识库管理、意图树、Agent 技能、会话与系统配置。
- **数据与测试**：平台业务表与 `mcp-server` 演示业务表分别维护，Java 测试覆盖检索、Agent 工具调用和服务端协议。

代码量本身不等于质量，但这些模块组成了数据进入系统—检索生成答案—展示证据—收集反馈—追踪与审计的完整闭环，另有 Agent ReAct 推理与工具调用链路，不仅是简单的 API 示例。

### 2. 工程质量

- **模块边界**：通用基础设施、AI 能力、RAG 业务、Agent 引擎和 MCP 服务相互隔离，替换模型或存储实现不会侵入问答编排。
- **配置防错**：模型档位、候选能力和检索漏斗在启动阶段完成一致性校验，错误配置直接失败而不是静默降质。
- **并发治理**：10 个专用线程池隔离负载，TTL 保证用户与 Trace 上下文跨线程传递。
- **关键路径测试**：覆盖模型路由、检索预算、会话摘要、入库 Pipeline、MCP 和 Agent 上下文与长期记忆。
- **工程约束**：统一响应、错误码和异常处理，认证、幂等、线程安全 SSE 与 Spotless 格式化均已落到代码。

> 项目中大量应用并发线程，建议配合社群里的 [oneThread 动态线程池框架](https://nageoffer.com/onethread) 搭配学习收获更多。

### 3. 代码是怎么组织的

AI 写代码很强，但掌舵的还是程序员。能不能接着扩展、改一处会不会牵一片，取决于动手前有没有把结构定好。项目里的设计模式，每个都对着一个具体问题：

| 设计方式 | 业务场景 | 解决的问题 |
|:---|:---|:---|
| 策略 | 检索通道、结果后处理、文档来源 | 不同实现可独立替换 |
| 工厂 | 意图树、分块策略、流式回调创建 | 集中复杂对象的创建逻辑 |
| 模板方法 | 并行检索、模型请求 | 固定通用流程，仅开放差异步骤 |
| 注册表 | 意图节点管理 | 统一注册和查找配置 |
| 装饰器 | 向量写入时同步关键词和图谱索引 | 在不修改主流程的前提下增强能力 |
| 责任链 | 检索后处理、模型故障降级 | 按顺序组合处理步骤 |
| 中间件 | Agent 记忆注入、上下文压缩、技能遮蔽、工具批处理 | 不改执行引擎就能插拔能力 |
| 适配器 | 多家模型方言、Agent MCP 业务规则、MQ 客户端 | 屏蔽外部接口差异 |
| 状态机 | 远程文档定时刷新 | 约束状态流转，挡住非法跳转 |
| 事件回调 | 模型流式响应、首包探测、SSE 输出 | 解耦事件生产与消费 |
| AOP | 链路追踪、幂等、审计日志 | 将横切逻辑与业务解耦 |

下一节的那些扩展点，基本都是这些模式带来的结果。

### 4. 可扩展性

核心能力通过接口、注册表和配置隔离，新增实现可以复用现有编排、容错、日志和管理能力：

| 扩展维度 | 如何接入 | 接入后的效果 |
|:---|:---|:---|
| 模型 | 实现 `ChatClient` / `EmbeddingClient` / `RerankClient`，加入模型候选配置 | 新供应商可进入模型档位与候选路由，复用首包探测、健康检查和熔断降级 |
| 存储 | 实现向量存取或 `ObjectStorageClient`，通过配置选择实现 | 可替换向量库或对象存储，知识入库与问答主流程保持不变 |
| 检索 | 实现 `SearchChannel` 或后处理器，注册为 Spring Bean 并设置顺序 | 新通道参与并行召回，新处理器可插入去重、融合、精排与富化链路 |
| 入库 | 实现 `IngestionNode` 或 `DocumentFetcher`，补充节点类型和配置 | 新处理步骤或文档来源进入 Pipeline，继续使用任务状态、节点日志和失败定位 |
| Agent | 新建人设与技能手册，或注册新的中间件 | 新智能体复用 ReAct 循环、记忆与人工确认，不必改动执行引擎 |
| MCP | 暴露 MCP 工具规范，在 `agent.mcp.servers` 配置外部 MCP Server | AgentScope 发现工具并将服务端 Schema 提供给模型 |

扩展的改动主要收敛在新实现和配置中，不必复制一套检索、会话或 Trace 主链路。

### 5. 生产级特性

这里的生产级特性指项目已经实现生产环境会遇到的关键机制，而不只是功能能跑：

| 特性 | 说明                                                                              |
|:---|:----------------------------------------------------------------------------------|
| **流量保护** | Redis ZSET 公平排队，结合 Lua 原子抢占、过期信号量和 Pub/Sub 唤醒                 |
| **模型容错** | 多候选模型自动切换，首包超时、空响应或异常时触发降级；三态熔断隔离故障节点        |
| **检索稳定性** | 多通道并行检索，单通道失败不影响主链；按召回、Rerank、TopK 分层控制检索规模       |
| **数据一致性** | RocketMQ 事务消息保障分块可靠执行；支持幂等，远程刷新使用分布式锁和状态机         |
| **可观测与审计** | 记录 Trace Run / Node 的耗时、输入输出和异常；管理端提供趋势、详情及配置变更 Diff |
| **流式体验** | SSE 分事件输出思考、正文、来源和推荐问题；支持全局超时及客户端断开取消            |
| **会话与证据** | 消息持久化和摘要控制上下文；保存引用来源，支持原文预览、追问和反馈                |
| **Agent 执行保障** | 上下文按水位裁剪与压缩，长期记忆跨会话沉淀；写操作可配置人工确认，支持执行中断    |
| **安全基础** | Sa-Token 认证、数据归属校验、上传限流、参数校验和统一异常处理                     |

### 6. 完整控制台

Ragent 提供覆盖**普通用户与管理员用户**的 React 控制台，不只是聊天页面，也把检索证据和运维入口暴露出来。

系统通过多轮 AI 辅助设计优化，在保证功能完整性的同时，提供更加现代化和友好的交互体验。

#### 6.1 用户问答界面

v2 面向 Agent 任务提供完整的执行过程视图。模型思考、工具调用、并行批次和最终回答会按时间线实时展开，任务做了什么、耗时多久都能直接看到。

![](assets/qa-home-v2.png)

遇到下单、修改数据等写操作时，系统会先展示操作内容和原始参数，用户确认后才会继续执行。

![](assets/qa-home-v2-confirm.png)

v1 保留更轻量的知识问答界面，适合检索路径固定、以答案和引用为主的场景。用户可以直接输入问题，也可以通过示例问题快速体验，并按需开启**深度思考模式**。

- 支持自然语言输入
- 支持示例问题快速填充
- 支持深度思考模式

![](assets/qa-home.png)

用户提交问题后，模型会实时生成回答结果，并提供良好的阅读体验：

- 支持 Markdown 格式渲染
- 支持图片内容展示
- 支持代码高亮显示
- 支持回答来源、原文预览和推荐追问
- 支持回答评价（点赞 / 点踩）

![](assets/qa-answer.png)

#### 6.2 管理后台

管理员可以通过后台查看仪表盘，管理知识库与 Chunk、知识图谱、意图树、查询词映射、入库任务、示例问题、用户和系统设置，并查看 RAG Trace 与业务变更日志。

<details>
<summary><b>管理后台界面截图</b>（点击展开）</summary>

v2 新增 Agent 运行概览，集中展示活跃会话、工具调用成功率、人工确认和上下文压缩等指标，运行状态和问题分布可以直接查看。

![](assets/admin-overview-v2.png)

同时接入 Langfuse，可按会话查看模型推理、工具调用、耗时与 Token 消耗，方便定位 Agent 链路中的具体问题。

![](assets/langfuse.png)

v1 管理后台侧重 RAG 配置和运行管理，覆盖模型、知识库、数据集、Trace 等常用功能。

![](assets/admin-overview.png)

![](assets/admin-settings.png)

![](assets/admin-knowledge-base.png)

![](assets/admin-datasets.png)

![](assets/admin-trace.png)

![](assets/admin-models.png)

</details>

控制台围绕实际使用和排障流程持续打磨，在功能完整的基础上，尽量把复杂能力做得直观、易用。

![](assets/admin-theme.png)

### 7. 和市面上项目的区别

Ragent 定位于 **Java AI 应用的源码级工程参考**，重点是完整链路、生产保障和二次开发能力。

| 对比维度 | 常见 RAG 教程 / Demo | Ragent                                                 |
|:---|:-----------------|:-------------------------------------------------------|
| 项目定位 | 跑通检索与生成          | 完整 Java Agentic RAG 应用                             |
| 检索 | 单路向量 TopK        | 向量 / 关键词 / 图谱 / 联网召回，RRF 融合与 Rerank     |
| 问题理解 | 原问题直接检索          | 查询词映射、问题重写与拆分、树形意图和多知识库路由     |
| 模型调用 | 单模型直连            | 模型档位、首包探测与熔断降级                           |
| 工具接入 | 以应用内函数调用为主 | MCP 协议、远程工具发现、Schema 校验与写操作人工确认    |
| 智能体 | 无或单轮函数调用 | ReAct 多轮推理、技能体系、执行中断与恢复               |
| 知识入库 | 一次性脚本            | 可编排 Pipeline、节点日志、远程文档定时刷新            |
| 会话记忆 | 以近期消息拼接为主 | 最近 N 轮消息 + 持久化摘要；Agent 侧三层上下文管理     |
| 回答可信度 | 只展示答案            | 来源引用、原文预览与用户反馈                           |
| 运行保障 | 基础日志             | 分布式限流、幂等、事务消息、Trace 与审计               |
| 监控指标 | 通用调用量与耗时 | 基于 LangFuse 提供个性化指标，覆盖链路质量、工具调用等 |
| 二次开发 | 流程写死             | 模型、存储、检索、入库和 MCP 均提供扩展接口            |
| 管理能力 | 无或简单页面           | 完整用户端与管理后台                                   |

## ❓ 常见问题答疑

一句话：学完 Ragent，你既能跟面试官聊 RAG/Agent 的技术深度，也能证明自己的 Java 工程化水平。

<details>
<summary><b>能学到什么 / 适合谁？</b>（点击展开）</summary>

### 1. 能够学到什么？

Ragent 不只是教你调 API，而是让你理解一个 Agentic RAG 系统从 0 到 1 落地的全过程。粗略来说，你能收获这些：

- **RAG 全链路工程能力**：文档解析、分块策略、Embedding 向量化、意图识别、问题重写与拆分、多路检索、重排序、Prompt 组装、流式生成，每个环节怎么做、为什么这么做。
- **Agent 智能体实战**：ReAct 执行引擎、MCP 工具调用、技能体系、三层上下文记忆、人工确认，这些是 AI 应用区别于传统 CRUD 系统的核心能力。
- **模型工程化实践**：模型档位、多候选路由、首包探测、熔断降级，解决模型不稳定时如何保障服务可用性。
- **高质量 Java 工程能力**：分层架构、设计模式实战、分布式并发限流、多线程池管理与上下文透传、全链路追踪，这些能力不局限于 AI 项目，放到任何 Java 后端岗位都是加分项。
- **前后端完整项目经验**：后端 Spring Boot 4 + 前端 React 18，从 API 设计到页面交互，完整的全栈项目经历。


<p align="center">
  <a href="https://www.star-history.com/?repos=nageoffer%2Fragent&type=date&legend=top-left">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/image?repos=nageoffer/ragent&type=date&theme=dark&legend=top-left" />
      <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/image?repos=nageoffer/ragent&type=date&legend=top-left" />
      <img alt="Star History Chart" src="https://api.star-history.com/image?repos=nageoffer/ragent&type=date&legend=top-left" />
    </picture>
  </a>
</p>

如果屏幕前的亦菲/彦祖觉得项目还不错，点个 Star 支持一下，这是对开源作者最好的认可！
