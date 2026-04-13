# Prometheus Certified Associate (PCA) — Study Plan

> Exam taken **after** shared foundations with OTelCA are complete (Weeks 1–5 of dual plan).
> PCA-specific deep dive: **Weeks 6–7**, then integration review in **Week 10**.


## learning

### fundamentals of monitoring 

### prometheus architecture 


- client libraries 
  - official
  - go, python, java/jvm,ruby, rust
  - third parties: c#/.net, node.js, haskell, erlang

- exporters
  - take request from prometheus
  - gathers the required data from the app
  - transform it into correct format
  - return response to prometheus
  - using custom collector or ConstMetrics


- service discovery 
  - to give prometheus know what to monitor and notice if soemthing it is meant to be monitoring is not responding
  - integration with many common svc discovery mechanisms: k8s, EC2, consul
  - prometheus allows you to config how metadata from svc discovery to monitoring targets and their labels using relabeling 

- scraping
  - svc discovery and relabeling give us a list of target to be monitored
  - HTTP request called a scrape
  - prometheus is pull-based system, when and what to scrape, based on its config

- storage
  - stores data locally in a custom db

- dashboards
  - has a number of HTTP API allows you to both request raw data and evaluate PromQL queries
  - provides the expression browser
  - use grafana for dashboard

- recording rules and alert
  - recording rules allow PromQL expression to be evaluated on regular basis
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



### learning 

#### install 

tar -xzf prometheus-*.linux-amd64.tar.gz

tar -xzf prometheus-3.11.0.linux-amd64.tar.gz

curl -L -O https://github.com/prometheus/prometheus/releases/download/v3.11.0/prometheus-3.11.0.linux-amd64.tar.gz
cd prometheus-*.linux-amd64/

curl -L -O https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz

tar -xzf node_exporter-*.linux-amd64.tar.gz

curl -L -O https://github.com/prometheus/alertmanager/releases/download/v0.31.1/alertmanager-0.31.1.linux-amd64.tar.gz

tar -xzf alertmanager-*.linux-amd64.tar.gz

#### app monitoring 

pip install prometheus_client

#### exposition 

exposition: process of making metrcis available in prometheus 
- support 2 human-readable text format:
  - prometheus text format 
  - openmetrics 

- common client libraries 
  - python : prometheus_client 
  - WSGI: 
  - twisted: python event driven network engine 
  - multiprocess with gunicorn
  - Go: http.Handler
  - java: simpleclient, HTTPServer, Servlet, 

- Pushgateway: metric cache for service-level batch jobs
  - due to batch job run on regular schedule, not continous running, prometheus can't exactly scrape them 
    - batch job to push to pushgateway 
    - prometheus regular scrape pushgateway


- Bridges 
  - bridge take metrics output from the client library and output it to something other than prometheus 


- Parsers
  - to feed prometheus metrics into other monitoring system 

- metrics types 

- labels 

- escaping 

- timestamps 

- OpenMetrics 

#### labels 

- instrumentation 

- metric 
  - latency_seconds_sum{path="/bar"}: family
  - latency_seconds{path="/bar"}: child
  - latency_seconds: family

- aggregating 


- label patterns 
  - enum
  - info


- cardinality 

#### dashboard with grafana 

docker run -d --name=grafana --net=host grafana/grafana:9.1.6


### infra monitoring 

#### node exporter 
- CPU collector 

- filesystem collector

- diskstats collector

- netdev collector

- meminfo collector

- hwmon collector 

- stat collector 

- uname collector 

- OS collector 

- loadavg collector 

https://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html

- Pressure collector 

- textfile collector 

#### service discovery 

- static config 

- file config 

- HTTP service discovery 

- consul

- EC2

- relabeling 

- target labels 

- case 


#### container and kubernetes 

- cAdvisor - exporter that provides metrics about cgroups 

- cpu 

- memory 

- how to use endpointslice

- pod 

- ingress

- kube-state-metrics 

- alternative deployments
  - prometheus operator 
  - prometheus helm charts 

#### common exporters

- Consul exporter

- MySQLd

- Grok exporter 

- blackbox 

- ICMP 

- TCP

- DNS 

- prometheus config 

#### working with other monitoring 

#### writing exporter

### promQL

#### basics

- gauge: snapshot of state, can be use with sum, average, minimum, maximum

container_cpu_usage_seconds_total

-  Average memory usage per namespace
avg by (namespace) (container_memory_usage_bytes)

-  Top 5 pods by memory usage
topk(5, container_memory_usage_bytes{namespace="cattle-monitoring-system"})


- counter: number of size of events, value of app expose on their /metrics 


- CPU usage across all pods in namespace
rate(container_cpu_usage_seconds_total{namespace="cattle-monitoring-system"}[5m])

- CPU usage grouped by pod
sum by (pod) (rate(container_cpu_usage_seconds_total{namespace="cattle-monitoring-system"}[5m]))

- summary 

- Total CPU usage across all pods in namespace
sum(rate(container_cpu_usage_seconds_total{namespace="cattle-monitoring-system"}[5m]))

