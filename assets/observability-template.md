# docs/observability.md Standard Template

Provides agents with runtime self-verification capabilities. Core principle from Harrison Chase: **"In traditional software, code documents what the app does; in AI, traces document what the app does."** If an agent cannot query logs/metrics/traces, it is flying blind.

---

````markdown
<!-- last_verified: YYYY-MM-DD -->
<!-- related_paths: [log config path, monitoring config path] -->

# Observability

Agents can query runtime data using the following methods to verify their own changes.

## Logs

```bash
# View application logs (last 100 lines)
[exact log command, e.g., "docker compose logs --tail 100 app"]

# Search for specific errors
[exact grep/query command, e.g., "docker compose logs app 2>&1 | grep ERROR"]

# View logs for a specific time range
[exact command with time filter]
```

Log format: [JSON / plaintext / structured]
Log location: `[path or service]`

## Metrics

```bash
# Check service health
[exact command, e.g., "curl -s http://localhost:3000/health | jq ."]

# Check response time
[exact command, e.g., "curl -w '%{time_total}' -s http://localhost:3000/api/ping"]

# Check database connection pool status
[exact command if applicable]
```

Key metrics and thresholds:

| Metric | Normal range | Alert threshold | Query command |
|---|---|---|---|
| Service startup time | < 3s | > 5s | `[command]` |
| API response time (p95) | < 200ms | > 500ms | `[command]` |
| Database query time | < 50ms | > 200ms | `[command]` |
| Memory usage | < 512MB | > 1GB | `[command]` |

## Traces (if applicable)

```bash
# Query recent traces
[exact command, e.g., query DSL, Jaeger API, etc.]

# View trace for a specific request
[command with trace ID placeholder]

# Find slow requests
[command to find traces exceeding threshold]
```

Trace query interface: [API endpoint / CLI tool / query DSL]
Trace storage: [Jaeger / Zipkin / OpenTelemetry Collector / etc.]

## Self-verification checklist

After completing the following task types, agents should verify using the commands above:

| Task type | Verification method |
|---|---|
| Modify API endpoint | Check logs for no errors + check metrics for normal response time |
| Modify database-related code | Check logs for normal queries + check metrics for no connection pool overflow |
| Modify performance-related code | Check metrics to confirm key indicators have not regressed |
| Modify startup flow | Check logs for no startup anomalies + check metrics for startup time within threshold |
| Deploy changes | Check health endpoint + check logs for no errors in first 30 seconds |

## Fallback when no observability infrastructure exists

If the project does not yet have a formal log/metrics/trace system, agents can still self-verify using these methods:

```bash
# Run tests and check output
[test command] 2>&1 | tail -20

# Start service and check if successful
[run command] & sleep 3 && curl -s http://localhost:[port]/health

# Check build artifact size (prevent unexpected bloat)
du -sh [build output dir]

# Check TypeScript types (zero-cost verification)
npx tsc --noEmit 2>&1 | tail -5
```
````

---

## Usage notes

1. Replace all `[placeholder]` entries with actual project commands
2. If the project lacks a specific layer (e.g., no traces), delete that section and note it at the top
3. Thresholds in the key metrics table should align with the team's SLA/SLO
4. The "Fallback when no observability infrastructure exists" section is for small projects or local dev environments, ensuring basic verification even without formal monitoring
5. The self-verification checklist is an operational guide for agents — run the corresponding checks after completing each task type
