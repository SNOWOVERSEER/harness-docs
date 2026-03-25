# docs/observability.md 标准模板

为 agent 提供运行时自我验证能力。核心原则来自 Harrison Chase：**"在传统软件中，代码记录了应用做什么；在 AI 中，traces 记录了应用做什么。"** Agent 如果无法查询 logs/metrics/traces，就是在盲飞。

---

```markdown
<!-- last_verified: YYYY-MM-DD -->
<!-- related_paths: [日志配置路径, 监控配置路径] -->

# Observability

Agent 可以通过以下方式查询运行时数据来验证自己的改动。

## Logs

```bash
# 查看应用日志（最近 100 行）
[exact log command, e.g., "docker compose logs --tail 100 app"]

# 搜索特定错误
[exact grep/query command, e.g., "docker compose logs app 2>&1 | grep ERROR"]

# 查看特定时间段的日志
[exact command with time filter]
```

日志格式: [JSON / plaintext / structured]
日志位置: `[path or service]`

## Metrics

```bash
# 检查服务健康状态
[exact command, e.g., "curl -s http://localhost:3000/health | jq ."]

# 查看响应时间
[exact command, e.g., "curl -w '%{time_total}' -s http://localhost:3000/api/ping"]

# 查看数据库连接池状态
[exact command if applicable]
```

关键指标及阈值:

| 指标 | 正常范围 | 告警阈值 | 查询命令 |
|---|---|---|---|
| 服务启动时间 | < 3s | > 5s | `[command]` |
| API 响应时间 (p95) | < 200ms | > 500ms | `[command]` |
| 数据库查询时间 | < 50ms | > 200ms | `[command]` |
| 内存使用 | < 512MB | > 1GB | `[command]` |

## Traces (if applicable)

```bash
# 查询最近的 traces
[exact command, e.g., query DSL, Jaeger API, etc.]

# 查看特定请求的 trace
[command with trace ID placeholder]

# 查看慢请求
[command to find traces exceeding threshold]
```

Trace 查询接口: [API endpoint / CLI tool / query DSL]
Trace 存储: [Jaeger / Zipkin / OpenTelemetry Collector / etc.]

## 自我验证清单

Agent 在完成以下类型的任务后，应使用上述命令验证：

| 任务类型 | 验证方法 |
|---|---|
| 修改 API endpoint | 查 logs 确认无 error + 查 metrics 确认响应时间正常 |
| 修改数据库相关代码 | 查 logs 确认查询正常 + 查 metrics 确认连接池无溢出 |
| 修改性能相关代码 | 查 metrics 确认关键指标未退化 |
| 修改启动流程 | 查 logs 确认启动无异常 + 查 metrics 确认启动时间在阈值内 |
| 部署变更 | 查 health endpoint + 查 logs 前 30 秒无 error |

## 无可观测性基础设施时的替代方案

如果项目还没有正式的 log/metrics/trace 系统，agent 仍然可以用以下方式自我验证：

```bash
# 运行测试并检查输出
[test command] 2>&1 | tail -20

# 启动服务并检查是否成功
[run command] & sleep 3 && curl -s http://localhost:[port]/health

# 检查构建产物大小（防止意外膨胀）
du -sh [build output dir]

# 检查 TypeScript 类型（零成本验证）
npx tsc --noEmit 2>&1 | tail -5
```
```

---

## 使用说明

1. 用项目实际的命令替换所有 `[placeholder]`
2. 如果项目没有某个层次（如没有 traces），删除该部分并在顶部注明
3. 关键指标表格中的阈值应与团队的 SLA/SLO 对齐
4. "无可观测性基础设施时的替代方案" 部分适用于小型项目或本地开发环境，确保即使没有正式监控也能做基本验证
5. 自我验证清单是给 agent 的操作指南——每种任务类型完成后应跑对应的检查
