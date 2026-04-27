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
  - understand what happening inside a system and predict how it will behave 
- What is RED Method?
  - monitoring 3 metrics: request rate, errors, duration 
- What are the distinctions between SLO, SLI, and SLA?
  - SLA - service level agreements - agreement with customer 
  - SLO - * objective - objective team must hit 
  - SLI - * indicator - number/measurement of your performance 
- In the context of tracing, what is the meaning or representation of a span?
  - span is singple operation/unit of work within a distributed system and captures the start and end time
- In which scenarios is distributed tracing less beneficial or NOT as applicable?
  - for monolith system 
- What are typically tracked within a span of a trace?
  - operation name, trace id, spand id, start and end timestamp, duration, parent span id 

- What is the good and bad metric?
  - bad: a metric with a lot of variance and poor correlation with user experience
  - good: metric to set easier threshold for because there is no overlap 
- Which type of data is monitored by Prometheus?
  - metrics (numeric values )
- In the context of monitoring and observability, what type of data is typically used to define a SLI?
  - metrics, time, percentage success/failure/error 
- What is the meaning or purpose of an error budget policy?
  - error budget policy is concept used in context of SLO and SLA and is to define the acceptable level of errors or service diruptions that a system or service and experience 
- What is one advantage of the push model for recoring metrics compared to pull models?
  - timely and proactive data collection /pushing into the centralized data system 
- How do Prometheus, ELK stack, and InfluxDB differ in terms of their functionalities and use cases?
  - prometheus - metrics, pull based http, simple yet powerful model and query language 
  - elk stack - logs aggregation and indexing, log analysis and visualize, scalable but requrie significant mgmt 
  - influx db - time-series that provide SQL-like query language, push-based system, handled high volume of time-stamped data(IoT, Sensor, Analytics)
- What is the definition of a metric?
  - metric ignore context and focus on continous quantified measurement of system, process or product 
  - for valuable insight 
  - numeric time-series data point 
- What are the Prometheus exemplars?
  - exemplar is a specific trace represent of measurement taken in a given time interval and provides additional information about specific data point
- What is one of the main purposes or goals of logging?
  - provide context for troubleshooting and debugging
  - gather and aggregate textual event data from a service for troubleshooting 
- What are the 3 core components of observability?
  - logs
  - metrics
  - trace
- What is the Monitoring
  - continue observation of a system to detect and alert on abnormal behavior
- What is the Telemetry?
  - automate collection and transmission of data from remote sources 
- What is the challenges of observability?
  - data silos
  - volume, velocity, variety, and complexity data
  - lack of pre-production 
#### Prometheus Fundamentals (20%)
- What is the CLI utility tool for Prometheus called?
  - promtool
- What are the limitations of Prometheus?
  - focus on metrics, not log
  - no long time storage by default 
  - scaling limitation with milions of TS 
  - high cardinality data limitations 
  - federation and HA limit 
- What is Service Discovery and which categories are there?
  - to listen and scrape metrics from app 
  - enable serice to find and comm without harcoded IP addresses or endpoints 
  - categories:
    - top down ( ec2)
    - bottom up( consol) 
- Which property configures the timing to scrape metrics from targets?
  - scrape_intervals
- Which section in the Prometheus configuration file governs the selection of targets to be scraped?
  - scrap_configs
- Which action in the label configuration is used to delete a specific target?
  - scrape_configs -> relabel_configs -> action:drop or action:keep
- How is managed data retention in prometheus?
  - default data retention is 15 days 
  - using tag storage.tsdb.retention.time and storage.tsdb.retention.size in the config file 
- What are the essential 3 components of Prometheus?
  - Retrieval
  - TSDB
  - HTTP server

- What is required to be able to reload Prometheus?
  - send SIGHUP signal to prometheus process using command: kill -HUP <pid>
  - send HTTP POST to /reload endpoint  
- What are 3 methods to restart the Prometheus server?
  - using systemclt restart prometheus
  - reloading prometheus 
  - 
- What HTTP method does Prometheus employ for performing scrapes?
  - pull based ?
  - GET request 
- Which SD configuration is recommended for scraping EC2 instances?
  - Prometheus EC2 Service Discovery 
  - ec2_sd_configs
- Which SD configuration is recommended for nodes of Elastic Kubernetes Service on AWS?
  - ec2_sd_configs

- What is the purpose of the scrape_interval configuration in Prometheus?
  - interval duration of for scrape endpoint
  - how frequently prometheus collect and update the metrics
- Which type of database does Prometheus utilize?
  - time-series database 
- What component is responsible for collecting metrics from an instance and exposing them in a format that Prometheus expects?
  - prometheus exporter
- Which component is suitable for collecting metrics from batch/cron jobs?
  - pushgateway 
- When is the configuration option honor_labels:true used?
  - keep the labels from the scrapped data and ignores conflicting server-side labels
- What is the purpose of port 9090/9093/9100/9091/9115 in Prometheus?
  - 9090: web UI and API
  - 9093: alertmanager
  - 9100: node exporter
  - 9091: pushgateway
  - 9115: blackbox exporter
- what are 2 default metric labels?
  - instance
  - job
