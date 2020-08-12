# Prometheus
## Architecture
<p align="center">
  <img src="https://raw.githubusercontent.com/apurvabhandari/kubernetes/master/monitoring/prometheus_architecture.png">
</p>

## Features

- a multi-dimensional data model with time series data identified by metric name and key/value pairs
- PromQL, a flexible query language to leverage this dimensionality
- no reliance on distributed storage; single server nodes are autonomous
- time series collection happens via a pull model over HTTP
- pushing time series is supported via an intermediary gateway
- targets are discovered via service discovery or static configuration
- multiple modes of graphing and dashboarding support

## Components
The Prometheus ecosystem consists of multiple components, many of which are optional:

- the main Prometheus server which scrapes and stores time series data
- client libraries for instrumenting application code
- a push gateway for supporting short-lived jobs
- special-purpose exporters for services like HAProxy, StatsD, Graphite, etc.
- an alertmanager to handle alerts
- various support tools

## CLIENT LIBRARIES

Before you can monitor your services, you need to add instrumentation to their code via one of the Prometheus client libraries. These implement the Prometheus metric types.

## METRIC TYPES
The Prometheus client libraries offer four core metric types. These are currently only differentiated in the client libraries (to enable APIs tailored to the usage of the specific types) and in the wire protocol. The Prometheus server does not yet make use of the type information and flattens all data into untyped time series. This may change in the future.

- **Counter:** A counter is a cumulative metric that represents a single monotonically increasing counter whose value can only increase or be reset to zero on restart. For example, you can use a counter to represent the number of requests served, tasks completed, or errors.
  
  Do not use a counter to expose a value that can decrease. For example, do not use a counter for the number of currently running processes; instead use a gauge.
- **Gauge:** A gauge is a metric that represents a single numerical value that can arbitrarily go up and down.

  Gauges are typically used for measured values like temperatures or current memory usage, but also "counts" that can go up and down, like the number of concurrent requests.<br>
- **Histogram:** A histogram samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values.

   A histogram with a base metric name of <basename> exposes multiple time series during a scrape:
  - cumulative counters for the observation buckets, exposed as <basename>_bucket{le="<upper inclusive bound>"}
  - the total sum of all observed values, exposed as <basename>_sum
  - the count of events that have been observed, exposed as <basename>_count (identical to <basename>_bucket{le="+Inf"} above)
  Use the histogram_quantile() function to calculate quantiles from histograms or even aggregations of histograms. A histogram is also suitable to calculate an Apdex score. When operating on buckets, remember that the histogram is cumulative. See histograms and summaries for details of histogram usage and differences to summaries.
- **Summary:** Similar to a histogram, a summary samples observations (usually things like request durations and response sizes). While it also provides a total count of observations and a sum of all observed values, it calculates configurable quantiles over a sliding time window.

   A summary with a base metric name of <basename> exposes multiple time series during a scrape:
  - streaming φ-quantiles (0 ≤ φ ≤ 1) of observed events, exposed as <basename>{quantile="<φ>"}
  - the total sum of all observed values, exposed as <basename>_sum
  - the count of events that have been observed, exposed as <basename>_count
  - See histograms and summaries for detailed explanations of φ-quantiles, summary usage, and differences to histograms.
 

## Sample kubernetes monitoring snippet
```
scrape_configs:
- job_name: 'kubernetes-apiservers'

  kubernetes_sd_configs:
  - role: pod

  scheme: https

  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_label_app_type]
    separator: ;
    regex: (storage-app.*)
    replacement: $1
    action: keep
  - source_labels: [__meta_kubernetes_pod_label_app_mode]
    separator: ;
    regex: (sg-.*)
    replacement: $1
    action: keep
```




# Reference 
[Prometheus.io](https://prometheus.io/)<br>
