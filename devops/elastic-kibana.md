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
```
