# Prometheus Certified Associate (PCA) — Study Plan

> Exam taken **after** shared foundations with OTelCA are complete (Weeks 1–5 of dual plan).
> PCA-specific deep dive: **Weeks 6–7**, then integration review in **Week 10**.

---

## Domains

### Observability Concepts
*Covered in Week 1–2 (shared with OTelCA)*

- **Metrics**: what they are, types (counter, gauge, histogram, summary), when to use each
- **Logs and events**: difference between structured logs, unstructured logs, and discrete events; how they complement metrics
- **Tracing and spans**: basics of distributed tracing, how traces differ from metrics — and why Prometheus does NOT do tracing natively
- **Push vs Pull**: Prometheus is pull-based (scrapes targets); understand trade-offs vs push (e.g., Pushgateway use case)
- **Service discovery**: how Prometheus discovers targets dynamically — Kubernetes SD, file SD, DNS SD, Consul SD
- **SLOs, SLAs, SLIs**:
  - SLI = measurement (e.g., % of requests < 200ms)
  - SLO = target (e.g., 99.9% of requests < 200ms)
  - SLA = contractual agreement built on SLOs
  - Know how to express SLIs as PromQL queries and SLOs as alerting thresholds

---

### Prometheus Fundamentals
*Covered in Week 3 (data model) + Week 5 (collection infrastructure)*

- **System architecture**:
  - Components: Prometheus server, exporters, Pushgateway, Alertmanager, Grafana
  - Prometheus scrapes `/metrics` endpoints at configurable intervals
  - TSDB stores data locally; no native clustering or long-term storage
- **Config and scraping** (`prometheus.yml`):
  - `global`: `scrape_interval`, `evaluation_interval`
  - `scrape_configs`: `job_name`, `static_configs`, `relabel_configs`
  - `rule_files`: path to alerting/recording rule files
  - `alerting`: Alertmanager targets
- **Prometheus limitations** — know these well for exam:
  - No native long-term storage (use Thanos, Cortex, VictoriaMetrics)
  - No native distributed/HA setup
  - No built-in tracing
  - High cardinality labels can cause TSDB performance issues
  - Pull model requires targets to be reachable (Pushgateway bridges short-lived jobs)
- **Data model and labels**:
  - Every time series identified by metric name + label set: `http_requests_total{job="api", status="200"}`
  - Labels are key-value pairs; avoid high-cardinality labels (e.g., user IDs, full URLs)
  - `__` prefix labels are internal/reserved
- **Exposition format**:
  - Plain text format: `# HELP`, `# TYPE`, then samples
  - OpenMetrics format (superset, used by OTel ecosystem)
  - Content-type negotiation during scrape

---

### PromQL
*Covered in Week 6–7 (PCA-specific deep dive)*

- **Selecting data**:
  - Instant vector: `http_requests_total{status="200"}`
  - Range vector: `http_requests_total[5m]`
  - Label matchers: `=`, `!=`, `=~`, `!~`
  - `offset` modifier: `metric[5m] offset 1h`
- **Rates and derivatives**:
  - `rate(counter[5m])`: per-second average rate over range — use for counters
  - `irate(counter[5m])`: instantaneous rate (last 2 samples) — more responsive, noisier
  - `increase(counter[5m])`: total increase over range (= rate × range duration)
  - `deriv(gauge[5m])`: per-second derivative of a gauge (linear regression)
  - Rule: **always use `rate()`/`increase()` on counters, never on gauges**
- **Aggregating over time**:
  - `_over_time` functions: `avg_over_time`, `max_over_time`, `min_over_time`, `sum_over_time`, `count_over_time`
  - Applied to range vectors of a single series
- **Aggregating over dimensions**:
  - Aggregation operators: `sum`, `min`, `max`, `avg`, `count`, `stddev`, `topk`, `bottomk`
  - `by (label)` — keep only specified labels
  - `without (label)` — drop specified labels, keep rest
  - Example: `sum(rate(http_requests_total[5m])) by (job, status)`
- **Binary operators**:
  - Arithmetic: `+`, `-`, `*`, `/`, `%`, `^`
  - Comparison: `==`, `!=`, `>`, `<`, `>=`, `<=`
  - Logical/set: `and`, `or`, `unless`
  - Vector matching: `on()`, `ignoring()`, `group_left()`, `group_right()` for many-to-one joins
- **Histograms**:
  - Metric types: `_bucket`, `_count`, `_sum`
  - `histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))` — p99 latency
  - Native histograms (Prometheus 2.40+): higher precision, no pre-defined buckets
  - Know bucket boundaries and why bucket choice matters for accuracy
- **Timestamp metrics**:
  - `time() - my_last_event_timestamp` — staleness/freshness checks
  - Use with alerting to detect stale data or missed jobs

