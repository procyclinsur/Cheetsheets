# elasticsearch cheatsheet

### get system health

```bash
curl -X GET "localhost:9200/_cat/health?v"
```

### get all indexes

```bash
curl -X GET "localhost:9200/_cat/indices?v"
```

### get specific index

```bash
curl -X GET "localhost:9200/index_name"
```

### get data from index matching "test"

```bash
curl -X GET "localhost:9200/test-nginx-2019-06/_search?q=test" | jq .
```
