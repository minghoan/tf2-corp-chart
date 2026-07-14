# Import Grafana Dashboards: Kubernetes Pod Health & Kafka Consumer Lag

**Date:** 2026-07-14  
**Files added:**
- `grafana/provisioning/dashboards/k8s-pod-health.json`
- `grafana/provisioning/dashboards/kafka-consumer-lag.json`

## Kubernetes Pod Health (`k8s-pod-health.json`)

Adapted from Grafana community dashboard ID 15760. Datasource hardcoded to
`webstore-metrics` (the project's Prometheus UID).

**Panels:**
- Stat row: Running pods, Pending/Failed pods, OOMKilled (last 1h), Container restarts (last 1h)
- CPU Usage by Pod (timeseries, `rate(container_cpu_usage_seconds_total[5m])`)
- CPU Throttling by Pod (timeseries, throttled vs total CFS periods)
- Memory Working Set by Pod (timeseries, `container_memory_working_set_bytes`)
- Memory Usage vs Limit % (timeseries)
- Container Restarts table (all containers with restarts > 0)
- OOMKilled Containers table (last 1h)
- Pod Phase Summary table (phase coloring: Running=green, Pending=yellow, Failed=red)

**Variables:** `$namespace` (multi-select), `$pod` (multi-select, filtered by namespace).

**Requires:** kube-state-metrics and cAdvisor metrics in Prometheus (standard kube-prometheus-stack).

---

## Kafka Consumer Lag (`kafka-consumer-lag.json`)

Custom dashboard targeting the `accounting` and `fraud-detection` consumer groups.
Metrics come from the OTel `kafkametrics` receiver (scrapers: `brokers`, `topics`, `consumers`)
already configured in the collector pipeline, exported to Prometheus via `otlphttp/prometheus`.

**Panels — accounting section:**
- Total consumer lag stat (color: green < 100, yellow < 1k, red ≥ 1k)
- Active partitions with lag count
- Consumer lag per partition timeseries with threshold shading
- Consumer offset rate (msg/s) timeseries

**Panels — fraud-detection section:** (same structure as accounting)

**Panels — Broker & Topic Overview:**
- Active brokers count (`kafka_brokers`)
- Total topic partitions (`kafka_topic_partitions`)
- Consumer Group Lag Summary table — combined view of both groups, sorted by lag desc,
  color-coded background

**Variables:**
- `$accounting_group` — auto-populated from `kafka_consumer_group_lag`, regex `.*account.*`
- `$fraud_group` — auto-populated from `kafka_consumer_group_lag`, regex `.*fraud.*`
- `$topic` — multi-select all topics (All by default)

**Key metrics used:**
| Metric | Description |
|---|---|
| `kafka_consumer_group_lag` | Messages behind latest offset, per group/topic/partition |
| `kafka_consumer_group_offset` | Current committed offset |
| `kafka_brokers` | Number of active brokers |
| `kafka_topic_partitions` | Partition count per topic |

**Note on group name regex:** The `.*account.*` and `.*fraud.*` regexes filter the variable
drop-downs automatically. If the actual Kafka consumer group IDs use a different naming
convention (e.g. `checkout-consumer` instead of `accounting`), update the `regex` field
in the variable definition inside the JSON or rename the consumer groups on the application side.
