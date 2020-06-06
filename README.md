# promql
This repository is a way to learn and store top promql queries.


## **Why labels are so important in Prometheus?**
Labels will provide a way to do a granular search after the data is collected and also write a configurable Alertmanager configuration. Before collecting data it's very important to define and use a set of labels that will have a definite meaning.

Labels can be(examples)
- `region: us-east-1`
- `product: ticketing-system`
- `env: qa`
- `host: ticketing.qa.example.org`
- `cloud-provider: aws`
- `instance-type: t2.micro`
- `instance-life: stop`

## Visualizing the data in Grafana
Grafana has built-in support for Prometheus and the concept of the variable is the icing on the cake.
![Grfana Drilldown Prometheus](https://github.com/shubhamc183/promql/blob/master/media/grfana_drill_down_prometheus.png?raw=true)

## rate vs irate
  - The rate takes the difference of the first and last two points within that range (allowing for counter resets) to calculate a per-second rate.
  - Whereas irate takes the difference of two consecutive values
  - rate smoothes the graph and good for graph over longer periods
  - irate shows a sudden spike.
  - Prefer rate for alerting
  - [Irate graphs are better graphs](https://www.robustperception.io/irate-graphs-are-better-graphs)

## Queries

## Node Exporter
- CPU Usage in last 5 mins
  - `(1 - avg(rate(node_cpu_seconds_total{instance=~"$instance",mode="idle"}[5m])) by (instance)) * 100`
- RAM Used
  - `(1 - node_memory_MemAvailable_bytes{instance=~"$instance"}/node_memory_MemTotal_bytes{instance=~"$instance"})*100`
- SWAP Used
  - `((node_memory_SwapTotal_bytes{instance=~"$instance"} - node_memory_SwapFree_bytes{instance=~"$instance"}) / (node_memory_SwapTotal_bytes{instance=~"$instance"} )) * 100`
- Number of CPU Cores
  - `sum(count(node_cpu_seconds_total{instance=~"$instance", mode='system'}) by (cpu))`
- Total RAM
  - `sum(node_memory_MemTotal_bytes{instance=~"$instance"})`
- Total SWAP
  - `node_memory_SwapTotal_bytes{instance=~"$instance"}`
- Memory Breakdown
  - Total: `node_memory_MemTotal_bytes{instance=~"$instance"}`
  - Available: `node_memory_MemAvailable_bytes{instance=~"$instance"}`
  - Cache + Buffer: `node_memory_Cached_bytes{instance=~"$instance"} + node_memory_Buffers_bytes{instance=~"$instance"}`
  - Free: `node_memory_MemFree_bytes{instance=~"$instance"}`
  - SWAP Used: `(node_memory_SwapTotal_bytes{instance=~"$instance"} - node_memory_SwapFree_bytes{instance=~"$instance"})`
- Node Load Percentage
  - 1min: `avg(node_load1{instance=~"$instance"}) / count(count(node_cpu_seconds_total{instance=~"$instance"}) by (cpu)) * 100`
  - 5min: `avg(node_load5{instance=~"$instance"}) / count(count(node_cpu_seconds_total{instance=~"$instance"}) by (cpu)) * 100`
  - 15min: `avg(node_load15{instance=~"$instance"}) / count(count(node_cpu_seconds_total{instance=~"$instance"}) by (cpu)) * 100`
- Disk Usage
  - `(1 - node_filesystem_avail_bytes{instance=~"$instance",device!~'rootfs'}/ node_filesystem_size_bytes{instance=~"$instance",device!~'rootfs'}) * 100`
- Disk IO
  - Completed iops
    - Read: `rate(node_disk_reads_completed_total{instance=~"$instance"}[5m])`
    - Write: `irate(node_disk_writes_completed_total{instance=~"$instance"}[5m])`
  - Read/Write kBs
    - Read: `irate(node_disk_read_bytes_total{instance=~"$instance"}[5m])`
    - Write: `irate(node_disk_written_bytes_total{instance=~"$instance"}[5m])**`
- Top 5 used and unsued node in last 1 hour for 5 mins(1min as step size)
  - topk(5, ((1 - avg(avg_over_time(rate(node_cpu_seconds_total{mode="idle"}[5m])[1h:1m])) by (host)) * 100))
  - bottomk(5, ((1 - avg(avg_over_time(rate(node_cpu_seconds_total{mode="idle"}[5m])[1h:1m])) by (host)) * 100))

## Black Box Exporter(tcp_probe)
- Probe Success Count
  - `count(probe_success == 1)`
- Probe Failure Count
  - `count(probe_success == 0)`
- Probe Success Percentage
  - `(count(probe_success == 1))/count(probe_success) * 100`
- Average Probe Duration
  - Take only consideration where probe succuess is true as false one will increase the value very much resulting in wrong interpretations
  - `avg(probe_duration_seconds and probe_success == 1)`
- Average DNS Lookup
  - `avg(probe_dns_lookup_time_seconds)`

## Nice articles
- https://www.robustperception.io/ is the best!
- https://timber.io/blog/promql-for-humans/
- https://medium.com/@valyala/promql-tutorial-for-beginners-9ab455142085
- https://stackoverflow.com/questions/58747562/how-to-get-max-cpu-useage-of-a-pod-in-kubernetes-over-a-time-interval-say-30-da