# promql
This repository is a way to learn, store, and share top promql queries.

promql: [Prometheus](https://prometheus.io/) provides a functional query language called PromQL

## **Why labels are so important in Prometheus?**
Labels will provide a way to do a granular search after the data is collected and also write a very configurable Alertmanager configuration. Before collecting data it's very important to define and use a set of labels that will have a definite meaning.

Labels can be(examples)
- `region: us-east-1`
- `product: ticketing-system`
- `env: qa`
- `host: ticketing.qa.example.org`
- `cloud-provider: aws`
- `instance-type: t2.micro`
- `instance-life: spot/ondemand/reserved`

## Visualizing the data over Grafana to allow dynamic filtering of data?

Grafana has built-in support for Prometheus and the concept of the [variable]((https://grafana.com/docs/grafana/latest/variables/templates-and-variables/)) is the icing on the cake if used correctly.

![Grafana Drilldown Prometheus](https://github.com/shubhamc183/promql/blob/master/media/grfana_drill_down_prometheus.png?raw=true)

[Grafana provides a way to get all labels, metrics and query the Promtheus.](https://grafana.com/docs/grafana/latest/features/datasources/prometheus/#query-variable)
| label_names()               | Returns a list of label names.                                        |
|-----------------------------|-----------------------------------------------------------------------|
| label_values(label)         | Returns a list of label values for the label in every metric.         |
| label_values(metric, label) | Returns a list of label values for the label in the specified metric. |
| metrics(metric)             | Returns a list of metrics matching the specified metric regex.        |
| query_result(query)         | Returns a list of Prometheus query result for the query.              |

1. First of all, make the data source as a variable so that it your dashboard is not limited to any specific Prometheus
   1. Let us name it "datasource"
   2. Now, all your queries will use it.

![Grafana datasource variable](https://github.com/shubhamc183/promql/blob/master/media/datasource.png?raw=true)

2. Now all your variables will depend upon it.

3. If you are using Prometheus in federation mode then you may need to further select any specific Prometheus or all of them.

![Grafana prometheus federation variable](https://github.com/shubhamc183/promql/blob/master/media/prometheus_ferderation_var.png?raw=true)

If you see here I have used
- variable type = Query
- Query = `label_values(prometheus)`, here it means I want all the label values names "prometheus"(bad choice for federation though) 
- Multi-value= Enabled, because I want my users to select more than one value of the vars like "dev" + "qa" + "stg" + "prod"
- Custom all value= .*, so that characters length is very less in query made by grafana on your behalf to grafana. To support these write all queries as `prometheus=~"$promtheus"` instead of `prometheus="$promtheus"`.

4. Now let us make a variable called "region" and it value will depend on what "prometheus" we have selected from the above "prometheus".
   1. `label_values(node_uname_info{prometheus=~"$prometheus"}, region)`
   2. Here, we are filtering on the basic of prometheus(which is the name of label which represent a fedeated promtheus in this use case)

![Grafana prometheus region variable](https://github.com/shubhamc183/promql/blob/master/media/region_var.png?raw=true)

5. Make a variable called "team" and its value will depend upong the "prometheus" and "region" you have selected above.
   1. `label_values(node_uname_info{prometheus=~"$prometheus", region=~"$region"},  team)`
   2. Here, we are filtering on the basic of prometheus(which is the name of label which represent a fedeated promtheus in this use case)

6. Make a variable called "instance_type" and its value will depend upong the "prometheus", "region", and "team" you have selected above.
   1. `label_values(node_uname_info{prometheus=~"$prometheus", region=~"$region", team=~"$team"},  instance_type)`

7. Like these we can make as much variable we want to provide users a robust data search.

8. At last we need to make use of these variables in our panel queries
   1. `CPU Usage Percentage: 1 - avg(irate(node_cpu_seconds_total{prometheus=~"$prometheus", region=~"$region", team=~"$team",  instance_type=~"$instance_type", instance=~"$instance", mode="idle"}[5m])) by (instance)`

## rate vs irate
  - The rate takes the difference of the first and last two points within that range (allowing for counter resets) to calculate a per-second rate.
  - Whereas irate takes the difference of two consecutive values
  - rate smoothes the graph and good for graph over longer periods
  - irate shows a sudden spike.
  - Prefer rate for alerting
  - [Irate graphs are better graphs](https://www.robustperception.io/irate-graphs-are-better-graphs)

## Queries

## [Node Exporter](https://github.com/prometheus/node_exporter)
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

## [BlackBoxExporter](https://github.com/prometheus/blackbox_exporter)(tcp_probe)
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

## Nice articles to read for further learning
- https://www.robustperception.io/ is the best!
- https://timber.io/blog/promql-for-humans/
- https://medium.com/@valyala/promql-tutorial-for-beginners-9ab455142085
- https://stackoverflow.com/questions/58747562/how-to-get-max-cpu-useage-of-a-pod-in-kubernetes-over-a-time-interval-say-30-da