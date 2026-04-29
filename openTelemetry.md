# OpenTelemetry Certified Associate (OTelCA) — Study Plan

> Studied **in parallel** with PCA. Shared foundations (Weeks 1–5) overlap heavily with Prometheus exam.
> OTelCA-specific deep dive: **Weeks 8–9**, then integration review in **Week 10**.

---

## Curriculum

### Fundamentals of Observability
*Covered in Week 1–2 (shared with PCA)*

- **Telemetry data** — the three pillars and how OTel unifies them:
  - **Metrics**: numeric measurements over time — same concept as Prometheus, but OTel supports push AND pull
  - **Traces**: end-to-end request flow across services, composed of spans — OTel's primary differentiator vs Prometheus
  - **Logs**: structured/unstructured event records; OTel adds trace context to correlate logs with traces
  - OTel's goal: one unified API/SDK to produce all three signals, vendor-agnostic
- **Semantic conventions** — study alongside Prometheus naming rules (Week 3):
  - Standardized attribute names across languages and frameworks: `http.method`, `db.system`, `service.name`
  - Stability levels: stable, experimental, deprecated
  - Resource attributes: describe the entity producing telemetry (e.g., `service.name`, `k8s.pod.name`)
  - Why they matter: enables consistent querying and dashboarding across different services and backends
  - **Prometheus link**: OTel semantic conventions are the formalized version of Prometheus naming best practices — learn them together
- **Instrumentation** — studied alongside Prometheus client libraries (Week 4):
  - **Automatic instrumentation**: agents/plugins that instrument frameworks without code changes (HTTP, DB, messaging)
  - **Manual instrumentation**: explicit API calls to create spans, record metrics, emit logs
  - **Zero-code instrumentation**: language-specific agents (Java agent, Python auto-instrumentation) — no code modification needed
  - Trade-off: auto-instrumentation is fast to adopt; manual gives more control and custom attributes
- **Analysis and outcomes**:
  - SLOs/SLIs expressed via OTel metrics — same concepts as PCA, different tooling
  - Traces enable root cause analysis for latency problems that metrics alone can't explain
  - Correlating signals: use `trace_id` in logs and exemplars in metrics to link all three signals
  - Backends: Jaeger/Zipkin (traces), Prometheus (metrics), Loki/Elasticsearch (logs), Grafana (unified UI)

---

### OpenTelemetry API and SDK
*Covered in Week 8–9 (OTelCA-specific deep dive)*

- **Data model**:
  - **Traces**: `Trace` → `Span` → `SpanContext` (`TraceID`, `SpanID`, `TraceFlags`, `TraceState`)
  - **Metrics**: `MeterProvider` → `Meter` → instruments (Counter, Gauge, Histogram, UpDownCounter, ObservableCounter...)
  - **Logs**: `LoggerProvider` → `Logger` → `LogRecord` (with optional `TraceID`/`SpanID` fields for correlation)
  - Resource: immutable description of the entity (attached to all telemetry from that SDK instance)
  - InstrumentationScope: identifies the library/component producing the telemetry
- **Composability and extension**:
  - Providers (`TracerProvider`, `MeterProvider`, `LoggerProvider`) are the entry points — configurable and replaceable
  - Exporters are pluggable: swap OTLP for Jaeger, Zipkin, Prometheus, console without changing instrumentation code
  - Propagators are pluggable: swap W3C TraceContext for B3, Jaeger headers
  - SDK is separate from API — libraries depend only on the API; apps wire up the SDK
- **Config**:
  - SDK configured via code (explicit) or environment variables (zero-code)
  - Key env vars: `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_TRACES_SAMPLER`, `OTEL_RESOURCE_ATTRIBUTES`
  - `OTEL_SDK_DISABLED=true` to disable without code change
  - File-based config (YAML) introduced in OTel spec — declarative alternative to env vars
- **Signals** — know all three well:
  - **Tracing**:
    - Span kinds: `CLIENT`, `SERVER`, `PRODUCER`, `CONSUMER`, `INTERNAL`
    - Span status: `UNSET`, `OK`, `ERROR`
    - Span events: timestamped annotations within a span (e.g., "cache miss")
    - Span links: connect spans across independent traces (e.g., async batch processing)
    - Sampling: `AlwaysOn`, `AlwaysOff`, `TraceIDRatioBased`, `ParentBased` — affects cost vs completeness
  - **Metrics**:
    - Synchronous instruments: `Counter`, `UpDownCounter`, `Histogram`, `Gauge` — called inline with operations
    - Asynchronous (observable) instruments: `ObservableCounter`, `ObservableGauge`, `ObservableUpDownCounter` — callbacks polled at collection time
    - **Prometheus link**: Counter → Prometheus counter; Histogram → Prometheus histogram; Gauge/UpDownCounter → Prometheus gauge
    - Views: customize metric aggregation, rename, filter attributes (reduce cardinality)
    - Exemplars: attach `TraceID` to histogram samples — bridges metrics and traces
  - **Logs**:
    - OTel Logs Bridge API: connects existing logging frameworks (log4j, logrus, zap) to OTel — not a new logging API
    - `SeverityNumber` + `SeverityText`: standardized severity levels
    - Correlation fields: `TraceID`, `SpanID`, `TraceFlags` added automatically when emitted inside a span