- CPU usage grouped by pod
sum by (pod) (rate(container_cpu_usage_seconds_total{namespace="cattle-monitoring-system"}[5m]))


- historgram : track the distribution of the size of events, calculating quantiles - need to revisit
histogram_quantile(0.90,sum(
        rate(container_cpu_usage_seconds_total{namespace="cattle-monitoring-system"}[5m])
    ))


- selector 
  - 4 matchers
    - =
    - !=
    - =~ : regular expression matcher
    - !~ : negative regular expression matcher

- instant vector 

- range vector 

- subqueries 

- offset 

- at modifier : @ modifier to change the evaluation of vector selector, range selector and subqueries 

- HTTP API 
  - query : /api/v1/query
  - query_range: /api/v1/query_range

#### aggregation operators 

#### binary operators 

- arithmetic operators 
  - add 
  - minus 
  - multiply 
  - division 
  - modulo
  - exponentiation 
- trigonometric operator 

- comparison 
  - ==
  - != 
  - > 
  - <
  - >=
  - <=

- vector matching ???
  - one to one 
  - one to many 
  - many to many 

- or operator 

- unless operator 

- and operator 


#### functions 

- changing type 
  - vector 
  - scalar
- math
  - abs
  - ln, log2, log10
  - exp
  - sqrt
  - ceil and floor 
  - round 
  - clamp, clamp_max, clamp_min
  - sgn
- trigonometric

- time and date
  - time 
  - minute, hour, day_of_week, day_of_month, day_of_year, days_in_month, month, year
  - timestamp

- labels
  - label_replace
  - label_join 

- missing series, absent, absent_over_time
- sorting
  - sort
  - sort_desc

- histogram
  - histogram_quantile

- counter 
  - rate 
  - increase
  - irate
  - resets
- changing gauges 
  - changes 
  - deriv 
  - predict_linear
  - delta 
  - idelta
  - holt_winters
- aggregation over time 


#### recording rules 
- rules as rules.yaml as rules config 

- when to use recording rules 
  - reducing cardinality 
  - composing range vector functions 
  - rules for api 
- when not to use rules 
  - undo benefits of labels 
  - preaggregating every metric an app exposes 


### alerting 

- prometheus -> alertmanager -> email/pagerduty/chat

---
 - name: node_rules
   rules:
    - record: job:up:avg
      expr: avg without(instance)(up{job="node"})
    - alert: ManyInstancesDown
      expr: job:up:avg{job="node"} < 0.5
---

- for 

- annotations and templates 

#### alert manager 
  - inhibition 

  - silencing 

  - routing 

  - grouping 

  - throttling and repetition 

  - notification 

  - routing tree 


  ### items need to revisit 

  - difference between gauge/counter... 

  - distributed tracing 

  2

### mock questions 

credit for https://yuyatinnefeld.com/2023-07-24-pca/


#### Observability Concepts (18%)

- What is the preferred approach used by Prometheus to collect metrics from a target?
  - pull method 
- What is the Observability?
  - monitoring desired metrics ?
- What is RED Method?
  - monitoring 3 metrics: request rate, errors, duration 
