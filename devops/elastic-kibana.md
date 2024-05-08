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
```
