## Kafka Cheetsheet

### repartition

change replication on a running topic

```bash
#!/bin/bash

if [ -z $1 ] || [ -z $2 ] || [ -z $3 ]; then
    echo "USAGE: repartition <topics> <replicas>"
    echo "    topics_example: topic1,topic2,topic3"
    echo "    replicas_example : 3"
    exit 344
fi

TOPICS=${1//,/ }
REPLIC=$2
ZK_NODE=$3

function create_input() {
    local topic=$1
    local replic=$2
    cat > input.json <<END
{"version":1,
  "partitions":[
$(get_partition_strings $topic $replic)
]}
END
}

function get_partition_strings() {
    local outstr=""
    local topic=$1
    local replic=$2
    local parts=$(kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic $topic 2>/dev/null | awk '/PartitionCount/ { print $4 }')
    for ((i=$parts;i>1;i--))
        do outstr=$outstr"    {\"topic\":\"$topic\",\"partition\":$(($i-1)),\"replicas\":$(rand_replica_string $replic)},
"
    done
    outstr=$outstr"    {\"topic\":\"$1\",\"partition\":0,\"replicas\":$(rand_replica_string $replic)}"
    echo "$outstr"
}

function rand_replica_string() {
    local replic=$1
    echo \[$(shuf -i 0-$(($replic-1)) | tr "\n" "," | head -c -1)\]
}

function main() {
    local topics=$1
    local replic=$2
    local zk=$3
    echo "Setting replicas to $replic for topics: ${topics// /, }"
    for topic in $topics
        do create_input $topic $replic
        kafka-reassign-partitions.sh --zookeeper $zk --reassignment-json-file input.json --execute
    done
    rm -f input.json
}

main "$TOPICS" "$REPLIC" "$ZK_NODE"
```

### kafka-service script

```bash
#!/usr/bin/env bash

CMD=$1

function create-compose() {
    cat > docker-compose.yml <<END
version: '3.5'

networks:
  dev-net:
    name: dev-net
    driver: bridge
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    ports:
      - 2181:2181
    networks:
      - dev-net
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    networks:
      - dev-net
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
END
}

function usage-help() {
    echo 'USAGE: kafka-service <up|down>'
}

function main() {
    case $CMD in
        up)
            create-compose
            docker-compose up -d
            echo "associated containers should be run in the network dev-net"
            ;;
        down)
            create-compose
            docker-compose down
            echo "local cluster destroyed"
            ;;
        *)
            usage-help
	    ;;
    esac
    rm -f docker-compose.yml
}

main
```