- **SDK pipelines**:
  - **Traces**: `SpanProcessor` (BatchSpanProcessor, SimpleSpanProcessor) → `SpanExporter` (OTLP, Jaeger, Zipkin, console)
  - **Metrics**: `MetricReader` (PeriodicExportingMetricReader, PrometheusReader) → `MetricExporter`
  - **Logs**: `LogRecordProcessor` → `LogRecordExporter`
  - Multiple exporters can run in parallel via `SpanProcessor` chaining
  - `BatchSpanProcessor`: buffers and sends in batches — production default; tune `maxQueueSize`, `scheduledDelayMillis`
- **Context propagation**:
  - `Context` object carries `Span`, `Baggage`, and other cross-cutting values — immutable, thread-safe
  - **Propagators** inject/extract context into carriers (HTTP headers, message metadata)
  - W3C `traceparent` header: `version-traceId-parentSpanId-flags` — the standard; know the format
  - W3C `tracestate`: vendor-specific metadata alongside traceparent
  - `Baggage`: key-value pairs propagated along with trace context — use for passing metadata (not sensitive data)
  - **Prometheus link**: no equivalent in Prometheus — this is pure OTel territory
- **Agents**:
  - Language-specific auto-instrumentation agents inject bytecode/monkey-patching at runtime
  - Java: `-javaagent:opentelemetry-javaagent.jar` — most mature, instruments 100+ libraries automatically
  - Python: `opentelemetry-instrument` wrapper command
  - Node.js: `--require @opentelemetry/auto-instrumentations-node/register`
  - Configurable via env vars — same `OTEL_*` variables as manual SDK

---

### OpenTelemetry Collector
*Covered in Week 5 (shared infrastructure concepts) + Week 8–9 (OTelCA deep dive)*

- **Config** (`config.yaml`):
  ```yaml
  receivers:
    otlp:
      protocols: { grpc: {}, http: {} }
    prometheus:          # scrapes Prometheus /metrics endpoints
      config:
        scrape_configs:
          - job_name: my-app
            static_configs:
              - targets: [localhost:8080]
  processors:
    batch: {}
    resource:
      attributes:
        - key: env
          value: production
          action: insert
  exporters:
    otlp:
      endpoint: jaeger:4317
    prometheusremotewrite:
      endpoint: http://prometheus:9090/api/v1/write
    debug:
      verbosity: detailed
  service:
    pipelines:
      traces:
        receivers: [otlp]
        processors: [batch]
        exporters: [otlp]
      metrics:
        receivers: [otlp, prometheus]
        processors: [batch, resource]
        exporters: [prometheusremotewrite]
  ```
  - **Prometheus link**: Collector's `prometheus` receiver = Prometheus scraping; `prometheusremotewrite` exporter sends to Prometheus TSDB
- **Deployment patterns** — know all three:
  - **Agent (sidecar/daemonset)**: runs alongside each service, low latency, local buffering; collects from one host
  - **Gateway**: centralized collectors, shared infrastructure, fan-in from many agents — apply auth, filtering, routing here
  - **No Collector**: SDK exports directly to backend — simpler but couples app to backend
  - Common pattern: Agent → Gateway → multiple backends (Jaeger + Prometheus + Loki)
- **Scaling**:
  - Horizontal scaling of gateway collectors behind a load balancer
  - `loadbalancing` exporter: routes spans from the same trace to the same collector instance (needed for tail sampling)
  - Tail sampling requires all spans of a trace on one collector instance — use `loadbalancingexporter` upstream
  - Head sampling (in SDK) vs tail sampling (in Collector): tail sampling can make policy decisions on complete traces
- **Pipelines**:
  - Each pipeline is `receivers → processors → exporters` for one signal type (traces, metrics, logs)
  - Components can be shared across pipelines (same receiver feeding multiple pipelines)
  - Pipeline isolation: failure in one pipeline does not affect others
  - `fanout`: one receiver → multiple exporters automatically
