# Prometheus
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
