# prometheus-jsonpath-exporter

Converts json data from a http url into prometheus metrics using jsonpath


### Config

```yml
exporter_port: 9158 # Port on which prometheus can call this exporter to get metrics
log_level: info
json_data_urls
  - url: http://stubonweb.herokuapp.com/ # Base url to get json data used for fetching metric values
    app: kong_cluster_1
    env: developement
    kind: kong_cluster # All metrics defined in kind "kong_cluster" will be executed on this endpoint
  - url: http://stubonweb.herokuapp.com/ 
    app: kong_cluster_1
    env: production
    kind: kong_cluster 
  - url: http://stubonweb.herokuapp.com/ 
    app: kong_cluster_2
    env: developement
    kind: kong_cluster 
  - url: http://stubonweb.herokuapp.com/ 
    app: kong_cluster_2
    env: production
    kind: kong_cluster 


metrics:
  - kind: kong_cluster
    prefix: kong_cluster # All metric names will be prefixed with this value
    jsonpath: kong-cluster-status # path to json
    metric:
      - name: total_nodes # Final metric name will be kong_cluster_total_nodes
        description: Total number of nodes in kong cluster
        path: $.total
      - name: alive_nodes # Final metric name will be kong_cluster_alive_nodes
        description: Number of live nodes in kong cluster
        path: count($.data[@.status is "alive"])
```

See the example below to understand how the json data and metrics will look for this config

### Run

#### Using code (local)

```
# Ensure python 2.x and pip installed
pip install -r app/requirements.txt
python app/exporter.py example/config.yml
```

#### Using docker

```
docker run -p 9158:9158 -v $(pwd)/example/config.yml:/etc/prometheus-jsonpath-exporter/config.yml sunbird/prometheus-jsonpath-exporter /etc/prometheus-jsonpath-exporter/config.yml
```

### JsonPath Syntax

This exporter uses [objectpath](http://objectpath.org) python library. The syntax is documented [here](http://objectpath.org/reference.html)

### Example

For the above config, if the configured endpoint returns

```json
{
  "data": [
    {
      "address": "x.x.x.15:7946",
      "status": "failed"
    },
    {
      "address": "x.x.x.19:7946",
      "status": "alive"
    },
    {
      "address": "x.x.x.12:7946",
      "status": "alive"
    }
  ],
  "total": 3
}
```

Metrics will available in http://localhost:9158



```
$ curl -s localhost:9158 | grep -v ^#
kong_cluster_total_nodes{app="kong_cluster_1",environment="developement",kind="kong_cluster"} 3.0
kong_cluster_alive_nodes{app="kong_cluster_1",environment="developement",kind="kong_cluster"} 2.0
kong_cluster_total_nodes{app="kong_cluster_1",environment="production",kind="kong_cluster"} 3.0
kong_cluster_alive_nodes{app="kong_cluster_1",environment="production",kind="kong_cluster"} 2.0
kong_cluster_total_nodes{app="kong_cluster_2",environment="developement",kind="kong_cluster"} 3.0
kong_cluster_alive_nodes{app="kong_cluster_2",environment="developement",kind="kong_cluster"} 2.0
kong_cluster_total_nodes{app="kong_cluster_2",environment="production",kind="kong_cluster"} 3.0
kong_cluster_alive_nodes{app="kong_cluster_2",environment="production",kind="kong_cluster"} 2.0
python_info{implementation="CPython",major="2",minor="7",patchlevel="13",version="2.7.13"} 1.0
process_virtual_memory_bytes 1.06340352e+08
process_resident_memory_bytes 4.2151936e+07
process_start_time_seconds 1.57537571808e+09
process_cpu_seconds_total 0.41000000000000003
process_open_fds 7.0
process_max_fds 1.048576e+06


```