- **Transforming data**:
  - `filter` processor: drop spans/metrics/logs based on conditions (reduce noise/cost)
  - `transform` processor: modify attributes using OTTL (OTel Transformation Language) — rename fields, set values, delete attributes
  - `attributes` processor: add/update/delete/hash resource or span attributes
  - `resource` processor: modify resource attributes
  - `metricstransform` processor: rename metrics, aggregate, scale values
  - `redaction` processor: mask sensitive data (PII) before exporting
  - **Prometheus link**: similar to Prometheus relabeling (`relabel_configs`) but more powerful and applies to all signals

---

### Maintaining and Debugging Observability Pipelines
*Covered in Week 9 (OTelCA-specific)*

- **Context propagation** (debugging aspect):
  - Broken propagation = traces that look like independent single-span traces instead of connected flows
  - Common causes: missing middleware, async boundaries (message queues), non-standard headers
  - Verify with `debug` exporter: check `traceparent` header is set on outbound requests
  - `Baggage` misuse: don't propagate sensitive data; it's visible to all downstream services
- **Debugging pipelines**:
  - `debug` exporter: prints telemetry to stdout — use `verbosity: detailed` to see all attributes
  - `zpages` extension: HTTP endpoint exposing live pipeline stats — `tracez`, `pipelinez`, `servicez`
    ```yaml
    extensions:
      zpages:
        endpoint: 0.0.0.0:55679
    ```
  - `health_check` extension: liveness/readiness endpoints for the Collector (`/`)
  - `pprof` extension: Go profiling endpoint for Collector performance debugging
  - Check pipeline metrics: `otelcol_receiver_accepted_spans`, `otelcol_exporter_send_failed_spans`
  - **Prometheus link**: Collector exposes its own Prometheus metrics — can scrape to monitor the Collector itself
- **Error handling**:
  - Exporters retry on transient failures — configure `retry_on_failure` (max elapsed time, backoff)
  - Queue before export — configure `sending_queue` (queue size, num consumers)
  - Persistent queue: survives Collector restarts (disk-backed)
  - Back-pressure: if queue full, Collector drops data — monitor `otelcol_exporter_queue_size`
  - Receiver errors: malformed OTLP data returns gRPC status codes to SDK — SDK handles retry
- **Schema management**:
  - Semantic conventions evolve — attributes get renamed between versions (e.g., `http.url` → `url.full`)
  - Schema URLs: telemetry carries a schema URL identifying the convention version used
  - `schemaprocessor` in Collector: translates between schema versions to ensure backend compatibility
  - Stability guarantees: stable conventions won't break; experimental ones may change
  - Strategy: pin to a schema version in production; test convention upgrades before rolling out

---

## Study Schedule (OTelCA focus within dual plan)

| Week | Topics | Notes |
|------|--------|-------|
| 1–2 | Observability pillars, telemetry data, SLOs/SLIs/SLAs | Shared with PCA |
| 3 | OTel data model, semantic conventions, exposition format | Shared with PCA |
| 4 | OTel SDK instrumentation, auto vs manual, client libs | Shared with PCA |
| 5 | OTel Collector deployment, config, Prometheus receiver | Shared with PCA |
| 8 | OTel signals deep dive: tracing, context propagation, agents | OTelCA only |
| 9 | SDK pipelines, Collector transformations, debugging, schema | OTelCA only |
| 10 | OTel → Prometheus integration patterns, mock exams | Both |

---

## Key Connections to Prometheus (exam cross-reference)

| OTel Concept | Prometheus Equivalent |
|---|---|
| `Counter` instrument | Prometheus counter (`_total`) |
| `Histogram` instrument | Prometheus histogram (`_bucket`, `_sum`, `_count`) |
| `Gauge` / `UpDownCounter` | Prometheus gauge |
| Semantic conventions | Prometheus naming conventions |
| Collector `prometheus` receiver | Prometheus scraping |
| Collector `prometheusremotewrite` exporter | Prometheus remote write |
| Exemplars in histograms | Prometheus exemplars (bridges traces ↔ metrics) |
| Resource attributes | Prometheus target labels |
| OTTL `filter` processor | Prometheus `relabel_configs` drop action |
| Agent deployment | Prometheus node_exporter on each host |

---

## Key Resources

- [OTel specification](https://opentelemetry.io/docs/specs/otel/) — authoritative source for data model and API contracts
- [OTel Collector docs](https://opentelemetry.io/docs/collector/) — config reference, processors, exporters
- [OTel demo app](https://github.com/open-telemetry/opentelemetry-demo) — full microservices example with all three signals wired up
- [Semantic conventions registry](https://opentelemetry.io/docs/specs/semconv/) — full attribute reference
- Practice environment: run OTel Collector + Jaeger + Prometheus via Docker Compose using the OTel demo 
OTel demo app:
https://github.com/open-telemetry/opentelemetry-demo
