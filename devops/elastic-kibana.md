# Elastic Stack, Kibana tips
Get content directly from Elastic API
````
curl -u elastic:password_here -X GET "http://10.10.10.10:9200/index-name-or-alias/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "size": 10
}'
````
## Kibana DEV tools queries
```
// Search through index
GET /k8s-dev-cluster/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "exists": {
            "field": "latency"
          }
        },
        {
          "term": {
            "application_name.keyword": "application-backend"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-15m",
              "lte": "now"
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ],
  "size": 50  
}

// See the mappings
GET /k8s-dev-cluster/_mapping

// Space consumption per index
GET /_cat/indices?v&h=index,store.size
..
index                                                      store.size
some-service-swarm-2024.03.28-000001                          1gb
machine-learning-2023.11.18                                   354.4kb
...

// Get index creation date
GET /_cat/indices?v&h=index,creation.date.string
..
index                                             creation.date.string
some-service-swarm-2024.03.28-000001              2024-03-28T06:29:40.175Z
.monitoring-logstash-7-2024.05.06                 2024-05-06T00:00:04.859Z

// Index usage statistics
GET /_nodes/stats/indices

// Manually delete an index
DELETE /index_name

GET /_cat/indices
..
yellow open .monitoring-logstash-7-2024.05.07          FW3uc31bT3-5aSPYks8GCA 1 1    1415976      0 157.5mb 157.5mb
green  open some-gateway-swarm-2023.11.28-000001       SWWKJzI7S5C8wBljIE5b2w 1 1          0      0    450b    225b
green  open machine-learning-2023.11.20                z9mymsHsTJCLLwAjjmOtnw 1 1          9      0 215.7kb 110.4kb
green  open k8s-cluster-2023.11.28-000001              S8Vnzq_gQ3eEhUlwIfj9xg 1 1          0      0    450b    225b

// GET ILM Policy
GET _ilm/policy/.monitoring-8-ilm-policy

// GET datastream
GET _data_stream/<data_stream_name>

// Clean ILM policy from indicies using pattern (in case it was deleted from the associated template, but the old indicies still using previous settings)
PUT /my-index*/_settings
{
  "index": {
    "lifecycle": {
      "name": null
    }
  }
}

// PUT deprecated template :D With data types
PUT _template/patroni_logs
{
  "index_patterns": ["patroni_logs*"],
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "log_timestamp": { "type": "date" },
      "timezone": { "type": "keyword" },
      "db_user": { "type": "keyword" },
      "db_name": { "type": "keyword" },
      "process_id": { "type": "integer" },
      "client_address": { "type": "ip" },
      "session_id": { "type": "keyword" },
      "transaction_id": { "type": "integer" },
      "statement": { "type": "text" },
      "session_start_time": { "type": "date" },
      "session_start_timezone": { "type": "keyword" },
      "virtual_transaction_id": { "type": "keyword" },
      "backend_start": { "type": "integer" },
      "log_level": { "type": "keyword" },
      "error_code": { "type": "keyword" },
      "query": { "type": "text" },
      "client_type": { "type": "keyword" }
    }
  }
}
```
### Case №1. I have an index which consumes alot of disk space and it weren't rotating for years and doesn't have ILM policy
```
// Creat a policy
PUT _ilm/policy/delete-after-14-days
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "14d"
          }
        }
      },
      "delete": {
        "min_age": "14d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

// Create and associate template with policy, be careful, if it's already exists the configuration will be overwrited
PUT _index_template/my_template
{
  "index_patterns": ["my-index*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "delete-after-14-days",
      "index.lifecycle.rollover_alias": "my-index-alias"
    }
  }
}

// Do rollover
POST /my-index-alias/_rollover
```
### Case №2. Index health is yellow, why?
```
GET /_cluster/health

GET /_cluster/health/<index_name>

GET /_cat/shards/<index_name>?v

GET /_cluster/allocation/explain
{
  "index": "<index_name>",
  "shard": <shard_number>,
  "primary": false
}
```
### Can't store an async search response larger than [10485760] bytes. This limit can be set by changing the [search.max_async_search_response_size] setting.
You can increase that limitation, but it will put more memory usage on Elastic JVM.
```
PUT /_cluster/settings 
{
  "persistent": { 
    "search.max_async_search_response_size":"50mb"
  }
}
```
### Check nodes state in the cluster after adding new master nodes
```
// Total count of nodes, count of data nodes
GET /_cluster/health?pretty

// Nodes, their state and roles
GET /_cluster/state?pretty

// List all nodes with ip and names
GET /_cat/nodes?v

// More clean request for roles of the nodes
GET /_nodes?filter_path=nodes.*.roles,pretty

// Get the leader node
GET /_cat/master?v
```
## Logstash tips
Enable verbose output to files in case of debugging
```
output {
  if ([type]=="patroni_logs"){
    elasticsearch {
      hosts => ["10.65.8.203:9200"]
      user=> "elastic"
      password => "gsgg"
      index => "patroni_logs-%{+YYYY.MM.dd}"
      manage_template => "true"
        template_name => "patroni_logs"
    }
    file {
      path => "/tmp/logstash_output_rubydebug.log"
      codec => rubydebug { metadata => true }
    }
    file {
      path => "/tmp/logstash_output_json.log"
      codec => json_lines
    }
  }
}

```
Grok pattern for postgres which was damn overload, high CPU usage, pipeline timouts
```
filter {
  if ([type] == "postgres_logs") {
    grok {
       match => { "message" => "%{TIMESTAMP_ISO8601:log_timestamp} %{DATA:timezone},(\"%{DATA:db_user}\")?,(\"%{DATA:db_name}\")?,(%{DATA:process_id})?,(\"%{DATA:client_address}\")?,(%{DATA:session_id})?,(%{DATA:transaction_id})?,(\"%{DATA:statement}\")?,(%{DATA:session_start_time})? (%{DATA:session_start_timezone})?,(%{DATA:virtual_transaction_id})?,(%{DATA:backend_start})?,(%{DATA:log_level})?,(%{DATA:error_code})?,(\"%{DATA:info_message}\")?,(\"%{DATA}\")?,(\"%{DATA}\")?,(\"%{DATA}\")?,(\"%{DATA}\")?,(\"%{DATA}\")?,(\"%{GREEDYDATA:query}\")?,(%{DATA})?,(%{DATA})?,(\"%{DATA}\")?,(\"%{DATA:client_type}\")?$" }
       remove_field => ["message"]
    }
    date {
      match => [ "log_timestamp", "ISO8601" ]
      target => "@timestamp"
    }
  }
}
```
Dissect filter, which is lightweight. There were no problems at all, but less field data
```
filter {
  if ([type] == "postgres_logs") {
    dissect {
      mapping => {
        'message' => '%{+log_timestamp} %{+log_timestamp/2} %{timezone},%{db_user},%{db_name},%{process_id},%{client_address},%{session_id},%{transaction_id},%{statement},%{+session_start_time} %{+session_start_time/2} %{session_start_timezone},%{virtual_transaction_id},%{backend_start},%{log_level},%{error_code},%{}'
      }
    }
    date {
      match => [ "log_timestamp", "ISO8601" ]
      target => "@timestamp"
    }
  }
}

```
