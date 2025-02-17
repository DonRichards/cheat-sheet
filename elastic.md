#Elasticsearch
[examples](https://dzone.com/articles/23-useful-elasticsearch-example-queries)  
[examples](https://www.tutorialspoint.com/elasticsearch)  

## installation
```sh
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update && sudo apt install elasticsearch
sudo systemctl start elasticsearch
curl -X GET 'http://localhost:9200'
```

## collaboration
```bash
# common part
ELASTIC_HOST=https://elasticsearch-label-search-prod.apps.vantage.org
INDEX_NAME=ubs-single-autolabel
```

### check connection
```sh
# version info
curl -X GET $ELASTIC_HOST

# health check
curl -X GET $ELASTIC_HOST/_cluster/health?pretty=true
curl -X GET $ELASTIC_HOST/_cluster/health?pretty=true&level=shards
curl -X GET $ELASTIC_HOST/$INDEX_NAME
```


### index
[create index](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/indices-create-index.html)
[mapping](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/mapping.html)
#### index info
```bash
# all indexes
curl -X GET $ELASTIC_HOST/_cat/indices | grep ubs | grep label
# count records by index
curl -X GET $ELASTIC_HOST/_cat/count/$INDEX_NAME
```

#### create index from file
```sh
curl -X POST $ELASTIC_HOST/$INDEX_NAME/_mapping/label \
-H 'Content-Type: application/json' \
-d @labels_mappings.json
```/

#### create index inline
```sh
# get index
curl -s --user "$SEARCH_USER:$SEARCH_PASSWORD" -X GET $ELASTIC_HOST/$ELASTIC_INDEX > file_with_index.json

# for using just have read index, pls remove next lines:
# {"index_name": {"aliases": {}, "mappings": {  # and last } 
# settings.index.provided_name
# settings.index.creation_date
# settings.index.uuid
# settings.index.version

# create index
json_mappings=`cat file_with_index.json`
curl -X PUT $ELASTIC_HOST/$INDEX_NAME -H 'Content-Type: application/json' \
-d @- << EOF
{
	"mappings": $json_mappings,
	"settings" : {
        "index" : {
            "number_of_shards" : 1,
            "number_of_replicas" : 0
        }
    }
}
EOF

# delete index
curl -s --user "$SEARCH_USER:$SEARCH_PASSWORD" -X GET $ELASTIC_HOST/$ELASTIC_INDEX > file_with_index.json
```


### search request query request
```bash
curl -X GET "$ELASTIC_HOST/$INDEX_NAME/_search?q=front_vehicle.distance:>100&size=11&pretty=true"
curl -X GET "$ELASTIC_HOST/$INDEX_NAME/_search?q=road_type:highway"
```
```bash
echo '{"query": {"match" : {"sessionId" : "a8b8-0174df8a3b3d"}}}' > request.json
echo '{"query": { "range" : {"front_vehicle.distance": {"gte": 100}}}}' > request.json

curl -X POST -H "Content-Type: application/json" -u $LABEL_SEARCH_USERNAME:$LABEL_SEARCH_PASSWORD -d @request.json "$ELASTIC_HOST/$ELASTIC_INDEX/_search"
```

### remove records delete records
```sh
curl -X PUT $ELASTIC_HOST/$INDEX_NAME/_delete_by_query' -H 'Content-Type: application/json' \
-d @- << EOF
{
    "query": {
        "term": {
            "sessionId.keyword": {
                "value": "8a140c23-420c-3bf0a285",
                "boost": 1.0
            }
        }
    }
}
EOF
```
