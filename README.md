# graph-engineering · 基于图的 Agent 编排工程

> **一句话定位**：把 agent 执行建模为显式图，其工程价值不在"画图"，而在 checkpoint / 恢复 / 并行 / 人工审批这些基础设施。Graph Orchestration ≠ GraphRAG（知识工程域，不在本目录范围）。

## 文档导航

| 文档 | 内容 |
|---|---|
| [01-技术研究规范.md](01-技术研究规范.md) | 七层规范：执行模型 / 持久化 / HITL / 容错 / durable execution 分层 / 多 agent / 选型，条款编号【G#.#】 |
| [02-最佳实践.md](02-最佳实践.md) | 选型决策树、状态与 HITL checklist、durable 引入判断、多 agent 模式表、报错速查、部署清单 |
| [03-参考资料.md](03-参考资料.md) | 30 条一手来源（LangGraph / MS AF / ADK / OpenAI SDK / Temporal / Restate / DBOS…，含验证状态） |
| [04-国内厂商研究.md](04-国内厂商研究.md) | 国内生态：字节 Eino + Coze Studio（同源自源完整栈）、阿里 Spring AI Alibaba / AgentScope、Dify Human Input、腾讯/百度平台、durable execution 国内空白带、国内框架对照表 |

## 核心结论（2026-09 调研）

1. LangGraph 1.x（当前 1.2.x）是稳定性发布；`create_react_agent` 已废弃，官方 supervisor/swarm 包**已归档**（多 agent 组合转向 middleware + deepagents）。
2. 图的并发语义有硬规则：fan-out 同步写同 key 必须声明 reducer；interrupt 恢复是**节点级重放**，副作用必须幂等，循环内 interrupt 会指数重放。
3. 2026 年最大行业争议："checkpointing ≠ durable execution"（Temporal / Restate 立场）——进程死了 run 就死；生产架构正在收敛为**两层**：agent framework + durable orchestrator。
4. durable execution 有代价：工作流必须确定性、DX 与成本开销、"not a silver bullet"——按"进程死了 run 是否必须活着"这一判据决定是否引入。
5. 多 agent handoff 只在单 run 内有效，guardrails 只覆盖链首与末端；跨 agent 校验必须显式设计。
6. 生产案例（Klarna 700 FTE 等效、LinkedIn SQL Bot）的共同点：价值来自基础设施与检索工程，不是图拓扑。
7. **国内增量**：Go 栈选 Eino（Graph 含环 / Workflow 字段映射无环，Coze Studio 构建其上）、Java 栈选 Spring AI Alibaba（内置 HITL）、消息派选 AgentScope；国内模型厂不开编排层；agent 场景 durable execution 是空白带，生产需自建持久层。

## 与其他目录的关系

- **降级路径**：单循环即可满足时不要上图 → [`../loop-engineering/`](../loop-engineering/README.md)（Anthropic 与 LangGraph 官方口径一致：固定路径用 workflow，动态决策才用 agent）
- **应用语境**：为客户现场选型/交付编排层时的角色与方法 → [`../fde/`](../fde/README.md)
- **历史调研**：Pregel 细节、最小示例、首版框架对比 → [`../fde/02-前沿实现调研-脚手架-Loop-Graph.md`](../fde/02-前沿实现调研-脚手架-Loop-Graph.md) §3
