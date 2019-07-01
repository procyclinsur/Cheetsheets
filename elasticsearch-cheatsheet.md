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
curl -X GET "localhost:9200/index_name/_search?q=test" | jq .
```

### Start ELK Dev Cluster

elk-service.sh

```bash
#!/usr/bin/env bash

CMD=$1
VER=$2

function command-help() {
    echo -e "\nUSAGE: elk-service <up|down> <elk-version>\n"
    exit 0
}

function create-pipeline() {
    cat > logstash.conf <<END
input {
  beats {
    port => 5044
  }
}
filter {
}
output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    manage_template => false
    index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
  }
}
END
}

function create-compose() {
    cat > docker-compose.yml <<END
version: '3.5'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:$1
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
      discovery.type: "single-node"
    networks:
      - elk
  logstash:
    image: docker.elastic.co/logstash/logstash-oss:$1
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5000:5000"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    networks:
      - elk
    depends_on:
      - elasticsearch
  kibana:
    image: docker.elastic.co/kibana/kibana-oss:$1
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch
networks:
  elk:
    external:
      name: dev-net
END
}

function main() {
    if [ -z $CMD ] || [ -z $VER ]; then
        command-help
    fi

    case $CMD in
        up)
            create-pipeline
            create-compose $VER
            docker-compose up -d
            ;;
        down)
            create-pipeline
            create-compose $VER
            docker-compose down
            ;;
        *)
            command-help
            ;;
    esac
    rm -f logstash.conf
    rm -f docker-compose.yml
}

main
```
