# Victoria Metrics, Prometheus, Alertmanager, Grafana OnCall tips
## Victoria Metrics
### Streaming aggregation
The aggregations are calculated during the interval specified in the config and then sent to the storage once per interval. The aggregated samples are named according to output metric naming . <br>
If by and without lists are specified in the config , then the aggregation by labels is performed additionally to aggregation by interval. <br>
By default, stream aggregation and deduplication stores a single state per each aggregation output result. 
## Prometheus
List all possible lable values in PromQL
````
count by (app) (company_http_external_request_total)
````
### Figuring out Prometheus current capacity
It gives you an idea of how many unique time series are currently being tracked and stored in memory by Prometheus.
```
prometheus_tsdb_head_series
```
Use this metric to monitor the current load on Prometheus in terms of the number of active time series. A significant increase in this metric can indicate that your Prometheus instance is handling more unique time series, which can affect memory usage and performance.

It shows the cumulative count of all samples (data points) that have been ingested by Prometheus.
```
prometheus_tsdb_head_samples_appended_total
```
Use this metric to understand the volume of data being ingested over time. It helps in monitoring the ingestion rate and identifying any sudden spikes in data input, which can affect both performance and resource usage.
```
prometheus_tsdb_head_series is a gauge metric, indicating the current number of active time series at a given point in time.
prometheus_tsdb_head_samples_appended_total is a counter metric, representing a continuously increasing total number of samples ingested.
```
## Alerting
### Alert routing from prometheus to Grafana OnCall -> Telegram
Example of prometheus alert
```
  - alert: BadRequestApplication
    annotations:
      description: The number of requests with status code 400 is 2 times the average
        for the last 24 hours
      summary: Application increase of requests with status code 400
    expr: sum(rate(app_request_duration_seconds_count{app="application", code_group="400"}[1m]))
      > 3 * avg_over_time(sum(rate(app_request_duration_seconds_count{app="application",
      code_group=~"400"}[1m]))[24h:1m])
    for: 1m
    labels:
      severity: critical
      team: monitoring             // Add a custom label
```
Example of alertmanager config
```
global:
  resolve_timeout: 3m
templates:
- '/etc/alertmanager/templates/*.tmpl'
receivers:
- name: "blackhole"
  webhook_configs:
  - send_resolved: true
    url: "https://oncall.company.com/integrations/v1/alertmanager/cegeegegegeeg"
    max_alerts: 100
- name: "monitoring"
  webhook_configs:
  - send_resolved: true
    url: "https://oncall.company.com/integrations/v1/alertmanager/zbzbbbzdfb/"
    max_alerts: 100
route:
  receiver: "blackhole"
  routes:
  - match_re:
#     alertname: '^MonitoringTeam_.*'   // You can also filter by alertnames, but by labels is more neat
      team: monitoring                  // Filter alerts by label
    group_by: ['alertname']
    receiver: "monitoring"
```
Example of routing template defined in Grafana Oncall
```
{% set commonLabels = payload.get("commonLabels", {}) -%}
{% set severity = commonLabels.severity -%}
{% set status = payload.get("status", "Unknown") -%}
{% set alertname = commonLabels.alertname -%}
{% set team = commonLabels.team -%}
{{ 
  severity != "info" and severity != "none" and 
  team == "monitoring"
}}
```
Or you can filter by names using regexp
```
{{ 
  severity != "info" and severity != "none" and 
  alertname | regex_match("^MonitoringTeam_.*")
}}
```
### Variables
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/variables-ds-regexp.png?raw=true)