---

### Instrumentation and Exporters
*Covered in Week 4 (shared with OTelCA)*

- **Client libraries**: Go (`prometheus/client_golang`), Python (`prometheus_client`), Java (`simpleclient`) — know how to register and expose metrics
- **Instrumentation**:
  - Counter: `Inc()`, `Add()` — only goes up, reset on restart
  - Gauge: `Set()`, `Inc()`, `Dec()` — can go up/down
  - Histogram: `Observe(value)` — measures distributions, pre-defined buckets
  - Summary: `Observe(value)` — calculates quantiles client-side (avoid in favor of histograms for aggregation)
- **Exporters** — know the most common:
  - `node_exporter`: OS-level metrics (CPU, memory, disk, network)
  - `blackbox_exporter`: probing endpoints (HTTP, TCP, ICMP, DNS)
  - `kube-state-metrics`: Kubernetes object state (not resource usage)
  - `cAdvisor`: container resource usage
  - `mysqld_exporter`, `redis_exporter`: database exporters
  - Textfile collector: allows node_exporter to expose custom metrics from files
- **Structuring and naming metrics**:
  - Format: `<namespace>_<subsystem>_<name>_<unit>`
  - Unit suffixes: `_seconds`, `_bytes`, `_total` (counters), `_ratio`
  - Avoid: abbreviations, verbs in names, inconsistent units
  - Base units only: seconds (not milliseconds), bytes (not kilobytes)

---

### Alerting and Dashboarding
*Covered in Week 6–7 (PCA-specific deep dive)*

- **Dashboarding basics**:
  - Grafana as the standard frontend for Prometheus
  - Data sources, panels, variables, templating
  - Common panel types: time series, stat, gauge, table, heatmap
  - Use recording rules to pre-compute expensive queries for dashboards
- **Configuring alerting rules**:
  ```yaml
  groups:
    - name: example
      rules:
        - alert: HighErrorRate
          expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate on {{ $labels.job }}"
            description: "Error rate is {{ $value | humanizePercentage }}"
  ```
  - `for`: alert must be in pending state for this duration before firing — avoids flapping
  - `labels`: used by Alertmanager for routing
  - `annotations`: human-readable context, support Go templating with `$labels` and `$value`
  - **Recording rules**: pre-compute expensive expressions, stored as new time series
    ```yaml
    - record: job:http_requests_total:rate5m
      expr: sum(rate(http_requests_total[5m])) by (job)
    ```
- **AlertManager** — understand the full pipeline:
  - **Routing**: tree of routes, matched by label matchers (`matchers`, `match`, `match_re`)
  - **Receivers**: where alerts go — email, Slack, PagerDuty, webhook, OpsGenie
  - **Grouping**: combines alerts with same labels into one notification (`group_by`, `group_wait`, `group_interval`)
  - **Inhibition**: suppress lower-priority alerts when higher-priority one is firing (`inhibit_rules`)
  - **Silences**: temporary mute based on label matchers, set via UI or `amtool`
  - **Repeat interval**: how often to re-notify for ongoing alert
  - HA setup: AlertManager gossip protocol for deduplication across replicas
- **Alerting basics**:
  - **When**: use `for` to avoid transient spikes; alert on symptoms not causes where possible
  - **What**: metric name + labels uniquely identify the problem source
  - **Why**: annotations provide runbook links and context for on-call engineers
  - Avoid alert fatigue: right severity levels, proper grouping, meaningful thresholds

---

## Study Schedule (PCA focus within dual plan)

| Week | Topics | Notes |
|------|--------|-------|
| 1–2 | Observability concepts, SLOs/SLIs/SLAs, push vs pull | Shared with OTelCA |
| 3 | Prometheus data model, labels, exposition format | Shared with OTelCA |
| 4 | Client libraries, instrumentation, exporters | Shared with OTelCA |
| 5 | Prometheus config, scraping, service discovery, limitations | Shared with OTelCA |
| 6 | PromQL: selectors, rate/irate/increase, aggregation | PCA only |
| 7 | PromQL: histograms, binary ops; AlertManager; alerting rules | PCA only |
| 10 | Integration patterns (OTel → Prometheus), mock exams | Both |

---

## Key Resources

- [Prometheus docs](https://prometheus.io/docs/) — official reference for config, PromQL, alerting
- [PromQL cheat sheet](https://promlabs.com/promql-cheat-sheet/) — promlabs.com
- [Robust Perception blog](https://www.robustperception.io/blog) — deep dives by Prometheus maintainers
- [Awesome Prometheus](https://github.com/roaldnefs/awesome-prometheus) — curated exporter and tooling list
- Practice environment: run Prometheus + node_exporter + Alertmanager locally via Docker Compose