- Which of the file systems is recommended/supported by Prometheus?
  - local POSIX file system liek ext4, XFS, NTFS
- How can you configure a Blackbox Exporter probe to check the successful response of your servers to PING?
  - internet control message protocol ( ICMP) -> prober:icmp
- How do you configure the targets that Prometheus should scrape?
  - config in scrape_configs -> static_configs -> targets:xxx
  - target specify 

- What is the agent deployment mode of Prometheus?
  - to be enabled --enable-feature=agent
  - data is only stored locally until it is transmitted to the Remote Write
  - make it more lightweight solution 
  - recommend for most env, as it separate logical responsibility and more efficient way 
  - Collector, a component of OpenTelemetry, can be deployed in agent mode 

- Which CLI command is suitable for unit testing Prometheus rules?
  - promtool to test and validate aspect of prometheus config file and ruels 
  - cmd: promtool test rules /etc/prometheus/first_rules.yml

Which CLI command is suitable for checking validity of the config files?
  - promtool to check config file
  - promtool check rules /etc/prometheus/test.yaml 
- How do you define the targets with SD that Prometheus should collect metrics from?
  - define target
  - scrape_configs and *_sd_configs on per-job basis
  - predefined plugins 
    - aws, azure, kubernetes 
- How can you delete the specific time series metrics of Prometheus?
  - --web.enable-admin-api
  - http request /api/v1/admin/tsdb/delete
  - curl - X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={xxxx="yyy"}'
- How can you delete the all time series metrics of Prometheus?
  - /api/v1/admin/tsdb/delete HTTP endpoint 
  - curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/clean_tombstones'
- Which format does file-based SD provide?
  - simple text file where each line contains a key value pair 
  - yaml and json
#### PromQL (28%)
- What is PromQL?
  - promQL is query language, 
What is histogram metric in Prometheus?
  - historgram_quantile
  - histogram sample observation and count them in config 
- Which 4 data types are used in PromQL?
  - scalar
  - string
  - instant vector
  - range vector
What is the name of the vector in Prometheus that stores a single sample value?
  - instant vector
- Which PromQL function is used to estimate the value of a time series at a future time, t seconds from the current time, based on the range vector v?
  - predict-linear()
- Between what type of expressions can logical operators be defined?
  - boolean
- Which function can be used to calculate the average of a range vector in Prometheus?
  - avg_over_time(metric[x])
- What is the diff between avg(...) and avg_over_time(...)?
  - avg: calculate value across for 1 selected time series, return aggregated number 
  - avg_over_time: calculate distint average value for every selected time series, return range vector as output 
- With which type of metrics is the rate(...) function primarily used in Prometheus?
  - counter 
  - rate return how fast a counter is increasing per second for each time series in the range vector
- What does the term “offset” refer to in Prometheus?
  - allows looking further back into the past or into the future, can set offset of 5h to tell prmetheus to run the query 
- What distinguishes the rate(...) and irate(...) query functions in Prometheus?
  - rate: averaging of change of a time series over the specified time range 
  - irate: only look at 2 sample of the range vector
- What distinguishes the rate(...) and deriv(...) query functions in Prometheus?
  - rate: work on counter 
  - deriv: to know how quickly a gauge is changing, using least-square regression to estimate the slope of each time series 

- Which type of metric is suitable for measuring the internal temperature of a server?
  - gauge
- What is the data type of Prometheus metric values?
  - float64
- How many unique series are generated by a histogram metric type?
  - multiply of number of bucket by number of dimensions 
  - <basename>_bucket, <basename>_sum and <basename>_count
- What are the 4 components of the Prometheus metrics data model?
  - metric name 
  - metric label
  - timestamp 
  - value

- What is the difference between the ceil and floor functions?
  - ceil: always round up to nearest integer
  - floor: always round down
- Which query function among the following returns a result of 1 in case the specified time series does not exist?
  - absent
- What is the logical/arithmetic/comparison binary operator?
  - logical : OR, AND, UNLESS
  - arithmetic: basic algebra
  - comparison: ==, !=, >, <, >=, <=
- What is the vector matching?
  - on, ignoring ?
- What is the group modifiers?
  - a part of vector matching: on, ignoring, + group_left, group_right
- Which function is NOT using counter metrics? irate(), increase(), reset(), idelta(), avg(), rate()
  - idelta
- How to calc the time in days until the LAST certificate expiration?
  - max(cert_expiry - time())/86400
- What is the dimensional aggregation?
  - allow user to summarize metric accrss multipel time series
  - sum, min, max, avg,count 
- What is the significance of the double underscore “__” before a label name?
  - label name with __ are reserved, is used to prevent accidental access and to provide an way to identify metrics uniquely 
#### Instrumentation and Exporters (16%)
- What is the HTTP headers to establish by Prometheus during each scrape?
  - HTTP GET
  - x-prometheus-scrape-timeout-seconds
- Which 2 query parameters are required when configuring a Blackbox Exporter probe?
  - target + module
- What is the exposition format of Prometheus?
  - under /metrics path
  - human readable format: prometheus text format and openmetrics