What are the distinctions between SLO, SLI, and SLA?
In the context of tracing, what is the meaning or representation of a span?
In which scenarios is distributed tracing less beneficial or NOT as applicable?
What are typically tracked within a span of a trace?
What is the good and bad metric?
Which type of data is monitored by Prometheus?
In the context of monitoring and observability, what type of data is typically used to define a SLI?
What is the meaning or purpose of an error budget policy?
What is one advantage of the push model for recoring metrics compared to pull models?
How do Prometheus, ELK stack, and InfluxDB differ in terms of their functionalities and use cases?
What is the definition of a metric?
What are the Prometheus exemplars?
What is one of the main purposes or goals of logging?
What are the 3 core components of observability?
What is the Monitoring
What is the Telemetry?
What is the challenges of observability?
#### Prometheus Fundamentals (20%)
What is the CLI utility tool for Prometheus called?
What are the limitations of Prometheus?
What is Service Discovery and which categories are there?
Which property configures the timing to scrape metrics from targets?
Which section in the Prometheus configuration file governs the selection of targets to be scraped?
Which action in the label configuration is used to delete a specific target?
How is managed data retention in prometheus?
What are the essential 3 components of Prometheus?
What is required to be able to reload Prometheus?
What are 3 methods to restart the Prometheus server?
What HTTP method does Prometheus employ for performing scrapes?
Which SD configuration is recommended for scraping EC2 instances?
Which SD configuration is recommended for nodes of Elastic Kubernetes Service on AWS?
What is the purpose of the scrape_interval configuration in Prometheus?
Which type of database does Prometheus utilize?
What component is responsible for collecting metrics from an instance and exposing them in a format that Prometheus expects?
Which component is suitable for collecting metrics from batch/cron jobs?
When is the configuration option honor_labels:true used?
What is the purpose of port 9090/9093/9100/9091/9115 in Prometheus?
what are 2 default metric labels?
Which of the file systems is recommended/supported by Prometheus?
How can you configure a Blackbox Exporter probe to check the successful response of your servers to PING?
How do you configure the targets that Prometheus should scrape?
What is the agent deployment mode of Prometheus?
Which CLI command is suitable for unit testing Prometheus rules?
Which CLI command is suitable for checking validity of the config files?
How do you define the targets with SD that Prometheus should collect metrics from?
How can you delete the specific time series metrics of Prometheus?
How can you delete the all time series metrics of Prometheus?
Which format does file-based SD provide?
#### PromQL (28%)
What is PromQL?
What is histogram metric in Prometheus?
Which 4 data types are used in PromQL?
What is the name of the vector in Prometheus that stores a single sample value?
Which PromQL function is used to estimate the value of a time series at a future time, t seconds from the current time, based on the range vector v?
Between what type of expressions can logical operators be defined?
Which function can be used to calculate the average of a range vector in Prometheus?
What is the diff between avg(...) and avg_over_time(...)?
With which type of metrics is the rate(...) function primarily used in Prometheus?
What does the term “offset” refer to in Prometheus?
What distinguishes the rate(...) and irate(...) query functions in Prometheus?
What distinguishes the rate(...) and deriv(...) query functions in Prometheus?
Which type of metric is suitable for measuring the internal temperature of a server?
What is the data type of Prometheus metric values?
How many unique series are generated by a histogram metric type?
What are the 4 components of the Prometheus metrics data model?
What is the difference between the ceil and floor functions?
Which query function among the following returns a result of 1 in case the specified time series does not exist?
What is the logical/arithmetic/comparison binary operator?
What is the vector matching?
What is the group modifiers?
Which function is NOT using counter metrics? irate(), increase(), reset(), idelta(), avg(), rate()
How to calc the time in days until the LAST certificate expiration?
What is the dimensional aggregation?
What is the significance of the double underscore “__” before a label name?
Instrumentation and Exporters (16%)
What is the HTTP headers to establish by Prometheus during each scrape?
Which 2 query parameters are required when configuring a Blackbox Exporter probe?
What is the exposition format of Prometheus?
Does Prometheus need to perform any format conversion on the metrics returned by a monitored Linux machine?
What is the default endpoint that Prometheus uses to scrape the metrics from the target?
Where is the version of the Prometheus exporter typically defined?
What is the most suitable exporter for monitoring an HTTP web server endpoint to verify that it returns a 200 status code?
Which Prometheus exporter is recommended for monitoring network devices?
Which networking protocol does Prometheus utilize for performing scrapes?
What is the purpose of a Prometheus metrics registry?
What is the purpose or definition of a Prometheus exporter?
In what scenarios would you use the Blackbox Exporter?
How does Prometheus identify the scrape path for its targets?
Which endpoints allows blackbox probing?
In a scenario where you have a dynamic etcd database containing scrape targets for Prometheus, how should you configure service discovery?
What are the 2 types of attributes that can be present in the /metrics endpoint?
Which exporter is the most suitable for monitoring Scala metrics among the following options?
How to keep pushgateway job labels? normally there are overwritten
How does Prometheus scrape the last batch job push time?
What is the 3 types of service system?
#### Recording & Alerting & Dashboarding (18%)
Is there a way to deactivate a specific route in Alertmanager for a specific time frame?
What is considered a best practice when it comes to alerting in monitoring systems: focusing on alerting based on symptoms or alerting based on causes?
What is the meaning of “alert symptoms” and “alert causes” in the context of monitoring systems?
Which aspect, symptoms or causes, is more visible to customers in the context of an issue?
What is the good naming convention for the recoring rules?
What is the acknowledge-based throttling and Waht is the time-based throttling?
What are the 3 statuses of a Prometheus alert?
How can I use a PromQL query to retrieve the currently active alerts in Alertmanager?
What is the recording rules in Prometheus?
How to define the recording rules?
Whas is the alert fatigue?
Which feature of Alertmanager is responsible for formatting and customizing the alerts?
How can you configure Alertmanager to disable the grouping of alerts for a specific route effectively?
Which software is commonly used for visualizing Prometheus metrics?
What does the term “inhibiting” refer to in the context of Alertmanager?
What is the format used for defining alerting rules?
What is the significance of the for attribute in a Prometheus alert rule?
How can you temporarily mute/snooze/suppress an alert during maintenance in Prometheus?
What is the name of Prometheus native dashboarding and visualization feature?
How can you coordinate the simultaneous sending of multiple alerts with similar label sets in Prometheus?
Which feature of Alertmanager is resonsilbe for sending alert to the right receiver?
What is the purpose of the repeat_interval/conitnue/group_wait/group_inteval attribute in an Alertmanager route configuration?
Which 2 attributes of an alerting rule can be used to include extra metadata?
What are required for a high-availability configuration of Alertmanager?
What are the 3 statuses of Alertmanager Silences?