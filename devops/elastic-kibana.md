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
