# Harness Engineering 七大原则

本文件是审计评分的依据。每条原则对应一个评分维度（0-3分）。

---

## 原则 1：AGENTS.md 是目录，不是百科全书

**来源**: OpenAI Codex 团队

单体式的指令文件会迅速腐烂。Agent 无法判断哪些规则还有效，人类也停止维护它。

**评分标准**:
- 3分: AGENTS.md ≤ 200 行，仅包含索引、关键命令和最关键的规则
- 2分: AGENTS.md 200-400 行，有一定结构但偏长
- 1分: AGENTS.md 400-800 行，包含大量具体规则
- 0分: AGENTS.md > 800 行，或不存在

**修复方向**: 把具体规则移入 `docs/` 目录，AGENTS.md 只保留指针和高频信息。

---

## 原则 2：渐进式披露（Progressive Disclosure）

**来源**: OpenAI Harness Post, Anthropic Skill 架构

Agent 的上下文窗口是稀缺资源。信息应分层提供：

- 第一层：AGENTS.md（目录 + 最关键规则，~100行）
- 第二层：docs/ 下的主题文档（按需读取）
- 第三层：深度参考资料（仅在具体任务时加载）

**评分标准**:
- 3分: 明确的三层结构，AGENTS.md 有指针指向各层
- 2分: 有两层（AGENTS.md + 一些文档），但没有系统化
- 1分: 所有信息在一个文件里，但文件结构清晰
- 0分: 所有信息散落各处，无结构

---

## 原则 3：机器可读性优先

**来源**: Anthropic 工程团队, LangChain

给 agent 看的文档和给人看的文档有本质区别：

- 状态追踪用 JSON，不用 Markdown checklist（agent 不太会意外覆盖结构化数据）
- 命令用精确的调用格式，不用自然语言描述
- 错误消息内嵌修复指南（linter error = fix guide）
- API/CLI 接口优先于 dashboard/可视化描述

**评分标准**:
- 3分: 状态用 JSON 追踪，命令有精确格式，错误消息含修复方法
- 2分: 部分结构化，但仍有自然语言描述的命令或状态
- 1分: 基本全是 Markdown 叙述
- 0分: 无结构化内容

---

## 原则 4：可执行性（Actionability）

**来源**: Mitchell Hashimoto, Boris Tane

每条指令必须是 agent 可以直接执行的动作，不是建议或背景知识。

**坏例子**: "建议遵循 REST 规范来设计 API"
**好例子**: "所有 API endpoint 必须在 `/api/v1/` 前缀下。创建新 endpoint 时运行 `make lint-api` 检查合规性。"

**评分标准**:
- 3分: 90%+ 的指令都是可执行的（有具体命令、路径、格式）
- 2分: 60-90% 可执行
- 1分: 30-60% 可执行，大量 "建议"、"考虑" 类语言
- 0分: 主要是描述性/解释性内容

---

## 原则 5：新鲜度信号（Freshness）

**来源**: OpenAI Codex 团队

过时的文档比没有文档更危险——agent 会按过时规则行动，产生难以排查的 bug。

**机制**:
- 文件顶部标注 `last_verified` 日期
- 版本号或关联的 commit hash
- 后台 agent 定期扫描过时文档并开清理 PR
- 规则关联具体的代码路径（路径消失 = 规则可能过时）

**评分标准**:
- 3分: 有日期/版本标记，且有机制检测过时内容
- 2分: 有日期标记但没有自动检测
- 1分: 部分文件有日期
- 0分: 无任何新鲜度信号

---

## 原则 6：反馈循环（Feedback Loop）

**来源**: Greg Brockman, Mitchell Hashimoto

文档是反馈循环，不是静态产物。每次 agent 犯错，就应该更新文档来防止复现。

关键问题：项目中是否有证据表明文档是在 agent 实际使用过程中迭代出来的？

**评分标准**:
- 3分: 有明确的错误→文档更新流程，历史记录可见
- 2分: 文档偶尔因 agent 错误更新，但没有系统化流程
- 1分: 文档偶尔更新但不是因为 agent 反馈
- 0分: 文档写完就没动过

---

## 原则 7：架构约束的文档化

**来源**: OpenAI Codex 团队

架构约束（依赖方向、模块边界、禁止的导入关系）必须被显式文档化，并通过机械手段（linter、结构测试）执行。

文档中的架构规则应该：
- 声明依赖方向（如 Types → Config → Repo → Service → Runtime → UI）
- 列出禁止的依赖关系
- 关联到具体的 linter 规则或测试
- 违规时的错误消息本身就是修复指南

**评分标准**:
- 3分: 架构约束有文档 + 机械执行 + 错误消息含修复方法
- 2分: 架构约束有文档但无机械执行
- 1分: 架构隐含在代码结构中，无显式文档
- 0分: 无架构约束文档

---

## 原则 8：可观测性（Observability）

**来源**: OpenAI Codex 团队, Harrison Chase (LangChain)

Harrison Chase 的核心洞察：**"在传统软件中，代码记录了应用做什么；在 AI 中，traces 记录了应用做什么。"** Agent 的实际决策发生在模型运行时内部，代码只是脚手架。如果 agent 无法查询 logs、metrics、traces，它就是在盲飞——猜测失败点，基于不完整信息提改动。

OpenAI 给 Codex agent 的每个 worktree 配备了独立的可观测性管道（logs via LogQL, metrics via PromQL, distributed traces），agent 可以通过 query DSL 查询。这让 "确保服务启动在 800ms 内" 这类指令变得可执行——agent 用运行时证据验证自身工作。

**关键转变**: 可观测性接口的主要消费者从人类变成了 agent。Dashboard 和图表不够——agent 需要通过 API/CLI 返回的结构化数据。机器可读性、结构化输出、可组合性是一等公民。

**评分标准**:
- 3分: 文档中记录了 agent 可用的 log/metrics/trace 查询命令，agent 能用运行时数据自我验证
- 2分: 有部分可观测性文档（如日志路径），但 agent 无法直接查询
- 1分: 有人类用的 dashboard/监控，但未为 agent 提供机器可读接口
- 0分: 无可观测性文档，agent 完全无法获取运行时反馈