- Does Prometheus need to perform any format conversion on the metrics returned by a monitored Linux machine?
  - no
- What is the default endpoint that Prometheus uses to scrape the metrics from the target?
  - /metrics
- Where is the version of the Prometheus exporter typically defined?
  - build_info
- What is the most suitable exporter for monitoring an HTTP web server endpoint to verify that it returns a 200 status code?
  - blackbox exporter 
- Which Prometheus exporter is recommended for monitoring network devices?
  - SNMP exporter
- Which networking protocol does Prometheus utilize for performing scrapes?
  - HTTP protocol
- What is the purpose of a Prometheus metrics registry?
  - metrics registry for storing and mange metrics data, allowing for efficient querying and analysis 

- What is the purpose or definition of a Prometheus exporter?
  - exporter to collect metrics from a system or app and expose them in a format that prometheus can scrape 
- In what scenarios would you use the Blackbox Exporter?
  - blackbox exporter to monitor system from the outside without requiring internal knowledge 
  - network service monitoring
  - health check, external monitoring 
- How does Prometheus identify the scrape path for its targets?
  - config file: scrap_configs -> static_configs > metrics_path: /metrics
  - service discovery 
  - annotation 
  - /metrics 
- Which endpoints allows blackbox probing?
  - endpoints over HTTP, HTTPS, DNS, TCP, ICMP
- In a scenario where you have a dynamic etcd database containing scrape targets for Prometheus, how should you configure service discovery?
  - service discovery: file_sd_configs 

- What are the 2 types of attributes that can be present in the /metrics endpoint?
  - HELP, TYPE

- Which exporter is the most suitable for monitoring Scala metrics among the following options?
  - JMX exporter
- How to keep pushgateway job labels? normally there are overwritten
  - honor_label 
- How does Prometheus scrape the last batch job push time?
  - job_last_success_unixtime
- What is the 3 types of service system?
  - online-service, offline processing, batch jobs 
#### Recording & Alerting & Dashboarding (18%)

- Is there a way to deactivate a specific route in Alertmanager for a specific time frame?
  - mute_time_internal
  - time_intervals
-  What is considered a best practice when it comes to alerting in monitoring systems: focusing on alerting based on symptoms or alerting based on causes?
  - based on symptons 
- What is the meaning of “alert symptoms” and “alert causes” in the context of monitoring systems?
  - alert symptoms: focus on symptons, may have many causes, "what broken" 
  - alert cause: focus on underlying issues, "why broken" 
- Which aspect, symptoms or causes, is more visible to customers in the context of an issue?
  - symptons
- What is the good naming convention for the recoring rules?
  - level:metric:operations
- What is the acknowledge-based throttling and Waht is the time-based throttling?
  - throttling: mechanism to prenvent too many executions fo the same actions 
  - acknowledge-based throttling: prevent watch action from being executed again while the watch condition remains true 
  - time-based throttling: limits how often an action is executed by defining a throttling period 
- What are the 3 statuses of a Prometheus alert?
  - inactive, pending, firign
- How can I use a PromQL query to retrieve the currently active alerts in Alertmanager?
  - Menu > alerts > query > alerts 
- What is the recording rules in Prometheus?
  - predefined processing step for metrics data
  - aggregate and filter metrics with promQL and storing them into Prometheus DB 
- How to define the recording rules?
  - yaml file 
  - rules -> record: xxx, expr: xxx
- Whas is the alert fatigue?
  - too many alerts triggering lead make the monitor ineffective 
- Which feature of Alertmanager is responsible for formatting and customizing the alerts?
  - notification template
- How can you configure Alertmanager to disable the grouping of alerts for a specific route effectively?
  - group_by
- Which software is commonly used for visualizing Prometheus metrics?
  - grafana
- What does the term “inhibiting” refer to in the context of Alertmanager?
  - suppresing low priority alert while high priority is firing
  - allow certain alerts to be stopped or prevented from generate noti for specified time  
- What is the format used for defining alerting rules?
  - yaml 
- What is the significance of the for attribute in a Prometheus alert rule?
  - specifies the duration for a condition must be true before an alert is triggered 
- How can you temporarily mute/snooze/suppress an alert during maintenance in Prometheus?
  - Silence
- What is the name of Prometheus native dashboarding and visualization feature?
  - Prometheus Console
- How can you coordinate the simultaneous sending of multiple alerts with similar label sets in Prometheus?
  - Grouping 
- Which feature of Alertmanager is resonsilbe for sending alert to the right receiver?
  - routing
- What is the purpose of the repeat_interval/conitnue/group_wait/group_inteval attribute in an Alertmanager route configuration?
  - control noti frequency and behavior for alert group 
  - repeat_interval: wait time before a firing alert that has already been succesful
  - contine: whether to continue processing subsequent routes 
  - group_wait: set how long initially wait to send a noti
  - group_internal: how long to wait before send noti about new alert
- Which 2 attributes of an alerting rule can be used to include extra metadata?
  - labels and annotations 
- What are required for a high-availability configuration of Alertmanager?
  - --cluster-* flags 
- What are the 3 statuses of Alertmanager Silences?
  - Active, Pending, Expired 