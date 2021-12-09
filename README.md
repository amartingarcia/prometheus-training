# Training prometheus stack

# Table of contents
1. [Introduction](#introduction-to-prometheus)
   1. [Prometheus Overview](#prometheus-overview)
   2. [How does prometheus work?](#how-does-prometheus-work)
   3. [Prometheus Instalation on Kubernetes](#prometheus-instalation-on-kubernetes)

# Introduction to Prometheus
## Prometheus Overview
* Prometheus is an Open source monitoring solution.
* Started at SoundCloud around 2012-2013, and was made public in early 2015.
* Prometheus provides Metrics & Alerting.
* It is inspired by Google's Borgmon, which uses time-series data as a datasource, to then send alerts bases on this data.
* It fits very well in the cloud native infrastructure.
* Prometheus is also a member of the CNCF (Cloud Native Fundation Computing).
* In Prometheus we talk about Dimensional Data: time series are idenfified by metric name and a set of key/value pairs.

| Metric name  | label  | Sample  |
|--------------|--------|---------|
| Temperature  | location=outside  | 90  |

* Prometheus includes a Flexible Query Language
* Visualizations can be shown using a built-in expression browser or with integrations like Grafana.
* It stores metrics in memory and local disk in an own custom, efficient format
* It si written in Go.
* Many client libraries and integrations available.


## How does Prometheus work?
![Prometheus Work](images/prometheus_work.png)
* Prometheus collects metrics from monitored targets by scraping metrics HTTP endpoints.
  * This is fundamentally different than other monitoring and alerting systems, (except is also how Google's Borgmon works).
  * Rather than using custom scripts that check on particular services and systems, the monitoring data itself is used.
* Scraping endpoints is much more efficient than other mechanisms, like 3rd party agents.
  * a single Prometheus server is able to ingest up to one millon samples per seconds as several million time series.


## Prometheus instalation on Kubernetes
First step, create your enviroment:
```sh
# Minikube start
$ minikube start --network-plugin=cni --cni=calico -p prometheus

# Install kubeprometheus stack
$ prometheus-community/kube-prometheus-stack
```


## Basic Concepts
* All data is stored as time series
  * Every time sries is identified by the __metric name__ and a set of __key-value pairs__, called __labels__.
![Prometheus Concepts](images/prometheus_concepts.png)
| Metric name  | label  | Sample  |
|--------------|--------|---------|
| go_memstat_alloc_bytes  | {container="alertmanager", endpoint="web", instance="10.244.116.72:9093", job="prometheus-kube-prometheus-alertmanager", namespace="default", pod="alertmanager-prometheus-kube-prometheus-alertmanager-0", service="prometheus-kube-prometheus-alertmanager"}  | 8569840  |

* The time series data also consists of the actual data, called Samples:
  * It can be a float64 value
  * or a milisecond-precision timestamp
* The notation of time series is often using this notation:
  * `<metric name>{<label name>=<label value>, ...}`
  * For example:
    * `node_boot_time{instance="localhost:9100",job="node_exporter"}`


## Prometheus configuration
* The configuration is stored in the Prometheus configuration file, in yaml format.
* The default configuration looks like this:

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
```


## Monitoring Nodes with Prometheus
* To monitor nodes, you need to install the node-exporter.
* The node exporter will expose machine metrics of Linux machines.
  * for example: cpu usage, memory usage
* The node exporter can be used to monitor machines, and later on, you can __create alerts based on these ingested metrics__.
* For Windows, there's a WMI exporter.
![Prometheus monitor nodes](images/prometheus_nodes_monitor.png)

* Posteriormente necesitará configurar prometheus para ir a por esas métricas:
```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

....

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```


## Extras
Links:
```sh
# Helm charts
https://github.com/prometheus-community/helm-charts

# Docs
https://prometheus.io/docs/introduction/overview/
```


# Monitoring
## Client Libraries
* Instrumenting your code
* Libraries
  * Official: Go, Java/Scala, Python, Ruby
  * Unofficial: bash, C++, Common Lisp, Elixr, Erlang, Haskell, Lua for Nginx, Lua for Tarantool, .NET/C#, Node.js, PHP, Rust
* No client library available?
  * Implement it yourself in one of the supported exposition formats

* Exposition formats:
  * Simple text-based format
  * Protocol-buffer format (Prometheus 2.0 removed support for the protocol-buffer format)
```
metric_name [
  "{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"
] value [ timestamp ]

# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"}    3 1395066363000

# Escaping in label values:
msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:
something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="0.5"} 129389
http_request_duration_seconds_bucket{le="1"} 133988
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320
```

* 4 types of metrics:
  * __Counter__. 
    * A value that only goes up (e.g Visits to a website).
  * __Gauge__. 
    * Single numeric value that an go up and down (e.g CPU load, temperature)
  * __Histogram__. 
    * Samples observations (e.g. requests durations or reponse sizes) and these observations get counted into __buckets__. Includes (_count and _sum). Main purpose is calculating quantiles.
  * __Summary__. 
    * Similar to a histogram, a summary samples observtiosn (e.g. request durations or response sizes). A summary also provides a total count of observations and a sum of all observed values, it calculates configurable quantiles over a sliding time window.
    * Example: you need 2 counters for calculating the latency
      * 1) Total request(_count)
      * 2) The total latency of those requets (_sum)
      * Take the rate() and divide = average latency.

* Client libraries examples https://prometheus.io/docs/instrumenting/clientlibs/
```py
from prometheus_client import start_http_server, Summary
import random
import time

# Create a metric to track time spent and requests made.
REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')

# Decorate function with metric.
@REQUEST_TIME.time()
def process_request(t):
    """A dummy function that takes some time."""
    time.sleep(t)

if __name__ == '__main__':
    # Start up the server to expose the metrics.
    start_http_server(8000)
    # Generate some requests.
    while True:
        process_request(random.random())
```

## Pushing Metrics
* Sometimes metrics cannot be scraped. Example: batch jobs, servers are not reachable due to NAT, firewall.
* Pushgateway is used as an intermediary service which allows you to push metrics. https://github.com/prometheus/pushgateway
![Prometheus pushgateway](images/prometheus_pushgateway.png).
* Pitfalls
  * Most of the times this is a single instance so this result is a SPOF.
  * Prometheus is automatic instance health monitoring is not possible.
  * The Pushgateway never forgets the metrics unless they are deletd via the api. Example: `curl -X DELETE http://localhost:9091/metrics/job/prom/instance/localhost`.


* Only 1 valid use case for the Pushgateway.
  * Service-lvel batch jobs and not related to a specific machine.
* If NAT or/both firewall is blocking you from using the pull mechanism.
  * Move the Prometheus Server on the same network.

* Example
```py
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway, Summary, Histogram
from time import sleep
from random import randint, random

registry = CollectorRegistry()

cpu_util_sum_metric = Summary('cpu_util_summary', 'cpu_util_summary', registry=registry)
cpu_util_hist_metric = Summary('cpu_util_hist', 'cpu_util_hist', registry=registry)

for i in range(90):
  cpu_util = randint(0, 100)

  cpu_util_sum_metric.observe(float(cpu_util))
  cpu_util_hist_metric.observe(float(cpu_util))
  print('cpu util is: {}'.format(cpu_util))
  res = push_to_gateway('localhost:9091', job='cpu_stats', registry=registry)
  print('push_to_gateway result is:', str(res))
  sleep(5)
```

* __Pushgateway__ functions take a grouping key.
  * __push_to_gateway__ replaces metrics with the same grouping key.
  * __pushadd_to_gateway__ only replaces metrics with the same name and grouping key.
  * __delete_from_gateway__ delete metrics with the given job and grouping key.


## Querying
* Prometheus provides a functional expresions language called __PromQL__.
  * Provides built in operators and functions.
  * Vector-based calculations like Excel.
  * Expressions over time-series vectors.
* PromQL is __read-only__.
* Example:
  * `100 - (avg by (instance) (irate(node_cpu_seconds_total{job='node_exporter',mode="idle"}[5m])) * 100)`

* Instante vector - a set of time series containing a simple sample for each time series, all sharing the same timestamp. Example: `node_cpu_seconds_total`.
* Range vector - a set of time containing a range of data points over time for each time series. Example: `node_cpu_seconds_total[5m`.
* Scalar - a simple numeric floating point value. Example: `-3.14`.
* String - a simple string value; curretly unused. Example: `foobar`.


* [__Arithmetic binary operators__](https://prometheus.io/docs/prometheus/latest/querying/operators/#arithmetic-binary-operators).
* [__Trigonometric binary operators__](https://prometheus.io/docs/prometheus/latest/querying/operators/#trigonometric-binary-operators).
* [__Comparision binary operators__](https://prometheus.io/docs/prometheus/latest/querying/operators/#comparison-binary-operators).
* [__Logical/set binary operators__](https://prometheus.io/docs/prometheus/latest/querying/operators/#logical-set-binary-operators).
* [__Aggregation operators__](https://prometheus.io/docs/prometheus/latest/querying/operators/#aggregation-operators).

* Check all metrics in your Prometheus:
```sh
up
up{job="kubelet"}
prometheus_http_requests_total{job="prometheus-kube-prometheus-prometheus"}
prometheus_http_requests_total{job=~".*prometheus"}
prometheus_http_requests_total{job=~".*prometheus",namespace="default"}
prometheus_http_requests_total{job=~".*prometheus",namespace="default"}[5m]
rate(prometheus_http_requests_total[5m])
sum(rate(prometheus_http_requests_total[5m])) by (job)
```


## Service Discovery
Service Discovery is the automatic detection of devices and services offered by these devices on a computer network.

* Not really a service discovery mechanism
```yaml
static_configs:
    - targets: ['localhost:9090']
```
* Cloud support for (AWS, Azure, Google, ...)
* Cluster managers (Kubernetes, Marathon, ...)
* Generic mechanism (DNS, Consul, Zookerper, ...)

### EC2 Example.
Add following config to prometheus.yaml
```yaml
global:
    scrape_interval: 1s
    evaluation_interval: 1s

scrape_configs:
    - job_name: 'node'
      ec2_sd_configs:
        - region: eu-west-1
          access_key: XX
          secret_key: XX
          port: 9100
      relabel_configs:
        # Only monitor instances with a tag Name starting with "PROD"
        - source_labels: [_meta_ec2_tag_name]
          regex: PROD.*
          action: keep
        # Use de instance ID as the instance label
        - source_labels: [_meta_ec2_instance_id]
          target_label: instance
```
* Make sure the user has the following IAM role: AmazonEC2ReadOnlyAccess
* Make sure you security grups allow access to port (9100, 9090)

### Kubernetes examples
```yaml
- job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (.+)
```

### DNS Example
```yaml
- job_name: 'mysql'
  dns_sd_configs:
    - names:
      - metrics.mysql.example.com
- job_name: 'haproxy'
  dns_sd_configs:
    - names:
      - metrics.haproxy.example.com
```

### Using file
* File example:
```yaml
scrape_configs:
    - job_name: 'dummy'
      file_sd_configs:
        - files:
          - targets.json
```

* Format target.json
```json
[
    {
        "targets": ["myslave1:9104", "mysqlave2:9104"],
        "labels": {
            "env": "prod",
            "job": "slave"
        }
    },
    {
        "targets": ["mymaster:9104"],
        "labels": {
            "env": "prod",
            "job": "master"
        }
    }
]
```
## Exporters
## Extras

# Alerting
## Introduction to Alerting
## Setting Up Alerts
## Extras

# Internals
## Prometheus Storage
## Prometheus Security
## TLS & Authentication on Prometheus Server
## Extras

# Prometheus client implementations
## Monitoring a web application - Python Flask
## Calculating Apdex score
## Monitoring a wb application - Java Spring
## Extras

# Another Use Cases
## AWS Cloudwatch Exporter
## Grafana Provisioning
## Scraping Kubernetes with Prometheus
## Consul integration with Prometheus
## AWS EC2 Auto Discovery
## Extras